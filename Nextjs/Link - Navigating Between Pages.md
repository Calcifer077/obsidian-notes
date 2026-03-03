[Link](https://nextjs.org/learn/dashboard-app/navigating-between-pages)

### Why optimize navigation?

When you are using traditional `<a>` HTML element, whenever you click on them (even if they are part of application's navigation) there will be full page reload.

### The `<Link>` Component
In Next.js, you can use the `<Link />` Component to link between pages in your application. `<Link>` allows you to do [client-side navigation](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works) with JavaScript.
Client-side navigation means that instead of reloading the page, it updates the content dynamically by:
- Keeping any shared layouts and UI
- Replacing the current page with the prefetched loading state or a new page if available.

How to use?
import `Link` from `next/Link`, and simply replace all your `<a>` elements with `<Link>`

```tsx
import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
 
// ...
 
export default function NavLinks() {
  return (
    <>
      {/* .. */}
          <Link
            key={link.name}
            href={link.href}
            className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        {/* .. */}
    </>
  );
}
```
Using `<Link>` you should now be able to navigate between the pages without seeing a full refresh. Although parts of your application are rendered on the server, there's no full page refresh, making it feel like a native web app. Why is that?

### Automatic code-splitting and prefetching

To improve the navigation experience, Next.js automatically code splits your application by route segments. This is different from a traditional React [SPA](https://nextjs.org/docs/app/building-your-application/upgrading/single-page-applications), where the browser loads all your application code on the initial page load.

Splitting code by routes means that pages become isolated. If a certain page throws an error, the rest of the application will still work. This is also less code for the browser to parse, which makes your application faster.

Furthermore, in production, whenever [`<Link>`](https://nextjs.org/docs/api-reference/next/link) components appear in the browser's viewport, Next.js automatically **prefetches** the code for the linked route in the background. By the time the user clicks the link, the code for the destination page will already be loaded in the background, and this is what makes the page transition near-instant!

This prefetching can cause certain types of problems, suppose for apps which have a lot of links, to avoid this we can stop prefetch or only prefetch on hover.
[How to avoid prefetching](https://nextjs.org/docs/app/getting-started/linking-and-navigating#disabling-prefetching)

### Pattern: Showing active links
A common UI pattern is to show an active link to indicate to the user what page they are currently on. To do this, you need to get the user's current path from the URL. Next.js provides a hook called [`usePathname()`](https://nextjs.org/docs/app/api-reference/functions/use-pathname) that you can use to check the path and implement this pattern. As `usePathname()` is a react hook you need to convert your component into a client component.

How to convert to client component?
```tsx
'use client'; // At the top of your file
```
How to use `usePathname()`?
```tsx
import {usePathname} from 'next/navigation';

export default function NavLinks(){
	const pathname = usePathname();
}
```

Short Notes:
1. Use `Link` instead of `<a>`
2. You can also avoid prefetching if you want.

Backlinks:
