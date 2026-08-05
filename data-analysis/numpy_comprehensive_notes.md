# NumPy Foundations: Comprehensive Reference Notes

This study guide provides structured reference notes on NumPy, strictly compiled from Keerti Purswani's *NumPy in One Hour* video tutorial. It contains foundational theory, implementation rules, and clear code examples.

---

## 1. Introduction

### Approach for Learning NumPy
* **Focus on Capabilities over Memorisation:** Avoid attempting to mug up or memorise specific syntaxes [3]. Instead, understand what operations and structures are supported, and refer to the official documentation when writing code [1, 3].
* **Active Parallel Coding:** Rather than passive watching, open tutorials and official documentation side-by-side with your environment. Write the code yourself to build confidence and muscle memory [2].
* **Project-Driven Learning:** The best way to internalise syntaxes is to work on end-to-end, hands-on projects where you write the same NumPy commands repeatedly [3].

### Installing and Running NumPy
* **Installation:** In your terminal or a Jupyter notebook cell, install the package using `pip` [6]:
  ```bash
  pip install numpy
  ```
* **Importing:** It is standard practice to import NumPy under the alias `np` to keep code clean and standardised [6]:
  ```python
  import numpy as np
  ```
* **Interactive Environments:** Using interactive environments like Jupyter notebooks allows you to execute blocks of code on-the-fly and check array shapes and dimensions immediately [6].

---

## 2. NumPy Fundamentals

### Why NumPy Arrays? (Need and Internals)
At the core of the NumPy library is the **NDArray** (N-Dimensional Array) object [10, 16]. Standard Python lists have significant performance bottlenecks when handling large-scale data [7, 8]:
1. **Homogeneity vs. Heterogeneity:** Standard Python lists can contain heterogeneous data types, while NDArrays are **homogeneous**—all elements must share the exact same data type (e.g., all integers, all floats, or all strings) [4, 5]. 
2. **The Cost of Looping in Python:** For large-scale math on millions of elements, Python lists require explicit, interpreted loops [8]. This incurs heavy overhead because Python continually interprets the code and manipulates individual Python objects in memory [9].
3. **The C-Speed Under the Hood:** NumPy operations are executed speedily at near-C speed [11]. This is because NumPy relies on optimized, precompiled C code under the hood, saving the interpreter's overhead and managing elements in contiguous memory blocks [1, 9, 10]. It gives you the "best of both worlds": the code simplicity of Python and the execution speed of C [9, 11].

### Vectorization
**Vectorization** is the absence of any explicit looping or indexing in your Python code [12]. 
* Instead of looping over elements manually (`for x in array`), you write standard mathematical notations (e.g., `C = A * B`) [10, 12].
* It produces concise, highly readable, and pythonic code with fewer lines and fewer bugs [11, 12].
* Looping is implicitly offloaded to precompiled C code, making operations incredibly efficient [10, 12].

### Broadcasting
**Broadcasting** describes the implicit element-by-element behaviour of array operations [13].
* By default, when an NDArray is involved, operations (arithmetic, logical, bitwise, or functional) happen element-wise [10, 13].
* If you operate on two arrays of different shapes, NumPy evaluates if their dimensions are compatible [14, 56]. 
* If they are compatible, NumPy expands the smaller array under the hood (without making actual copies in memory) until its shape matches the larger array, allowing the element-wise operation to proceed unambiguously [14, 56]. If they are incompatible, a `ValueError` is raised [56].

---

## 3. Creating Arrays

### Creating Arrays
To instantiate a standard array from a Python list or iterable, use `np.array()` [16]:
```python
# Creating a 1D Array (Vector)
a = np.array([1, 2, 3])
print(a)  # Output: [1 2 3]
```

