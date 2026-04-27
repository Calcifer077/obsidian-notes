You can use **Motion** by installing:

```bash
npm install motion
```

Official website: [https://motion.dev/](https://motion.dev/)

Then import it:

```jsx
import { motion, AnimatePresence } from "motion/react";
```

---

## Converting elements to motion components

You can convert any HTML tag into a motion component by prefixing it with `motion.`

- `div` → `motion.div`
- `p` → `motion.p`
- any HTML element works the same way

---

## How to animate

### Basic animation using `initial` and `animate`

```jsx
<motion.div
  initial={{
    opacity: 0,
    scale: 0.98,
    filter: "blur(10px)",
  }}
  animate={{
    opacity: 1,
    scale: 1,
    filter: "blur(0px)",
  }}
/>
```

- `initial` → starting state
- `animate` → state after the element enters the viewport

---

### Exit animation

```jsx
exit={{
  opacity: 0,
  scale: 0.98,
  filter: "blur(10px)",
}}
```

- Runs when the element leaves the viewport / unmounts

---

### Transition

```jsx
transition={{
  duration: 0.5,
  ease: "easeInOut",
}}
```

- Controls timing and easing
- Ideal duration: **0.3s**

---
### Hover animation

```jsx
<motion.div
  initial={{
    opacity: 0,
    scale: 0.98,
    filter: "blur(10px)",
  }}
  whileHover={{
    opacity: 1,
    scale: 1.05,
    filter: "blur(0px)",
  }}
/>
```

- `whileHover` runs when the element is hovered

---

## Using `AnimatePresence` for exit animations

If your component is conditionally rendered like:

```jsx
{show && <Comp />}
```

The `exit` animation won’t work by default.

To enable it, wrap with `AnimatePresence`:

```jsx
import { AnimatePresence, motion } from "motion/react";
import { useState } from "react";

export default function Page() {
  const [state, setState] = useState(true);

  return (
    <>
      <AnimatePresence>
        {state && (
          <motion.div></motion.div>
        )}
      </AnimatePresence>
    </>
  );
}
```
