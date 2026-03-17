Metadata is crucial for SEO and shareability.

#### What is Metadata?
In web development, metadata provides additional details about a webpage. Metadata is not visible to the users visiting the page. Instead, it works behind the scenes, embedded within the page's HTML, usually within the `<head>` element. This hidden information is crucial for search engines and other systems that need to understand your webpage's content better.

#### Why is metadata important?
Metadata plays a significant role in enhancing a webpage's SEO, making it more accessible and understandable for search engines and social media platforms. Proper metadata helps search engines effectively index webpages, improving their ranking in search results. Additionally, metadata like Open Graph improves the appearance of shared links on social media, making the content more appealing and informative for users.

#### Types of Metadata
There are various types of metadata, each serving a unique purpose. Some common types include:

**Title Metadata:** Responsible for the title of a webpage that is displayed on the browser tab. It's crucial for SEO as it helps search engines understand what the webpage is about.
```html
<title>Page Title</title>
```

**Description Metadata**: This metadata provides a brief overview of the webpage content and is often displayed in search engine results.
```html
<meta name="description" content="A brief description of the page content." />
```

**Keyword Metadata**: This metadata includes the keywords related to the webpage content, helping search engines index the page.
```html
<meta name="keywords" content="keyword1, keyword2, keyword3" />
```

**Open Graph Metadata**: This metadata enhances the way a webpage is represented when shared on social media platforms, providing information such as the title, description, and preview image.
```html
<meta property="og:title" content="Title Here" />
<meta property="og:description" content="Description Here" />
<meta property="og:image" content="image_url_here" />
```

**Favicon Metadata**: This metadata links the favicon (a small icon) to the webpage, displayed in the browser's address bar or tab.
```html
<link rel="icon" href="path/tp/favicon.co" />
```

#### Adding Metadata
Next.js has a Metadata API that can be used to define your application metadata. There are two ways you can add metadata to your application:
- **Config-based**: Export a [static `metadata` object](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-object) or a dynamic [`generateMetadata` function](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#generatemetadata-function) in a `layout.js` or `page.js` file.
- **File-based**: Next.js has a range of special files that are specifically used for metadata purposes:
    - `favicon.ico`, `apple-icon.jpg`, and `icon.jpg`: Utilized for favicons and icons
    - `opengraph-image.jpg` and `twitter-image.jpg`: Employed for social media images
    - `robots.txt`: Provides instructions for search engine crawling
    - `sitemap.xml`: Offers information about the website's structure

#### Page title and descriptions
We can also add a [`metadata` object](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-fields) to any `layout.ts` or `page.ts` file to add additional page information like title and description. Any metadata in `layout.js` will be inherited by all pages that use it unless overridden. 

**Usage**
```tsx
import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: 'Acme Dashboard',
  description: 'The official Next.js Course Dashboard, built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
 
export default function RootLayout() {
  // ...
}
```
You can simply override this in any nested route.

Above way works, but we are repeating the title of the application in every page. If something changes, we would have to update it everywhere.

Instead, we can use the `title.template` field in the `metadata` object to define a template for your page title.
```tsx
import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: {
    template: '%s | Acme Dashboard',
    default: 'Acme Dashboard',
  },
  description: 'The official Next.js Learn Dashboard built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
```
The `%s` in the template will be replaces with the specific page title.

Now, in any nested route we can use:
```tsx
export const metadata: Metadata = {
  title: 'Invoices',
};
```
The title for the above page will be `Invoices | Acme Dashboard`