### Zeros Array (`np.zeros`)
Creates an array filled entirely with zeros [26].
* **Usage:** Pass the desired shape as a single integer or a tuple [26, 27]. 
* *Note:* Do not pass multiple dimensions as flat arguments (e.g., `np.zeros(2, 3)`), as NumPy will attempt to interpret the second argument as a data type [27]. Always wrap multi-dimensional shapes in an outer tuple [27].
```python
# 1D array of zeros
zeros_1d = np.zeros(3)  # Output: [0. 0. 0.]

# 2D array of zeros (2 rows, 3 columns)
zeros_2d = np.zeros((2, 3)) 
```

### Ones Array (`np.ones`)
Creates an array filled entirely with ones, acting similarly to `np.zeros` [28].
```python
# 3D array of ones (6 blocks, 3 rows, 4 columns)
ones_3d = np.ones((6, 3, 4))
```

### Empty Array (`np.empty`)
Creates an uninitialised array of a specified shape [29].
* **How it works:** Instead of setting elements to zero or one, `np.empty` simply allocates the memory block and leaves whatever garbage values were already present in those memory addresses [29].
* **Why use it:** It is faster than `np.zeros`, `np.ones`, or `np.random` because it avoids the overhead of value initialisation [29, 30]. It is highly useful when you plan to immediately overwrite every element in the array anyway [29, 30].
```python
# Allocating a 3x8 uninitialised array
empty_arr = np.empty((3, 8))
```

### Range Creation (`np.arange`)
Generates sequences of numbers over a range [30].
* **Syntax:** `np.arange([start], stop, [step])` [31, 40].
* **Default values:** `start` defaults to `0`, and `step` defaults to `1` [40].
* **Inclusivity:** The `stop` value is **not inclusive** [31, 40].
```python
# Single argument: Treated as 'stop' (creates 0 to 5)
a = np.arange(6)  # [0, 1, 2, 3, 4, 5]

# Three arguments: start, stop, step
b = np.arange(2, 10, 2)  # [2, 4, 6, 8]  (10 is not included)

# Supports floating-point step values
c = np.arange(2, 4, 0.5)  # [2.0, 2.5, 3.0, 3.5]
```

### Linearly Spaced Arrays (`np.linspace`)
Generates a specified number of evenly spaced values over a specified interval [32].
* **Key Difference from `arange`:** Instead of specifying a step size, you specify the exact *number of elements* you want [32].
* **Defaults:** Returns elements as `float64` by default [32, 33]. You can explicitly override this using the `dtype` parameter [32, 33].
```python
# Generate 5 equally spaced values between 1 and 10
linspace_arr = np.linspace(1, 10, 5)  # [1. 3.25 5.5  7.75 10.]

# Generate spaced values as integers
linspace_int = np.linspace(1, 10, 5, dtype=np.int64)
```

### Random Number Generation (`np.random`)
Modern NumPy uses a default generator object for producing random numbers [62].
* To use it, instantiate the generator first using `np.random.default_rng()` [62].
* Generate random integers within a range using the `.integers(low, high, size)` method [62, 63].
```python
# Initialise the default random number generator
rng = np.random.default_rng()

# Generate a 3x2 array of random integers between 2 (inclusive) and 9 (exclusive)
random_ints = rng.integers(2, 9, size=(3, 2))
```

---

## 4. Array Properties

Knowing your array properties is essential for debugging structural errors and matching dimensional shapes.

| Property | Method/Attribute | Description | Example |
| :--- | :--- | :--- | :--- |
| **Dimensions** | `a.ndim` | Returns the number of axes (dimensions) of the array [18, 19]. | Scalar (0D): `0`<br>Vector (1D): `1`<br>Matrix (2D): `2`<br>Tensor (>2D): `3+` [25, 26] |
| **Shape** | `a.shape` | Returns a tuple of non-negative integers specifying elements along each axis [20]. | 2D matrix: `(rows, columns)` [6]<br>Scalar (0D): empty tuple `()` [20] |
| **Size** | `a.size` | Total number of elements. Always equals the product of shape elements [20, 22]. | Shape `(2, 4)` size: `8` [22] |
| **Data Type** | `a.dtype` | Identifies the homogeneous data type of the elements [26]. | e.g., `int64`, `float64` [26] |
| **Item Size** | `a.itemsize` | *Optional/Useful:* Length of one array element in bytes. | (Not detailed in video transcript) |
| **Total Bytes** | `a.nbytes` | *Optional/Useful:* Total bytes consumed by all array elements (`size * itemsize`). | (Not detailed in video transcript) |

