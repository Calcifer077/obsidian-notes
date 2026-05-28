## Setup & Loading Images

```python
import numpy as np
import matplotlib.pyplot as plt
from PIL import Image

img = np.array(Image.open('photo.jpg'))
plt.imshow(img)
plt.show()
```

**Saving an array back to an image:**

```python
pil_img = Image.fromarray(img)
pil_img.save('output.jpg')
```

> **Why NumPy?** OpenCV internally stores image data as NumPy `ndarray`. Mastering NumPy-based image processing gives you a foundation that transfers directly to OpenCV, PyTorch, TensorFlow, and other computer vision libraries.

---

## Images as Arrays

An image is NOT stored as a "picture" — it becomes a grid of numbers.

```python
img.ndim    # → 3  (for RGB)
img.shape   # → (H, W, 3)  e.g. (1200, 800, 3)
img.dtype   # → uint8
```

### Shape interpretation

|Axis|Meaning|Range|
|---|---|---|
|0|Rows (height, vertical)|0 → H-1|
|1|Columns (width, horizontal)|0 → W-1|
|2|Channels (R, G, B)|0, 1, 2|

### Pixel values

Each pixel is a triplet `[R, G, B]`, each value in `[0, 255]` (`uint8`).

```python
img[20, 20]       # → [R, G, B] at row 20, col 20
img[20, 20, 0]    # → Red value at that pixel

# Common colors
[255, 0,   0]   # red
[0,   255, 0]   # green
[0,   0, 255]   # blue
[255, 255, 255] # white
[0,   0,   0]   # black
```

> **Coordinate system:** `(0,0)` is top-left. Rows increase **downward**, columns increase **rightward**. This is the opposite of standard math `(x, y)`.

---

## Channel Slicing & Manipulation

```python
img[:, :, 0]   # red channel   → shape (H, W)
img[:, :, 1]   # green channel → shape (H, W)
img[:, :, 2]   # blue channel  → shape (H, W)
```

Selecting a single channel index **removes** that axis:

```
(H, W, 3)  →  img[:,:,0]  →  (H, W)
```

### Isolating individual channels for display

```python
img_R, img_G, img_B = img.copy(), img.copy(), img.copy()

img_R[:, :, (1, 2)] = 0   # zero out green and blue
img_G[:, :, (0, 2)] = 0   # zero out red and blue
img_B[:, :, (0, 1)] = 0   # zero out red and green

# Side-by-side comparison
# 'concatenate' is used to join a sequence of arrays along a specified axis.
combined = np.concatenate((img_R, img_G, img_B), axis=1)
plt.imshow(combined)
```

### Zeroing a channel (vectorized)

```python
img[:, :, 0] = 0   # remove red → cyan-ish
img[:, :, 1] = 0   # remove green → magenta/purple
img[:, :, 2] = 0   # remove blue → yellow-ish
```

### Channel reversal (RGB ↔ BGR)

```python
img[:, :, ::-1]
```

Reverses channel order: `[R, G, B]` → `[B, G, R]`.

> **Real-world note:** OpenCV loads images in BGR format by default. Channel reversal converts between OpenCV (BGR) and Matplotlib/PIL (RGB).

---

## Vectorized Assignment & Broadcasting

NumPy can assign to entire regions without Python loops.

```python
img[:, :200] = 255           # white strip on left
img[100:300, 200:400] = 0    # black rectangle

img[:] = [255, 0, 0]         # entire image → red
img *= [1, 0, 0]             # keep only red channel (channel filtering)
img *= [0.5, 1, 1]           # halve red intensity (channel scaling)
```

**Broadcasting aligns dimensions from the right:**
It will check if some dimension match or some dimension is 1. If any of the previous condition match, broadcasting happens.

```
(H, W, 3)
(      3)     ← broadcasts across all rows & columns
```

---

## Views vs Copies

Slicing usually creates a **view** — it shares memory with the original.

