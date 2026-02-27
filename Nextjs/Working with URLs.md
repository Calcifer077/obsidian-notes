[Link](https://nextjs.org/learn/dashboard-app/adding-search-and-pagination)
To showcase working with URLs, we will implement searching functionality. You can incorporate pagination too, in a similar way.

How search works? Our search functionality will span the client and the server. When a user searches for anything on the client, the URL will be updated, data will be fetched on the server, and component will re-render on the server with new data.

##### Why use URL search params?
Benefits of implementing search with URL params:
- **Bookmarkable and shareable URLs**: Since the search parameters are in the URL, users can bookmark the current state of the application, including their search queries and filters, for future reference or sharing.
- **Server-side rendering**: URL parameters can be directly consumed on the server to render the initial state, making it easier to handle server rendering.
- **Analytics and tracking**: Having search queries and filters directly in the URL makes it easier to track user behaviour without requiring additional client-side logic.

##### Adding search functionality
These are the Next.js client hooks that we'll use to implement the search functionality:

- **`useSearchParams`**- Allows you to access the parameters of the current URL. For example, the search params for this URL `/dashboard/invoices?page=1&query=pending` would look like this: `{page: '1', query: 'pending'}`.
- **`usePathname`** - Lets you read the current URL's pathname. For example, for the route `/dashboard/invoices`, `usePathname` would return `'/dashboard/invoices'`.
- **`useRouter`** - Enables navigation between routes within client components programmatically. There are [multiple methods](https://nextjs.org/docs/app/api-reference/functions/use-router#userouter) you can use.
Above are client hooks, so you can only use them in client components.

Here's a quick overview of the implementation steps:

1. Capture the user's input.
2. Update the URL with the search params.
3. Keep the URL in sync with the input field.
4. Update the UI to reflect the search query after fetching from DB.

##### 1. Capture the user's input
Just add a `handler` function and a `onChange` event listener.
```tsx
'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
 
export default function Search({ placeholder }: { placeholder: string }) {
  function handleSearch(term: string) {
    console.log(term);
  }
 
  return (
    //..
      <input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
        onChange={(e) => {
          handleSearch(e.target.value);
        }}
      />
     //..
  );
}
```

##### 2. Update the URL with the search params
Steps:
* Import the `useSearchParams` hook from `next/navigation` and assign it to a variable.

```tsx
'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
 
  function handleSearch(term: string) {
    console.log(term);
  }
  // ...
}
```
	
* Inside `handleSearch`, create a new [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) instance using your new `searchParams` variable. `URLSearchParmas` is a Web API that provides utility methods for manipulating the URL query parameters.
```tsx
'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
 
  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
  }
  // ...
}
```
* Next, `set` the params string based on user's input. If the input is empty, you want to delete it.
```tsx
'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
 
  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
  }
  // ...
}
```
 * Now that you have the query string. You can use Next.js's `useRouter` and `usePathname` hooks to update the URL.
```tsx
'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams, usePathname, useRouter } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();
 
  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }
}
```
Here's a breakdown of what's happening in above code:
- `${pathname}` is the current path, in your case, `"/dashboard/invoices"`.
- As the user types into the search bar, `params.toString()` translates this input into a URL-friendly format.
- `replace(${pathname}?${params.toString()})` updates the URL with the user's search data. For example, `/dashboard/invoices?query=lee` if the user searches for "Lee".
- The URL is updated without reloading the page, thanks to Next.js's client-side navigation (which you learned about in the chapter on [navigating between pages](https://nextjs.org/learn/dashboard-app/navigating-between-pages).
##### 3. Keeping the URL and input in sync
To ensure the input field is in sync with the URL and will be populated when sharing, we can pass a `defaultValue` to input by reading from `searchParams`.
```tsx
<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
  // Convert to string if exists else null
  defaultValue={searchParams.get('query')?.toString()}
/>
```
##### 4. Update the UI to reflect the search query after fetching from DB.
Finally, you need to update server component to reflect search query.

```tsx
import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import { CreateInvoice } from '@/app/ui/invoices/buttons';
import { lusitana } from '@/app/ui/fonts';
import { Suspense } from 'react';
import { InvoicesTableSkeleton } from '@/app/ui/skeletons';
 
export default async function Page(props: {
  searchParams?: Promise<{
    query?: string;
    page?: string;
  }>;
}) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
 
  return (
    //..
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
    //..
  );
}
```
Breakdown of above code:
* First we accept a `prop` called `searchParams` if it exists. If it does exists, it is a `promise` and can contain two values `query` and `page`.
* Inside the component we first `await` the result from `searchParams` and get values from it.
* We than send the data to some other component which does the `asynchronous` task.

* **When to use the `useSearchParams()` hook vs. the `searchParams` prop?**
You might have noticed we used two different ways to extract search params. Whether you use one or the other depends on whether you're working on the client or the server.

- `<Search>` is a Client Component, so you used the `useSearchParams()` hook to access the params from the client.
- `<Table>` is a Server Component that fetches its own data, so you can pass the `searchParams` prop from the page to the component.

As a general rule, if you want to read the params from the client, use the `useSearchParams()` hook as this avoids having to go back to the server. 

##### Best practice: Debouncing
We have implemented the search functionality but there is a issue here. With every keystroke, we are querying to the database. Suppose the user is typing **Delba** what will happen?

```console
Searching... D
Searching... De
Searching... Del
Searching... Delb
Searching... Delba
```
Imaging if our application has thousand of users, each sending a new request to our database on each keystroke, that will create problems.

**Debouncing** is a programming practice that limits the rate at which a function can fire. In our case, we only want to query the database when the user has stopped typing.

* **How Debouncing Works:**
1. **Trigger Event**: When an event that should be debounced (like a keystroke in the search box) occurs, a timer starts.
2. **Wait**: If a new event occurs before the timer expires, the timer is reset.
3. **Execution**: If the timer reaches the end of its countdown, the debounced function is executed.

We will use a library called [`use-debounce`](https://www.npmjs.com/package/use-debounce). 
```bash
pnpm i use-debounce
```
In our `<Search>` component:
```tsx
// ...
import { useDebouncedCallback } from 'use-debounce';
 
// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);
 
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
```
This function will wrap the contents of `handleSearch`, and only run the code after a specific time once the user has stopped typing (300ms).
By debouncing, we can reduce the number of requests sent to your database, thus saving resources.