*Mental Model for Multi-Dimensional Layouts:* In mathematical notation, we access a 2D matrix by row index first and column index second [25]. For higher dimensions, a helpful mental model is that the column index comes last, and the row index is second-to-last [25].

---

## 5. Indexing and Slicing

### Accessing Elements
NumPy is **zero-indexed** [16]. You access elements using square brackets [16].
* **Positive Indexing:** `a[0]` accesses the first element [16].
* **Negative Indexing:** `a[-1]` accesses the last element, and `a[-2]` accesses the second-to-last element [46].

### Modifying Elements
You can mutate array elements in place by assigning a value directly to a targeted index [17].
```python
a = np.array([10, 20, 30, 40])
a[3] = 13  # Modifies index 3 to 13
# Array is now: [10, 20, 30, 13]
```

### Basic Slicing
Extracts sections of an array using `[start:stop:step]` notation [17].
* Slices include the `start` index but exclude the `stop` index [17].
```python
arr = np.array([12, 89, 45, 67, 23])

# Slice up to (excluding) index 2
print(arr[:2])  # Output: [12 89]

# Slice from index 3 to the end
print(arr[3:])  # Output: [67 23]

# Slice excluding the last two elements
print(arr[:-2])  # Output: [12 89 45]
```

### Conditional (Boolean) Slicing
This is a powerful filtering technique where elements are selected based on logical conditions [47, 48].
1. **Generating a Boolean Mask:** Running an operator like `a < 6` returns an array of `True`/`False` flags corresponding to whether each element meets the condition [47].
2. **Filtering:** Passing this mask back into the array returns a flat, new array containing only the elements where the mask is `True` [47].
```python
a = np.array([3, 75, 6, 84, 2])

# Create a condition mask
mask = a < 7  # [True, False, True, False, True]

# Filter values
filtered_values = a[mask]  # Output: [3, 6, 2]
```
* **Multiple Conditions:** You can combine multiple logical conditions. You **must** wrap each condition in round brackets and use bitwise operators (`&` for AND, `|` for OR) [48, 49].
```python
# Select values less than 7 OR greater than 74
filtered_multi = a[(a < 7) | (a > 74)]  # Output: [3, 75, 6, 84, 2]
```
* **Retrieving Indices of Matches (`np.nonzero`):** If you want to find the **indices** where the condition is met instead of the values themselves, use `np.nonzero(condition)` [49]. It returns a tuple of arrays (one for each dimension) containing coordinates of matching elements [49, 50].

---

## 6. Array Manipulation

### Reshaping Arrays (`reshape`)
Changes the shape structure of an array without modifying its underlying data [37].
* **Sizing Constraint:** The total size (number of elements) in the reshaped array must exactly match the original array size, otherwise NumPy throws a `ValueError` [38].
```python
a = np.arange(6)  # Size = 6. Elements: [0, 1, 2, 3, 4, 5]

# Reshaping to 3 rows, 2 columns
reshaped_3x2 = a.reshape(3, 2)
```
* **Row-Major vs. Column-Major Order (`order`):** You can control how elements are read/placed in memory during a reshape [41, 42]:
  * `order='C'` (default, C-like order): Fills elements row-by-row [41, 42].
  * `order='F'` (Fortran-like order): Fills elements column-by-column [41, 42].
