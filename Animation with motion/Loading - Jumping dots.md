What we are creating?
[Motion.dev](https://examples.motion.dev/react/loading-jumping-dots)

To define it, there are three dots that jumps up and down.

This example used `variants` to define the jump animation and `staggerChildren` to create the wave effect as each dot animates in sequence.
## Get started

Creating a basic spinner element using CSS styling:
```js
function LoadingJumpingDots() {
    return (
        <div className="container">
            <div className="dot" />
            <div className="dot" />
            <div className="dot" />
        </div>
    )
}
```

The dots are styled as small circles:
```CSS
.container {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 10px;
}

.dot {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: var(--hue-1);
}
```

## Let's animate!

### Import from motion
```ts
import { motion, type Variants } from "motion/react";
```

### Define the jump animation
Create a variants object that defines the jump animation:
```ts
const dotVariants: Variants = {
    jump: {
        transform: "translateY(-30px)",
        transition: {
            duration: 0.8,
            repeat: Infinity,
            repeatType: "mirror",
            ease: "easeInOut",
        },
    },
}
```

The `transform: "translateY(-30px)"` moves each dot 30 pixels upward. The `repeatType: "mirror"` makes the animation play in reverse after completing, creating a smooth up-and-down bounce.

![Why use transform directly?](Loading%20-%20Circle%20spinner.md#Why%20use%20transform%20directly?)

### Apply the animation with stagger

Convert the elements to motion components and apply the staggered animation:
```js
function LoadingJumpingDots() {
    return (
        <motion.div
            animate="jump"
            transition={{ staggerChildren: -0.2, staggerDirection: -1 }}
            className="container"
        >
            <motion.div className="dot" variants={dotVariants} />
            <motion.div className="dot" variants={dotVariants} />
            <motion.div className="dot" variants={dotVariants} />
        </motion.div>
    )
}
```

The `staggerChildren: -0.2` creates a 0.2 second delay between each dot's animation start. The `staggerDirection: -1` reverses the order so the last dot starts first.

## Conclusion

We've built an engaging jumping dots animation using Motion's variant and stagger systems. The `repeatType: "mirror"` creates smooth bouncing, while `staggerChildren` orchestrates the wave effect across all three dots.

## Sources

[motion.dev](https://motion.dev/tutorials/react-loading-jumping-dots)

## tags
#javascript 
#animation
#motion 