```python
red = img[:, :, 0]   # view
red[:] = 0           # ALSO modifies img!

cropped = img[100:400, 200:500]  # view
cropped[:, :, 2] = 0             # ALSO modifies img!
```

To get an independent copy:

```python
cropped = img[100:400, 200:500].copy()
```

|Operation|Result|
|---|---|
|`img[a:b, c:d]`|View (shared memory)|
|`img[mask]` (boolean)|Copy|
|`img[[3,0,2]]` (fancy)|Copy|
|`.copy()`|Always independent|

---

## Region Selection & Cropping

```python
# Crop a rectangle
cropped = img[100:400, 200:500]       # shape (300, 300, 3)
cropped = img[128:-128, 128:-128, :]  # trim 128px border all around

# Implicit trailing dimensions — these are equivalent:
img[:, :200]
img[:, :200, :]
```

---

## Flipping Images

### Using NumPy functions

```python
np.fliplr(img)   # flip left-right (mirror) → reverses axis 1
np.flipud(img)   # flip up-down (invert) → reverses axis 0
```

### Using slice notation (equivalent)

```python
img[:, ::-1]    # horizontal flip
img[::-1]       # vertical flip
img[::-1, ::-1] # 180° rotation
```

**General slice syntax:** `start:stop:step`

```python
a = np.array([1, 2, 3, 4, 5])
a[::-1]   # → [5, 4, 3, 2, 1]
```

Manual flip (loop version, for conceptual understanding):

```python
img0 = img.copy()
for i in range(img0.shape[0] // 2):
    c = img0[i, :, :].copy()
    img0[i, :, :] = img0[img0.shape[0] - i - 1, :, :]
    img0[img0.shape[0] - i - 1, :, :] = c
```

---

## Rotation

### Using `np.rot90`

```python
np.rot90(img)        # 90° counterclockwise (default k=1)
np.rot90(img, k=2)   # 180°
np.rot90(img, k=3)   # 270° counterclockwise = 90° clockwise
```

**Shape changes during rotation:**

```
(H, W, 3)  →  np.rot90  →  (W, H, 3)
```

### Manual rotation (transpose + flip)

```python
# 90° clockwise
img.transpose(1, 0, 2)[:, ::-1]

# 90° counterclockwise
img.transpose(1, 0, 2)[::-1]
```

> **Deep insight:** Rotation is fundamentally **axis swapping + index reversal** — not special image magic.

Manual rotation via column-swap loop (conceptual):

```python
img0 = img.transpose(1, 0, 2)
for j in range(img0.shape[1] // 2):
    c = img0[:, j, :].copy()
    img0[:, j, :] = img0[:, img0.shape[1]-j-1, :]
    img0[:, img0.shape[1]-j-1, :] = c
```

---

## Transpose & Axis Reordering

Transpose **reorders axes**, it doesn't "rotate" the image in the traditional sense.

```python
img.transpose(1, 0, 2)
```

|Old axis|New axis|
|---|---|
|0 (H)|→ 1|
|1 (W)|→ 0|
|2 (C)|→ 2|

Shape: `(H, W, 3)` → `(W, H, 3)`

```python
img.T                  # full reverse: (H,W,3) → (3,W,H)
img.transpose(2, 0, 1) # channel-first: (H,W,3) → (3,H,W)  ← PyTorch format
img.transpose(1, 0, 2) # swap H and W:  (H,W,3) → (W,H,3)
```

> **Deep learning connection:** PyTorch expects `(batch, channels, height, width)`, so `transpose(2,0,1)` is commonly used to convert PIL/NumPy images into model-ready tensors.

**Performance note:** Transpose usually creates a **view** with different strides, not a copy — same memory, different interpretation.

### 2D grayscale transpose example

```python
arr = np.array([[1,2,3],[4,5,6]])  # shape (2,3)
arr.T                               # shape (3,2)
# [[1,4], [2,5], [3,6]]
```

---

## Step Slicing & Downsampling

