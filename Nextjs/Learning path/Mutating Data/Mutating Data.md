#### What are Server Actions?[](https://nextjs.org/learn/dashboard-app/mutating-data#what-are-server-actions)

React Server Actions allow you to run asynchronous code directly on the server. They eliminate the need to create API endpoints to mutate your data. Instead, you write asynchronous functions that execute on the server and can be invoked from your Client or Server Components.

Security is a top priority for web applications, as they can be vulnerable to various threats. This is where Server Actions come in. They include features like encrypted closures, strict input checks, error message hashing, host restrictions, and more — all working together to significantly enhance your application security.

#### Using forms with Server Actions[](https://nextjs.org/learn/dashboard-app/mutating-data#using-forms-with-server-actions)

In React, you can use the `action` attribute in the `<form>` element to invoke actions. The action will automatically receive the native [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) object, containing the captured data.

For example:
```tsx
// Server Component
export default function Page() {
  // Action
  async function create(formData: FormData) {
    'use server';
 
    // Logic to mutate data...
  }
 
  // Invoke the action using the "action" attribute
  // FormData is automatically passed by react to the function, you don't have to specifically tell it to.
  return <form action={create}>...</form>;
}
```
An advantage of invoking a Server Action within a Server Component is progressive enhancement - forms work even if JavaScript has not yet loaded on the client. For example, with slower internet connections.

#### How to create a server action:
There are two ways to create a server action. One is to create a dedicated file which will contain all your actions in a central place preferably under `/lib/action.ts`. Mention `use server;` directive on top and all functions inside this will be treated as server actions. Make sure to export them. Now you can use these functions in both client and server component.

Another way is to create simple function and use `use server;` directive as the first line in the functions body. Can only use these functions in server component.

```tsx
function doSomething(){
	'use server';
	
	// This is a server action.
}
```

Links to different mutations:
[Creating data on server](Creating%20data%20on%20server.md)
[Updating data on server](Updating%20data%20on%20server.md)
[Deleting data on server](Deleting%20data%20on%20server.md)


