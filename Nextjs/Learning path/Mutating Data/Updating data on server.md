Steps to be followed:
1. Create a new dynamic route segment with the invoice `id`.
2. Read the invoice `id` from the page params.
3. Fetch the specific invoice from your database.
4. Update the invoice data in your database.

##### 1. Create a dynamic route segment
Next.js allows you to create [Dynamic Route Segments](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) when you don't know the exact segment name and want to create routes based on data. This could be blog post titles, product pages, etc. You can create dynamic route segments by wrapping a folder's name in square brackets. For example, `[id]`, `[post]` or `[slug]`.

How it would look:
![](../../../assets/dynamic_route_segement_from_next_doc.png)
/
/
Use the `id` from above in your `Link` which will refer to some other page:
```tsx
export function UpdateInvoice({ id }: { id: string }) {
  return (
    <Link
      href={`/dashboard/invoices/${id}/edit`}
      className="rounded-md border p-2 hover:bg-gray-100"
    >
      <PencilIcon className="w-5" />
    </Link>
  );
}
```
##### 2. Read the invoice `id` from page `params`
```tsx
// edit page
import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchCustomers } from '@/app/lib/data';
 
export default async function Page(props: { params: Promise<{ id: string }> }) { 
	const params = await props.params; 
	const id = params.id;
	
	return (
	    <main>
	      <Breadcrumbs
	        breadcrumbs={[
	          { label: 'Invoices', href: '/dashboard/invoices' },
	          {
	            label: 'Edit Invoice',
	            href: `/dashboard/invoices/${id}/edit`,
	            active: true,
	          },
	        ]}
	      />
	      {/* This form should be pre-populated with data*/}
	      <Form invoice={invoice} customers={customers} />
	    </main>
	);
}
```
Why are we using `Promise` above?
This page comes under a dynamic route segment.
In order to access that id (the one that makes this page dynamic), you need to await it, because it is a promise like `searchParams`.

##### 3. Fetch the specific invoice
Just fetch the data that you need using the `id` obtained in previous step.

##### 4. Pass the `id` to Server Action
Lastly we want to pass the `id` to server action so that the required action can be performed. 

Now what won't work for the above step.
```tsx
<form action={updateInvoice(id)}> </form>
```
Why above won't work?
Because the action will be called at the time of rendering, that way there is no data to be sent to the server.

We could have done?
```tsx
<form action={() => updateInvoice(id)}> </form>
```
Above also won't work because our sever action depend on `formData` and if you do it like above react won't pass any `formData`.

Best way
```tsx
const updateInvoiceWithId = updateInvoice.bind(null, invoice.id); 
return <form action={updateInvoiceWithId}>{/* ... */}</form>;
```
Using `bind` will make sure that both `id` and `formData` is passed to the Server action.
We could have also used a hidden input but that can't be used if you have some sensitive data which you don't want to expose to the frontend.

Now just create a server action that performs the required task.