```python
img[::2, ::2]          # every 2nd row and column → (H/2, W/2, 3)
img[1::2, 1::2]        # same but starting from index 1
img[100:400:2, 200:500:2]  # crop + downsample → shape (150, 150, 3)
```

This is **primitive downsampling** — it produces lower resolution with a pixelated appearance because pixels are skipped rather than averaged.

---

## Boolean Masks & Conditional Editing

### Creating masks

```python
mask = img[:, :, 0] > 200        # red channel > 200 → shape (H, W), dtype bool
mask = img[:, :, 1] > 150        # strong green pixels
```

### Combining masks

```python
mask = (img[:,:,0] > 200) & (img[:,:,1] < 50)   # high red AND low green
mask = (img[:,:,0] > 100) | (img[:,:,2] > 100)  # high red OR high blue
mask = ~mask                                    # invert (NOT)
```

### Applying masks

```python
img[mask]              # → shape (num_true_pixels, 3) — flattens geometry
img[mask] = [255,0,0]  # set matching pixels to red
img[~mask] = 0         # set non-matching pixels to black, ~ means logical not
img[mask] *= [1, 0, 0] # keep only red in matching pixels
```

> **Mental model:** Boolean indexing behaves like a database `WHERE` clause — "give me pixels satisfying this condition." The spatial structure is flattened; you get a list of matching pixels.

### Image binarization

```python
img_binary = (img > 128) * 255   # threshold to black/white
```

---

## Broadcasting In Depth

Broadcasting lets smaller arrays automatically expand to match larger ones. Dimensions are compared **right to left**.

**Compatibility rule:** Dimensions are compatible if they are equal, or one of them is `1`.

```
(H, W, 3)
(      3)   → broadcasts: (1,1,3) expanded to (H,W,3) ✓

(4, 5)
(   5)      → broadcasts: (1,5) expanded to (4,5) ✓

(4, 5)
(4, 1)      → broadcasts: each row value expands across columns ✓
```

### Horizontal gradient in red channel (broadcast across rows)

```python
# img.shape == (4, 5, 3)
img[:, :, 0] = [10, 20, 30, 40, 50]  # shape (5,) → (4,5)
# Result: same row pattern repeated across all rows
```

### Vertical gradient (broadcast across columns)

```python
img[:, :, 0] = [[100], [200], [50], [0]]  # shape (4,1) → (4,5)
# Result: each row gets uniform value → horizontal stripes
```

### Channel filtering & scaling

```python
img *= [1, 0, 0]     # keep red only: [R,G,B] → [R,0,0]
img *= [0.5, 1, 1]   # halve red:     [R,G,B] → [R/2,G,B]
```

---

## Reduction Operations & keepdims

Reduction operations (`mean`, `sum`, `max`, `min`) **collapse** the reduced axis by default.

```python
# Grayscale conversion via channel mean
gray = img.mean(axis=2)          # shape (H, W, 3) → (H, W)
gray = img.sum(axis=2) / (255*3) # alternative normalization
```

### keepdims — preserve broadcasting compatibility

```python
gray = img.mean(axis=2, keepdims=True)   # shape (H, W, 1)
```

With `keepdims=True`, the size-1 axis remains, making broadcasting back possible:

```python
# Subtract average brightness (normalize)
img - gray   # (H,W,3) - (H,W,1) → broadcasts along channel axis
```

This pattern removes overall brightness and emphasizes **relative color differences**. It appears constantly in normalization, machine learning preprocessing, and feature extraction.

---

## Procedural Image Generation

Images can be created entirely from mathematical patterns — no photo needed.

```python
H, W = 256, 512
img = np.zeros((H, W, 3), dtype=np.uint8)
```

### Negative of an image

```python
img_neg = 255 - img            # RGB negative
gray = img.sum(2) / (255*3)
gray_neg = 255*3 - gray        # grayscale negative (sum-based)
```

### Color reduction (quantization)

```python
img_64  = (img // 64) * 64    # 4 levels per channel
img_128 = (img // 128) * 128  # 2 levels per channel
```