```python
# If you reshape a 3x2 array to a 2x3 using different orders:
arr = np.array([[0, 1], [2, 3], [4, 5]])

print(np.reshape(arr, (2, 3), order='C'))  
# Output: [[0 1 2]
#          [3 4 5]]

print(np.reshape(arr, (2, 3), order='F'))  
# Output: [[0 2 4]
#          [1 3 5]]  (Read down columns first)
```

### Adding New Dimensions (`np.newaxis`, `np.expand_dims`)
Allows you to insert an extra dimension to convert vectors into explicit 2D row/column matrices without altering the underlying data [43, 44].
```python
a = np.array([1, 2, 3, 4, 5, 6])  # Shape: (6,)

# 1. Using np.newaxis
row_vec_new = a[np.newaxis, :]    # Shape: (1, 6) (Row Vector) [44]
col_vec_new = a[:, np.newaxis]    # Shape: (6, 1) (Column Vector) [44]

# 2. Using np.expand_dims
row_vec_expand = np.expand_dims(a, axis=0)  # Shape: (1, 6) [44, 45]
col_vec_expand = np.expand_dims(a, axis=1)  # Shape: (6, 1) [44, 45]
```

### Flattening Arrays (`flatten`, `ravel`)
Converts a multi-dimensional array into a flat 1D array [67].

* **`ravel` (Shallow/View):** Returns a flattened view of the original array [67]. Changes made to the ravelled array **will** directly mutate the original parent array [67]. It is highly memory-efficient because no copy of the underlying data is made [67].
* **`flatten` (Deep Copy):** Returns a completely new 1D copy of the array [67, 68]. Modifying the flattened array will **not** affect the parent array [67, 68].

```python
parent = np.array([[1, 2], [3, 4]])

# Ravel modification:
rav = parent.ravel()
rav[0] = 99
print(parent[0, 0])  # Output: 99 (Parent array modified!) [67]

# Flatten modification:
flat = parent.flatten()
flat[1] = 88
print(parent[0, 1])  # Output: 2 (Parent array remains unchanged!) [67, 68]
```

### Transposing Arrays (`transpose`, `.T`)
Transposes the matrix by swapping row indices with column indices [63].
* Access via `.T` or the `.transpose()` method [64].
* Transposing twice returns the original array structure [64]. It does not modify the original parent array in place [64].

### Reversing/Flipping Arrays (`flip`, slicing)
Reverses the order of elements along axes [64, 65].
* **`np.flip(arr)`**: Flips the elements across all axes [64, 65].
* **Axis-Specific Flipping**: 
  * `np.flip(arr, axis=0)` reverses vertically along the rows [65].
  * `np.flip(arr, axis=1)` reverses horizontally along the columns [66].
* **Sub-array Flipping**: You can flip targeted portions. For example, `np.flip(arr[1])` reverses only the second row, while `np.flip(arr[:, 1])` reverses only the second column [66].

---

## 7. Combining and Splitting Arrays

### Concatenation (`concatenate`)
Combines multiple arrays along a specified axis [36].
* By default, `axis=0` is used, which vertically stacks them [36].
* **Constraint:** All arrays must have matching dimensions except along the concatenation axis, or NumPy will raise an error [36, 51, 52].
```python
# Vertically concatenating two compatible arrays
x = np.array([[1, 2], [3, 4]])
y = np.array([[5, 6]])
z = np.concatenate((x, y), axis=0)
```

### Vertical Stacking (`vstack`)
Stacks arrays on top of each other (vertically along `axis=0`) [51].
```python
v_stacked = np.vstack((a, b))
```

### Horizontal Stacking (`hstack`)
Stacks arrays adjacent to each other side-by-side (horizontally along `axis=1`) [51].
```python
h_stacked = np.hstack((a, b))
```

