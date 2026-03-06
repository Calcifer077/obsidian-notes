#### What is Static Rendering?

With static rendering, data fetching and rendering happens on the server at build time (when you deploy) or when [revalidating data](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#revalidating-data). Page is made once and not changed after deployment.

Whenever a user visits your application, the cached result is served. There are a couple of benefits of static rendering:

- **Faster Websites** - Prerendered content can be cached and globally distributed when deployed to platforms like [Vercel](https://vercel.com/). This ensures that users around the world can access your website's content more quickly and reliably.
- **Reduced Server Load** - Because the content is cached, your server does not have to dynamically generate content for each user request. This can reduce compute costs.
- **SEO** - Prerendered content is easier for search engine crawlers to index, as the content is already available when the page loads. This can lead to improved search engine rankings.

Static rendering is useful for UI with **no data** or **data that is shared across users**, such as a static blog post or a product page. It might not be a good fit for a dashboard that has personalized data which is regularly updated. 

The opposite of static rendering is dynamic rendering.

#### What is Dynamic Rendering?

With dynamic rendering, content is rendered on the server for each user at **request time** (when the user visits the page). There are a couple of benefits of dynamic rendering:

- **Real-Time Data** - Dynamic rendering allows your application to display real-time or frequently updated data. This is ideal for applications where data changes often.
- **User-Specific Content** - It's easier to serve personalized content, such as dashboards or user profiles, and update the data based on user interaction.
- **Request Time Information** - Dynamic rendering allows you to access information that can only be known at request time, such as cookies or the URL search parameters.
With dynamic rendering, **your application is only as fast as your slowest data fetch**. Suppose you grouped together multiple async requests inside a `Promise.all` to avoid the waterfall problem, but the result of `Promise.all` will only be available after the slowest request has returned its value.