### Padding with black border

```python
img_grey = img.sum(2) / (255*3)
padded = np.pad(img_grey, ((100, 100), (100, 100)), mode='constant')
# mode='constant' defaults to 0 (black)
```

---

## Gradients with linspace + tile

### `np.linspace`

```python
np.linspace(0, 255, 5)    # → [0., 63.75, 127.5, 191.25, 255.]
np.linspace(0, 10, 3)     # → [0., 5., 10.]
```

Creates evenly spaced values — the foundation for gradients.

### `np.tile`

```python
x = np.arange(5).reshape(1, -1)   # shape (1, 5)
np.tile(x, (3, 1))   # repeat 3 times along rows → shape (3, 5)
np.tile(x, (2, 2))   # repeat in both axes → shape (2, 10)
```

### Horizontal gradient

```python
gradient = np.tile(np.linspace(0, 255, W), (H, 1))
# Brightness varies left → right, same pattern each row
```

### Vertical gradient

```python
gradient = np.tile(np.linspace(0, 255, H).reshape(-1, 1), (1, W))
# Brightness varies top → bottom, same pattern each column
```

> **Key shape insight:** `(N,)` vs `(N,1)` behaves completely differently in broadcasting. The reshape controls which axis the variation occurs along.

### Full gradient image (multi-channel)

```python
def gradation_2d(start, stop, width, height, is_horizontal):
    if is_horizontal:
        return np.tile(np.linspace(start, stop, width), (height, 1))
    else:
        return np.tile(np.linspace(start, stop, height), (width, 1)).T

def gradation_3d(width, height, start_list, stop_list, is_horizontal_list):
    result = np.zeros((height, width, len(start_list)), dtype=np.float64)
    for i, (start, stop, is_h) in enumerate(zip(start_list, stop_list, is_horizontal_list)):
        result[:, :, i] = gradation_2d(start, stop, width, height, is_h)
    return result

# Example: horizontal white-to-black gradient
img = np.uint8(gradation_3d(256, 256, (0,0,0), (255,255,255), (True,True,True)))
```

---

## Coordinate Grids & meshgrid

### `np.meshgrid`

Converts 1D coordinate vectors into full 2D coordinate maps.

```python
x = [0, 1, 2]
y = [10, 20]
X, Y = np.meshgrid(x, y)
```

```
X = [[0, 1, 2],    Y = [[10, 10, 10],
     [0, 1, 2]]         [20, 20, 20]]
```

Both have shape `(len(y), len(x))` = `(2, 3)`.

- **`X`** — x-coordinates vary across **columns**
- **`Y`** — y-coordinates vary across **rows**

### For a full image

```python
H, W = img.shape[:2]
cx, cy = W // 2, H // 2   # center

x = np.arange(W)
y = np.arange(H)
X, Y = np.meshgrid(x, y)  # both shape (H, W)
```

Now every pixel knows its `(x, y)` position. Images can be generated from **equations on coordinates**.

---

## Distance Maps & Circular Masks

### Distance from center (vectorized)

```python
dist = np.sqrt((X - cx)**2 + (Y - cy)**2)   # shape (H, W)
```

No loops needed. NumPy computes the Euclidean distance for every pixel simultaneously. Pixels equidistant from center form perfect circles.

### Circular mask

```python
radius = 200
mask = dist < radius         # True inside circle, False outside
```

### Applying the mask

```python
img[~mask] = 0               # black out everything outside circle → spotlight effect
img[mask] = [255, 0, 0]      # fill circle with red
```

### Blending with a mask (soft mask)

```python
# Build a rectangular highlight mask
ones  = np.ones((img.shape[0] // 2, img.shape[1] // 2, 3))
zeros = np.zeros((img.shape[0] // 4, img.shape[1] // 4, 3))
zeros_mid = np.zeros((img.shape[0] // 2, img.shape[1] // 4, 3))

up     = np.concatenate((zeros, zeros, zeros, zeros), axis=1)
middle = np.concatenate((zeros_mid, ones, zeros_mid), axis=1)
down   = np.concatenate((zeros, zeros, zeros, zeros), axis=1)

mask_float = np.concatenate((up, middle, down), axis=0) / 255.0
img_masked = (mask_float * img).astype(np.uint8)
```

