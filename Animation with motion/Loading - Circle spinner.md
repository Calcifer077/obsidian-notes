What we are creating?
[Motion.dev](https://examples.motion.dev/react/loading-circle-spinner)

To define it, it's just a circle going on in circles Infinitely.

Infinite rotation is achieved with a single `animate` prop:
```js
animate={{ transform: "rotate(360deg)" }}
```

## Get started

Creating a basic spinner element using CSS styling:
```js
export default function LoadingCircleSpinner() {
    return (
        <div className="container">
            <div className="spinner" />
        </div>
    )
}
```

The spinner is styled as a circle with a partially colored border, creating the classic loading indicator appearance:
```CSS
.spinner {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    border: 4px solid var(--divider);
    border-top-color: var(--hue-1);
}
```

## Let's animate!

### Import from motion
```js
import { motion } from "motion/react"
```

### Convert to motion component
Replace the static `div` with a `motion.div`:
```js
<motion.div className="spinner" />
```

### Add the rotation animation
Now add the `animate` prop to create the spinning effect:
```js
<motion.div
    className="spinner"
    animate={{ transform: "rotate(360deg)" }}
    transition={{
        duration: 1.5,
        repeat: Infinity,
        ease: "linear",
    }}
/>
```

The `transition` configuration:
- `duration: 1.5` - Each full rotation takes 1.5 seconds
- `repeat: Infinity` - The animation loops forever
- `ease: "linear"` - Maintains constant speed throughout the rotation

Using `linear` easing is essential for spinners, as it prevents the animation from speeding up or slowing down, creating a smooth, consistent rotation.

### Why use transform directly?

We're using `transform: "rotate(360deg)"` instead of the shorthand `rotate: 360`. Animating the `transform` property directly enables hardware-accelerated animations, which run on the GPU rather than the CPU. This is especially important during loading sequences, as loading is typically a time of heavy CPU load. By offloading the animation to the GPU, the spinner remains smooth even when the main thread is busy. [Learn more about web animation performance](https://motion.dev/magazine/web-animation-performance-tier-list).

## Conclusion

In this note, we learned how to:
- Create an infinitely rotating animation using the `animate` prop
- Use `repeat: Infinity` to loop an animation forever
- Apply `linear` easing for consistent rotation speed
- Use `transform` directly for hardware-accelerated animations

## Sources
[motion.dev](https://motion.dev/tutorials/react-loading-circle-spinner)

## tags
#javascript 
#animation
#motion 