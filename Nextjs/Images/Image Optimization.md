[Link](https://nextjs.org/docs/app/getting-started/images)

The Next.js [`<Image>`](https://nextjs.org/docs/app/api-reference/components/image) component extends the HTML `<img>` element to provide:

- **Size optimization:** Automatically serving correctly sized images for each device, using modern image formats like WebP.
- **Visual stability:** Preventing [layout shift](https://web.dev/articles/cls) automatically when images are loading.
- **Faster page loads:** Only loading images when they enter the viewport using native browser lazy loading, with optional blur-up placeholders.
- **Asset flexibility:** Resizing images on-demand, even images stored on remote servers.

You can use it by importing it from `next/image`
```tsx
import Image from 'next/image';
```

**Local Images:**
You can store static files, like images and fonts under a folder called `public`. Files inside `public` can be referenced by your code starting from the base URL (`/`)

```tsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```
If the image is statically imported, Next.js will automatically determine the intrinsic [`width`](https://nextjs.org/docs/app/api-reference/components/image#width-and-height) and [`height`](https://nextjs.org/docs/app/api-reference/components/image#width-and-height). These values are used to determine the image ratio and prevent [Cumulative Layout Shift](https://web.dev/articles/cls) while your image is loading.

**Remote Images:**
To use a remote image, you can provide a URL string for the `src` property.
```tsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```
You have to specify `width`,  `height` and any additional properties that you need, since Next.js does not have access to remote files during the build process. 
To safely allow images from remote servers, you need to define a list of supported URL patterns in `next.config.js`.

Example:
```ts
import type { NextConfig } from 'next'
 
const config: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
        search: '',
      },
    ],
  },
}
 
export default config
```

**Short Notes:**
1. Use `<Image/>` for optimization of your images.
2. If image is local, comes from public folder, than Next.js refers much of the thing on its own.
3. If the image is remote, you have to tell everything about it and even configure in config file.