---

## Fancy Indexing

Selects **arbitrary, non-contiguous** elements in a specified order.

```python
arr = np.array([10, 20, 30, 40, 50])
arr[[3, 0, 2]]    # → [40, 10, 30]
```

Contrast with slicing:

| Method         | Selects             | Creates  |
| -------------- | ------------------- | -------- |
| `arr[1:4]`     | Contiguous range    | View     |
| `arr[[3,0,2]]` | Arbitrary positions | **Copy** |

> **Fancy indexing always creates a copy** because rearrangement may be required in memory.

---

### Image pasting via slice assignment

```python
src = np.array(Image.open('small.jpg').resize((128, 128)))
dst = np.array(Image.open('large.jpg').resize((256, 256)))

dst_copy = dst.copy()
dst_copy[64:192, 64:192] = src   # shapes must match exactly!
```

If shapes don't match: `ValueError: could not broadcast input array from shape...`

### Blending two images

```python
img_a = np.array(Image.open('a.jpg'))
img_b = np.array(Image.open('b.jpg').resize(img_a.shape[1::-1]))  # resize takes (W, H)

blended = (img_a * 0.6 + img_b * 0.4).astype(np.uint8)
```

> **Note:** Multiply first, then cast back to `uint8` — casting before arithmetic causes overflow.

### Pixel intensity histogram

```python
img_flat = img.flatten()   # (H*W*3,) — all pixel channel values
plt.hist(img_flat, bins=200, range=[0, 256])
plt.title("Pixel intensity distribution")
plt.xlabel("Intensity value")
plt.ylabel("Count")
plt.show()
```

### Side-by-side display helper

```python
combined = np.concatenate((img1, img2, img3), axis=1)  # horizontal
combined = np.concatenate((img1, img2), axis=0)         # vertical
plt.figure(figsize=(15, 5))
plt.imshow(combined)
```

---

## Key Mental Models Cheatsheet

```
Images are arrays            → pixels are numbers, not pictures
Channels are just axes       → axis 2 is RGB, nothing magic
Cropping is slicing          → img[r1:r2, c1:c2]
Color editing = assignment   → img[:,:,0] = 0
Rotation = transpose + flip  → axis manipulation, not image rotation
Flipping = reversing indices → img[::-1] or img[:,::-1]
Broadcasting = auto-expand   → smaller arrays repeat to match shape
Views share memory           → edits to slice edit original
.copy() = independence       → safe when you need isolation
Masks = boolean arrays       → True/False per pixel
Boolean indexing = filtering → flattens spatial structure
Integer index removes axis   → img[:,:,0] → (H,W)
Slice index keeps axis       → img[:,:,0:1] → (H,W,1)
keepdims preserves shape     → critical for broadcasting back
Fancy indexing = arbitrary   → always makes a copy
Transpose ≠ rotate           → it reorders axis interpretation
```

### Important syntax summary

```python
img[row, col, channel]      # pixel access
img[:, :, 0]                # red channel
img[:, :, ::-1]             # BGR ↔ RGB
img[100:400, 200:500]       # crop
img[::-1]                   # vertical flip
img[:, ::-1]                # horizontal flip
img.transpose(1, 0, 2)      # swap H and W
img.transpose(2, 0, 1)      # HWC → CHW (PyTorch format)
np.rot90(img)               # 90° CCW
img[::2, ::2]               # downsample 2×
mask = img[:,:,0] > 200     # boolean mask
img[mask] = [255, 0, 0]     # conditional edit
img.mean(axis=2, keepdims=True)  # grayscale, broadcast-safe
(img // 64) * 64            # color quantization
255 - img                   # invert / negative
```