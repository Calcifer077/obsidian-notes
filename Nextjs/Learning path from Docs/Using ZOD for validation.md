In our project there are basically two type of components, one that is shown to the user (client component) and other is server action (that talks to the database). When the data is sent to the server action for insertion, updating we will validate the data in `actions` and give appropriate errors to the client.

Below is a basic `zod` structure for validation:
```ts
'use server';

import { z } from "zod";

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string({
    invalid_type_error: "Please select a customer.",
  }),
  amount: z.coerce
    .number()
    .gt(0, { message: "Please enter an amount greater than $0." }),
  status: z.enum(["pending", "paid"], {
    invalid_type_error: "Please select an invoice status.",
  }),
  date: z.string(),
});
  
const CreateInvoice = FormSchema.omit({ id: true, date: true });

// This is the true error object which will be used in the frontend.
export type State = {
  errors?: {
    customerId?: string[];
    amount?: string[];
    status?: string[];
  };
  message?: string | null;
};
```
The `FormSchema` defines what kind of our function will be using (validation). So say there is `customer_id`, we convert it to `string` but if the length is less than 1 we will give a error.

`State` is used to return errors to the client.

How can we use the above things in our action.
```ts
export async function createInvoice(prevState: State, formData: FormData) {
    const validatedFields = CreateInvoice.safeParse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  // If form validation fails, return errors early. Otherwise, continue.
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: "Missing Fields. Failed to Create Invoice.",
    };
  }

  const { customerId, amount, status } = validatedFields.data;
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];
  
  try {
    await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
  } catch (error) {
    // We'll also log the error to the console for now
    console.error(error);
    throw new Error("Database Error: Failed to Create Invoice.");
  }
  
  // 'redirect' is outside the try catch block because 'redirect' works by throwing a redirect error. If it were inside try-catch it will be caught by the catch blockl
  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```
How does `safeParse` work?
Zod checks 3 things:
- Did user pick a customer?
- Is amount > 0?
- Did user pick pending or paid?

If anything is wrong → `validatedFields.success` = false
We returns errors which we can render on the client, else the request will go through to the database.

#### Frontend / Client side
This will use [useActionState](../../React/Hooks/useActionState.md) from react.
Earlier you could have just called the server action directly (with binding) but with the help of `useActionState` you have to change a few things.
```tsx
import { useActionState } from "react";
import { createInvoice, State } from "@/app/lib/actions";

export default function Form({ customers }: { customers: CustomerField[] }) {
	// This initial state follow 
	const initialState: State = { message: null, errors: {} };
	const [state, formAction] = useActionState(createInvoice, initialState);
  
  // ...
 };
```
change in action of form
```tsx
<form action={createInvoice}>
```
converts to
```tsx
<form action={formAction}>
```

Using errors:
```tsx
<div className="mb-4">
	<label htmlFor="amount" className="mb-2 block text-sm font-medium">
		Choose an amount
	</label>
	<div className="relative mt-2 rounded-md">
		<div className="relative">
			<input
				id="amount"
                name="amount"
                type="number"
                step="0.01"
                placeholder="Enter USD amount"
                className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
                aria-describedby="amount-error"
              />
              <CurrencyDollarIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
            </div>
          </div>
          <div id="amount-error" aria-live="polite" aria-atomic="true">
            {state.errors?.amount &&
              state.errors.amount.map((error: string) => (
                <p className="mt-2 text-sm text-red-500" key={error}>
                  {error}
                </p>
              ))}
          </div>
        </div>
```
You can also access `message` from `state` in a similar way.

#### using with `bind`
```tsx
 const initialState: State = { message: null, errors: {} };
 const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
 const [state, formAction] = useActionState(updateInvoiceWithId, initialState);
```
##### A brief intro about `useActionState`:
New React hook that connects form ↔ server function. 
```tsx
const [state, formAction] = useActionState(createInvoice, initialState);
```
The meaning of above line:
- It says: "Hey React, whenever this form is submitted, please call the function `createInvoice` that lives on the server"
- It also gives us state → this contains errors or success message after we try to submit

How it all happens:
```text
User fills form → presses "Create Invoice"
   ↓
Form sends data to → createInvoice function (in actions.ts)
   ↓
createInvoice checks if data is good (using Zod)
   ↓
If bad → sends error messages back
If good → saves to database → redirects to invoice list
```
