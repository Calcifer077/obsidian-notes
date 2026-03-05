Steps to be followed (can be varied according to your case):
1. Create a form to capture the user's input.
2. Create a Server Action and invoke it from the form.
3. Inside your Server Action, extract the data from the `formData` object.
4. Validate and prepare the data to be inserted into your database.
5. Insert the data and handle any errors.
6. Revalidate the cache and redirect the user back to parent page of form.

##### 1. Create a form to capture user's input
In this step you just need to create a simple input inside a form.

##### 2. Create a Server Action
There are two ways to create a server action. One is to create a dedicated file which will contain all your actions in a central place preferably under `/lib/action.ts`. Mention `use server;` directive on top and all functions inside this will be treated as server actions. Make sure to export them. Now you can use these functions in both client and server component.

Another way is to create simple function and use `use server;` directive as the first line in the functions body. Can only use these functions in server component.

```tsx
function doSomething(){
	'use server';
	
	// This is a server action.
}
```

Will be using first method:
```ts
'use server'; 

export async function createInvoice(formData: FormData) {}
```

Then in the file where you capture user's input, import above function and use it.
```tsx
import { createInvoice } from '@/app/lib/actions';

export default function Form({ 
	customers,
}: { 
	customers: CustomerField[];
}) { 
	return ( 
		<form action={createInvoice}> // ... 
	)
}
```
**Good to know**: In HTML, you'd pass a URL to the `action` attribute. This URL would be the destination where your form data should be submitted (usually an API endpoint).
However, in React, the `action` attribute is considered a special prop - meaning React builds on top of it to allow actions to be invoked.
Behind the scenes, Server Actions create a `POST` API endpoint. This is why you don't need to create API endpoints manually when using Server Actions.

##### 3. Extract `formData`
In our `action.ts` for extracting `formData`there are a [couple of methods](https://developer.mozilla.org/en-US/docs/Web/API/FormData) we can use. For this example, let's use the [`.get(name)`](https://developer.mozilla.org/en-US/docs/Web/API/FormData/get) method.

```ts
'use server';
 
export async function createInvoice(formData: FormData) {
  const rawFormData = {
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  };
}
```

##### 4. Validate and Prepare the data
Before sending data to the server we need to make sure that the data is in correct format and with correct types.
In our case data should look like this:
```ts
export type Invoice = { 
	id: string; // Will be created on the database 
	customer_id: string; 
	amount: number; // Stored in cents 
	status: 'pending' | 'paid'; 
	date: string;
};
```
We need to validate the type, for example if you were to check `type` of `amount` it will be a `string`. This is because `input` elements with `type="number"` actually return a string, not a number!

To handle type validation, we can use  [Zod](https://zod.dev/).

In our `action.ts` file:
```ts
'use server';
 
import { z } from 'zod';
 
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  // The `amount` field is specifically set to coerce (change) from a string to a number while also validating its type.
  amount: z.coerce.number(),
  status: z.enum(['pending', 'paid']),
  date: z.string(),
});
 
const CreateInvoice = FormSchema.omit({ id: true, date: true });
 
export async function createInvoice(formData: FormData) {
  // ...
}
```

You can then pass your `rawFormData` to `CreateInvoice` to validate the types:
```ts
// ...
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
}
```
As we are dealing with money here, its a good practice to store monetary values in cents in your database to eliminate JavaScript floating-point errors and ensure greater accuracy so convert them to cents.

##### 5. Inserting the data into our database
Now that we have all the values we can write a sql query and pass the data to database.
```ts
// ...
import postgres from 'postgres';
 
const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });
 
// ...
 
export async function createInvoice(formData: FormData) {
  // ...
  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
}
```

##### 6. Revalidate and Redirect
Next.js has a client-side router cache that stores the route segments in the user's browser for a time. Along with [prefetching](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#1-prefetching), this cache ensures that users can quickly navigate between routes while reducing the number of requests made to the server.

Since you're updating the data displayed in the invoices route, you want to clear this cache and trigger a new request to the server. You can do this with the [`revalidatePath`](https://nextjs.org/docs/app/api-reference/functions/revalidatePath) function from Next.js.
Once the database has been updated, the `/dashboard/invoices` path will be revalidated, and fresh data will be fetched from the server.

At this point, you also want to redirect the user back to the `/dashboard/invoices` page. You can do this with the [`redirect`](https://nextjs.org/docs/app/api-reference/functions/redirect) function from Next.js.
```ts
'use server';
 
import { z } from 'zod';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import postgres from 'postgres';
 
const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });
 
// ...
 
export async function createInvoice(formData: FormData) {
  // ...
 
  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```

Workflow of above:
1. We should be redirected to the `/dashboard/invoices` route on submission.
2. We should see the new invoice at the top of the table.