### Horizontal Splitting (`hsplit`)
Splits an array horizontally along columns [52].
* **Split into Equal Sections:** Pass an integer specifying the number of equal sub-arrays [52].
* **Split at Specific Coordinates:** Pass a list of column indices where cuts should happen [53].
```python
# x is an array with 12 columns
# 1. Split into 3 equal arrays:
part1, part2, part3 = np.hsplit(x, 3)  [52]

# 2. Split at specific column boundaries (e.g., after columns 3 and 4):
split_a, split_b, split_c = np.hsplit(x, [3, 4])  [53]
```

---

## 8. Sorting and Copying

### Sorting Arrays (`sort`, `argsort`, partition)
* **`np.sort`**: Returns a sorted copy of the array [33].
* **`np.argsort`**: Returns the indices that would sort the array, allowing you to perform indirect sorting [33, 34].
  ```python
  arr = np.array([30, 10, 20])
  indices = np.argsort(arr)  # Output: [1, 2, 0]
  # 10 is at index 1 (smallest), 20 is at index 2, 30 is at index 0 (largest) [33, 34]
  ```
* **`np.partition`**: Partitions an array around a specified index `k` [34, 35]. All elements smaller than the element at `k` are shuffled to the left, and all larger elements are moved to the right [35]. The elements on either side are not guaranteed to be sorted [35]. This is highly useful for top-k selection algorithms [35].
  ```python
  a = np.array([98, 90, 23, 12, 43])
  # Partition at index 3
  partitioned = np.partition(a, 3) 
  # Elements before partition index 3 are smaller than the element at 3 [35]
  ```

### Views vs. Copies
Understanding memory management in NumPy is critical to preventing unintended mutations.

```
       Original Array in Memory
             /          \
            /            \
    View (Shallow Copy)   Deep Copy (dot copy)
    - Created by slicing  - Created via .copy() [55]
      or indexing [55]     - Allocates separate memory [55]
    - Points to SAME      - Modifying does NOT [55]
      memory [53, 54]       affect original
    - Modifying changes
      original [54, 55]
```

* **Views (Shallow Copies):** To save memory and execution overhead, basic operations like slicing and indexing return **views** rather than copies [55]. If you modify a slice, the changes propagate to the original array [54, 55].
* **Deep Copies:** If you need a completely isolated array, call the `.copy()` method explicitly to allocate separate memory [55].

---

## 9. Mathematical Operations

### Arithmetic Operations
All default arithmetic operations are executed **element-by-element** [56].
* **Addition (`+`)**: Computes element-wise sum [56].
* **Subtraction (`-`)**: Computes element-wise difference [57].
* **Multiplication (`*`)**: Performs element-wise multiplication (this is *not* matrix multiplication) [57].
* **Division (`/`)**: Computes element-wise division [57, 58].

*Constraint:* The arrays must have compatible shapes for broadcasting, or a `ValueError` is raised [56, 57].

### Aggregate Functions
Aggregations can collapse an entire array or be calculated along specific dimensions using the `axis` parameter [58].
* **`axis=0` (Rows Axis):** Computes operations vertically down columns [58, 59].
* **`axis=1` (Columns Axis):** Computes operations horizontally across rows [59].

```python
a = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

# Sum
print(a.sum())          # 45 (sum of entire matrix) [58]
print(a.sum(axis=0))    # [12, 15, 18] (sum down each column: 1+4+7, 2+5+8, etc.) [59]
print(a.sum(axis=1))    # [6, 15, 24] (sum across each row: 1+2+3, 4+5+6, etc.) [59]

# Minimum / Maximum
print(a.min(axis=1))    # [1, 4, 7] (minimum value of each row) [60]
print(a.max(axis=0))    # [7, 8, 9] (maximum value of each column) [60]

# Product
print(a.prod(axis=0))   # [28, 80, 162] (product down each column: 1*4*7, 2*5*8, etc.) [60, 61]

# Mean
print(a.mean(axis=1))   # [2., 5., 8.] (mean value of each row) [61]

# Standard Deviation
print(a.std())          # Computes standard deviation across the entire array [61]
```
