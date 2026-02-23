Why optimize images?
Next.js can serve **static assets**, like images, under the top-level [`/public`](https://nextjs.org/docs/app/building-your-application/optimizing/static-assets) folder. Files inside `/public` can be referenced in your application.

With regular HTML, you can do as follows:
```html
<img src="/hero.png" />
```

However, this means you have to manually:
- Ensure your image is responsive on different screen sizes.
- Specify image sizes for different devices.
- Prevent layout shift as the images load.
- Lazy load images that are outside the user's viewport.

**The `<Image>` component:**
The `<Image>` Component is an extension of the HTML `<img>` tag, and comes with automatic image optimization, such as:

- Preventing layout shift automatically when images are loading.
- Resizing images to avoid shipping large images to devices with a smaller viewport.
- Lazy loading images by default (images load as they enter the viewport).
- Serving images in modern formats, like [WebP](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#webp) and [AVIF](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#avif_image), when the browser supports it.

How to use:
```tsx
import Image from 'next/image';

export default function Page(){
	retrun (
		<Image
			src="/hero-desktop.png"
			width={1000}
			height={760}
		/>
	)
}
```

Above we have set `width` and `height`. It's good practice to set the `width` and `height` of your images to avoid layout shift, these should be an aspect ratio **identical** to the source image. These values are _not_ the size the image is rendered, but instead the size of the actual image file used to understand the aspect ratio.
