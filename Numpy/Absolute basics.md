## Installation
```
pip install numpy
```

## How to import NumPy
```bash
import numpy as np
```

## What is an “array"
An array is a structure for storing and retrieving data. We might visualize a "one-dimensional" array like a list:
/
![](../assets/numpy_one_dimensional_array.png)

A two-dimensional array would be like a table:
/
![](../assets/numpy_two_dimensional_array.png)


A three-dimensional array would be like a set of tables, perhaps stacked as though they were printed on separate pages. In `NumPy`, this idea is generalized to an arbitrary number of dimensions, and so the fundamental array is called `ndarray`: it represents an "N-dimensional array".

Most NumPy arrays have some restrictions. For instance:
- All elements of the array must be of the same type of data.
- Once created, the total size of the array can’t change.
- The shape must be “rectangular”, not “jagged”; e.g., each row of a two-dimensional array must have the same number of columns.

	`>>>` refers to python code. If there is no prefix, it means output of the above command.

## Array fundamentals
One way to initialize an array is using a Python sequence, such as a list. For example:
```python
>>> a = np.array([1, 2, 3, 4, 5, 6])
>>> a
array([1, 2, 3, 4, 5, 6])
```
We can access array elements using bracket notation `[idx]`

Like the original list, the array is mutable.
```python
>>> a[0] = 10
array([10,  2,  3,  4,  5,  6])
```

Also like the original list, Python slice notation can be used for indexing.
```python
>>> a[:3]
array([10, 2, 3])
```

One major difference is that slice indexing of a list copies the elements into a new list, but slicing an array returns a `view`: an object that refers to the data in the original array. The original array can be mutated using the view.
```python
>>> b = a[3:]
>>> b
array([4, 5, 6])
b[0] = 40
a
array([10, 2, 3, 40, 5, 6])
```

Two- and higher-dimensional arrays can be initialized from nested Python sequences:
```python
>>> a = np.array([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]])
>>> a
array([[ 1,  2,  3,  4],
       [ 5,  6,  7,  8],
       [ 9, 10, 11, 12]])
```

## Array Attributes
The number of dimensions of an array is contained in the `ndim` attribute.
```python
>>> a.ndim
2
```

The shape of an array is a tuple of non-negative integers that specify the number of elements along each dimension.
```python
>>> a.shape
(3, 4)
>>> len(a.shape) == a.ndim
True
```

The fixed, total number of elements in array is contained in the `size` attribute.
```python
>>> a.size
12
>>> import math
>>> a.size == math.prod(a.shape)
True
```

Arrays are typically "homogeneous", meaning that they contain elements of only one "data type". The data type is recorded in the `dtype` attribute.
```python
>>> a.dtype
dtype('int64') # 'int' for integer, '64' for 64-bit
```

## How to create a basic array

Besides creating an array from sequence of elements, you can easily create an array filled with `0`'s:
```python
>>> np.zeroes(2)
array([0., 0.])
```

Or an array filled with `1`'s:
```python
>>> np.ones(2)
array([1., 1.])
```

Or even an empty array! The function `empty` creates an array whose initial content is random and depends on the state of the memory. The reason to use `empty` over `zeroes` is speed.
```python
>>> # Create an empty array with 2 elements
>>> np.empty(2) 
array([3.14, 42. ]) # may vary
```

You can create an array with a range of elements:
```python
>>> np.arange(4) # Starts elements from zero
array([0, 1, 2, 3])
```

And even an array that contains a range of evenly spaced intervals. To do this, you will specify the **first number**, **last number**, and the **step size**.
```python
>>> np.arange(2, 9, 2)
array([2, 4, 6, 8])
```

You can also use `np.linspace` to create an array with values that are spaced linearly in a specified interval:
```python
>>> np.linspace(0, 10, num=5)
array([0., 2.5, 5., 7.5, 10.])
```

**Specifying your data type**
While the default data type is floating point ( `np.float64` ), you can explicitly specify which data type you want using the `dtype` keyword.
```python
>>> x = np.ones(2, dtype=np.int64)
>>> x
array([1, 1])
```

## Adding, removing, and sorting elements
Sorting an array is simple with `np.sort()`. You can specify the axis, kind, and order when you call the function.

If you start with this array:
```python
>>> arr = np.array([2, 1, 5, 3, 7, 4, 6, 8])
```
You can quickly sort the numbers in ascending order with:
```python
>>> np.sort(arr)
array([1, 2, 3, 4, 5, 6, 7, 8])
```

In addition to sort, which returns a sorted copy of an array, you can use:
- `argsort`, which is an indirect sort along a specified axis,
- `lexsort`, which is an indirect stable sort on multiple keys,
- `searchsorted`, which will find elements in a sorted array, and
- `partition`, which is a partial sort.

If you start with these arrays:
```python
>>> a = np.array([1, 2, 3, 4])
>>> b = np.array([5, 6, 7, 8])
```
You can concatenate them with `np.concatenate()`.
```python
>>> np.concatenate((a, b))
array([1, 2, 3, 4, 5, 6, 7, 8])
```

Or, if you start with these arrays:
```python
>>> x = np.array([[1, 2], [3, 4]])
>>> y = np.array([[5, 6]])
```

You can concatenate them with:
```python
>>> np.concatenate((x, y), axis=0)
array([[1, 2],
       [3, 4],
       [5, 6]])
```
In order to remove elements from an array, it's simple to use indexing to select the elements that you want to keep.

## How do you know the shape and size of an array?
`ndarray.ndim` will tell you the number of axes, or dimensions, of the array.
`ndarray.size` will tell you the total number of elements of the array. This is the product of the elements of the array's shape.
`ndarray.shape` will display a tuple of integers that indicate the number of elements stored along each dimension of the array. If, for example, you have a 2-D array with 2 rows and 3 columns, the shape of your array is ` (2, 3) `.

## Can you reshape an array?
Yes!
Using `arr.reshape()` will give a new shape to an array without changing the data. Just remember that when you use the reshape method, the array you want to produce needs to have the same number of elements as the original array. If you start with an array with 12 elements, you'll need to make sure that your new array also has a total of 12 elements.