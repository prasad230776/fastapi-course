# Pandas for Machine Learning: The Essential Python Data Guide
## Comprehensive Tutorial Notes

This study guide provides a detailed breakdown of the essential Pandas operations for Data Science, Machine Learning, and Generative AI, complete with clear, copy-pasteable code examples. All concepts and examples are grounded directly in the video crash course tutorial.

---

## 1. Introduction to Pandas

### Need and Overview of Pandas
* **The Data Science Bottleneck:** Real-world data is rarely immediately usable. It is typically massive, unstructured, noisy, and unclean [1]. 
* **The Role of Pandas:** Instead of writing boilerplate code to read, clean, restructure, and analyze data from scratch [2], Pandas serves as an open-source Python library that provides highly optimized data structures and functions [2]. 
* **Prerequisite for ML:** Knowing Pandas is an essential step before diving into Machine Learning and Generative AI [3].

### Importing Pandas
By convention, Pandas is imported using the standard alias `pd` [6]:
```python
import pandas as pd
```

### Setup and Installation
* **Cloud Environments (Recommended for Beginners):** Google Colab runs on the cloud, requiring zero local setup, while tracking and saving progress automatically [3].
* **Local Environments:** You can install Jupyter Notebook, VS Code, or PyCharm [4]. To get started with Jupyter Notebook, install it via pip:
```bash
pip install notebook
``` [4]

---

## 2. Pandas Data Structures

Pandas has two primary data structures: **Series** (1D) and **DataFrame** (2D) [6, 8].

### Series
A **Series** is a 1-dimensional array-like object [6]. While it looks like a Python list, it is built with optimized functions (like calculating `mean()`, `median()`, and `sum()`) that standard Python lists lack [7].
```python
import pandas as pd

# Creating a Series from a list
data_list = [1, 2, 3, 4, 5]
series = pd.Series(data_list)

# Creating a Series with custom label indices
custom_labels = ['a', 'b', 'c', 'd', 'e']
series_with_labels = pd.Series(data_list, index=custom_labels)

print(series_with_labels)
# Output:
# a    1
# b    2
# ...
``` [6, 7]

### DataFrame
A **DataFrame** is a 2-dimensional, table-like structure with labeled rows and columns [8]. It can be thought of as a spreadsheet or a SQL table, offering rich tools for filtering, manipulation, and analysis [8].

### DataFrame vs 2D Arrays
* **Heterogeneous Data:** A DataFrame can hold multiple different data types across different columns (e.g., one column of `str`, another of `int`, and another of `float`) [8, 9].
* **Homogeneous Data:** A traditional 2D array (such as in NumPy) is limited to holding a single, uniform data type across all elements [9].

---

## 3. Creating DataFrames

### Creating DataFrames from Different Sources
You can construct DataFrames programmatically from nested Python lists or dictionaries [9].
```python
# Programmatically creating a simple DataFrame with custom row and column labels
data = [[1, 2, 3]]
df = pd.DataFrame(data, columns=['c0', 'C1', 'C2'], index=['r0'])
print(df)
``` [9, 10]

### Common File Formats for Datasets
* **CSV, Excel, JSON:** Universal and compatible, but often slow and storage-heavy [2, 10]. Excel is especially bulky because it packs rich formatting and metadata [11].
* **Parquet and Feather:** Columnar binary formats optimized for big data [10, 11]. They employ built-in compression and are highly efficient [11]. E.g., a 1 GB CSV file can be compressed to ~100 MB in **Parquet** (Apache ecosystem) or ~200–300 MB in **Feather** [11].

### Reading Data from CSV Files
Pandas provides dedicated reading and exporting tools [12, 13]:
```python
# Reading a CSV file
obesity_data = pd.read_csv('obesity prediction.csv')

# Alternative readers:
# pd.read_excel('file.xlsx')
# pd.read_json('file.json')
# pd.read_parquet('file.parquet')
# pd.read_feather('file.feather')

# Exporting data:
# obesity_data.to_csv('output.csv', index=False)
# obesity_data.to_feather('output.feather')
``` [12, 13]

---

## 4. Exploring Data

### Viewing Data (head(), tail(), sample())
* **`head(n)`:** Returns the first `n` rows (defaults to 5) to verify data is loaded properly [13, 14].
* **`tail(n)`:** Returns the last `n` rows (defaults to 5) [13].
* **`sample(n, frac, random_state)`:** Selects random rows. This is superior to `head()` and `tail()` for checking sorted datasets [14, 15].
  * `n`: exact number of rows [15].
  * `frac`: fraction of rows (e.g., `frac=0.1` is 10% of the dataset) [15].
  * `random_state`: set to an integer seed to guarantee the same random row selection across runs [16].

```python
# Viewing top 3 rows
print(obesity_data.head(3))

# Viewing bottom 15 rows
print(obesity_data.tail(15))

# Getting a reproducible sample of 10 random rows
sample_df = obesity_data.sample(n=10, random_state=29)
``` [14, 15, 16]

### DataFrame Information
* **Labels (Index & Columns):** Access labels directly as properties [17, 18].
* **Shape:** Returns a tuple indicating dimensionality `(rows, columns)` [19].
* **Size:** Returns an integer of total elements (rows × columns) [19].
* **`info()`:** Lists column names, non-null counts, indices, data types, and memory usage [20].
* **`describe()`:** Computes descriptive statistics (mean, median, standard deviation, min, max, percentiles) for numerical columns [19, 20].

```python
# Inspecting labels
columns_list = obesity_data.columns.tolist()  # Converts columns to standard list
index_list = obesity_data.index.tolist()      # Converts indices to list

# Attributes (No parentheses)
print(obesity_data.shape)  # e.g., (2111, 17)
print(obesity_data.size)   # total number of elements

# Metadata Functions
obesity_data.info()
print(obesity_data.describe())
``` [17, 18, 19, 20]

---

## 5. Accessing Data

### Row and Column Selection

#### 1. `loc[]` (Label-Based Indexing)
Accesses rows/columns by their label names [21, 22]. **Slicing with `loc` is fully inclusive of both endpoints** [23].
```python
# Accessing cell at row label 0, column 'age'
age_val = obesity_data.loc[0, 'age']

# Slicing rows 0 to 5 (inclusive) and columns 'age' through 'weight'
subset_loc = obesity_data.loc[0:5, 'age':'weight']

# Selecting specific rows and columns via lists
list_subset = obesity_data.loc[[0, 7, 10], ['age', 'height', 'weight']]

# Selecting all rows for specified columns
all_rows_subset = obesity_data.loc[:, 'age':'weight']
``` [22, 23, 24]

#### 2. `iloc[]` (Integer Position-Based Indexing)
Accesses rows/columns by their integer index positions [25, 26]. **Slicing with `iloc` is exclusive of the upper bound** [26].
```python
# Slicing rows 0 to 9 and columns 0 to 4 (upper bound 10 and 5 are exclusive)
subset_iloc = obesity_data.iloc[0:10, 0:5]

# Specific coordinates: rows 10, 20, 30 and columns 0, 1, 2
coords_iloc = obesity_data.iloc[[10, 20, 30], [0, 1, 2]]
``` [26, 27]

#### 3. `at[]` (Optimized Label-Based Scalar Access)
Designed to quickly fetch or update a single, specific value [27, 28]. Built on top of NumPy, it bypasses safety checks and is faster than `.loc[]` [28]. Slices are not supported [28].
```python
# Fetching scalar value
scalar_val = obesity_data.at[0, 'age']
``` [28]

#### 4. `iat[]` (Optimized Integer-Based Scalar Access)
Bypasses overhead checks to retrieve a single value by integer index [27, 29]. Requires both row and column indices [30].
```python
# Fetching scalar value at row index 5, column index 0
scalar_val_iat = obesity_data.iat[5, 0]
``` [29, 30]

### Accessing Columns: Shorthand vs Dot Notation
Columns can be queried using brackets or attributes [31]:
* **Bracket (Shorthand) Notation:** `obesity_data['family_history']` [31]. For multiple columns, pass a nested list: `obesity_data[['age', 'weight']]` [32]. Highly recommended because it easily handles column names with spaces or special characters [31, 32].
* **Dot Notation:** `obesity_data.family_history` [31]. Fails when column names contain spaces (e.g., `family history` instead of `family_history`) [31, 32].

---

## 6. Filtering Data

### Filtering with Conditions (Boolean Indexing)
By applying comparison operations to columns, Pandas generates a "Boolean Mask" of True/False values, which `.loc[]` uses to extract corresponding rows [34].
```python
# Filter rows where height is greater than 1 meter
tall_folks = obesity_data[obesity_data['height'] > 1.0]
# Also valid: obesity_data.loc[obesity_data['height'] > 1.0]
``` [35, 38]

### Multiple Conditions
Combine multiple conditional filters using bitwise operators:
* `&` for AND (all conditions must be true) [36, 37]
* `|` for OR (at least one condition must be true) [37]
* **Crucial Rule:** Each conditional block **must** be enclosed in parentheses `()` to maintain correct order of operations [36].
```python
# AND: Weight less than 50 AND Normal weight category
filtered_and = obesity_data[(obesity_data['weight'] < 50) & (obesity_data['category'] == 'normal weight')]

# OR: Weight less than 50 OR Normal weight category
filtered_or = obesity_data[(obesity_data['weight'] < 50) | (obesity_data['category'] == 'normal weight')]
``` [36, 37]

### Regular Expressions (Regex)
You can search text patterns inside object/string columns using `.str` methods [39].
```python
# Find rows where category column contains the substring "normal"
normal_weight_df = obesity_data[obesity_data['category'].str.contains('normal', regex=True)]

# Find rows where category column starts with "normal"
starts_with_df = obesity_data[obesity_data['category'].str.startswith('normal')]
``` [39, 40]

---

## 7. Updating and Transforming Data

All updates modify the original DataFrame structure [41, 42].

### Updating Data using `loc[]`
Update values across specific labels [41]:
```python
# Update single scalar value
obesity_data.loc[0, 'age'] = 22

# Update entire column values
obesity_data.loc[:, 'smoke'] = 'yes'

# Update range of rows for a column
obesity_data.loc[0:2, 'smoke'] = 'yes'

# Update multiple specified rows
obesity_data.loc[[2, 3], 'height'] = 1.6
``` [41, 42, 44]

### Updating using `iloc[]`, `at[]`, `iat[]`
```python
# iloc[]: Update value at row position 1, column position 4
obesity_data.iloc[1, 4] = 'no_data'

# at[]: Label-based single value update
obesity_data.at[2, 'category'] = 'overweight'

# iat[]: Integer-position single value update
obesity_data.iat[0, 0] = 'secret'
``` [44, 45, 46]

### Transforming Data with `apply()`
`.apply()` allows running a custom function along an entire row or column [47, 48]. By default, it runs column-wise (`axis=0`) [48, 49].
* **Assignment is Required:** By default, `.apply()` does not edit in-place; you must assign it back [49, 50].

```python
# Defining a transformation function
def increment_age_by_five(x):
    return x + 5

# Transforming and assigning the changes back
obesity_data['age'] = obesity_data['age'].apply(increment_age_by_five)
``` [49, 50]

### Using Lambda Functions with `apply()`
For simple, temporary transformations, write a short, anonymous **lambda function** directly inside `.apply()` to avoid writing a full function definition [51].
```python
# Simple lambda subtraction
obesity_data['age'] = obesity_data['age'].apply(lambda x: x - 5)

# Adding a new column with custom conditional lambda logic
obesity_data['age_category'] = obesity_data['age'].apply(
    lambda x: 'very young' if x < 25 else 'mature'
)
``` [51, 52, 53]

### Transforming Data with `where()` (Vectorized Alternative)
For simple conditions, NumPy's vectorized function `np.where(condition, value_if_true, value_if_false)` is significantly faster and more computationally efficient than `.apply()` [54].
```python
import numpy as np

# Highly efficient conditional column creation
obesity_data['new_age_category'] = np.where(
    obesity_data['age'] < 25, 
    'very young', 
    'not so young'
)
``` [54, 55]

---

## 8. Column Operations

### Inserting Columns
Use `.insert(loc, column, value)` to add a new column at a specific index location [55].
```python
# Insert column 'BMI' at column position index 2, with value 20 for all rows
obesity_data.insert(2, 'BMI', 20)

# Inserting with calculations based on existing columns
obesity_data.insert(
    2, 
    'calculated_BMI', 
    obesity_data['weight'] / (obesity_data['height'] ** 2)
)
``` [55, 56, 57, 58]

### Dropping Columns
The `.drop(columns=[...])` method does not modify the original data unless you explicitly assign it back or set `inplace=True` [56, 57, 58].
```python
# Option 1: Using inplace=True
obesity_data.drop(columns=['BMI'], inplace=True)

# Option 2: Assignment back to the original variable
obesity_data = obesity_data.drop(columns=['BMI', 'calculated_BMI'])
``` [57, 58, 59]

### Deleting Columns
Use Python's built-in `del` keyword to instantly remove the column in-place [60].
```python
# Instant, in-place deletion
del obesity_data['new_age_category']
``` [60]

### Renaming Columns
Use `.rename(columns={old_name: new_name})` [60]. Requires `inplace=True` or assignment [60, 61].
```python
# Renaming multiple columns in-place
obesity_data.rename(
    columns={
        'obesity': 'category',
        'smoke': 'smoker_status'
    }, 
    inplace=True
)
``` [61]

---

## 9. Combining Data

### Merging DataFrames (SQL Joins)
You can perform relational merges on common key columns using `pd.merge(left_df, right_df, how, on)` [62, 63].

**Setup Example DataFrames:**
```python
df1 = pd.DataFrame({
    'ID': [1, 2, 3, 4], 
    'name': ['Alice', 'Bob', 'Charlie', 'David']
})

df2 = pd.DataFrame({
    'ID': [3, 4, 5, 6], 
    'course': ['Math', 'Science', 'History', 'Art']
})
``` [62, 63]

* **Inner Join (`how='inner'`):** Returns only rows where keys match in both DataFrames (intersection) [63, 64].
  ```python
  inner_df = pd.merge(df1, df2, how='inner', on='ID')
  # Output contains IDs 3 & 4
  ``` [64]
* **Outer Join (`how='outer'`):** Returns all rows, inserting `NaN` where matches are missing (union) [64, 65].
  ```python
  outer_df = pd.merge(df1, df2, how='outer', on='ID')
  ``` [64]
* **Left Join (`how='left'`):** Retains all rows from the left DataFrame (`df1`), introducing `NaN` for missing right matches [65].
  ```python
  left_df = pd.merge(df1, df2, how='left', on='ID')
  ``` [65]
* **Right Join (`how='right'`):** Retains all rows from the right DataFrame (`df2`), introducing `NaN` for missing left matches [65, 66].
  ```python
  right_df = pd.merge(df1, df2, how='right', on='ID')
  ``` [65, 66]

#### Different Column Names
If the key columns are named differently (e.g., `ID_1` and `ID_2`), use `left_on` and `right_on` [66, 67]:
```python
different_keys_df = pd.merge(
    df1, df2, 
    how='inner', 
    left_on='ID_1', 
    right_on='ID_2'
)
``` [67]

### Concatenating DataFrames (Stacking)
`pd.concat([df1, df2], axis)` stacks DataFrames vertically or horizontally [68, 69].

* **Vertical Stacking (`axis=0`):** Stack rows on top of each other [69].
  ```python
  # reset index from 0 to N instead of repeating original row indices
  vert_df = pd.concat([df1, df2], axis=0, ignore_index=True)
  
  # Keep track of source data frames using multi-level (hierarchical) indexing
  hierarchical_df = pd.concat([df1, df2], axis=0, keys=['df1', 'df2'])
  ``` [69, 70]
* **Horizontal Stacking (`axis=1`):** Stack columns side-by-side [70].
  ```python
  horiz_df = pd.concat([df1, df2], axis=1)
  ``` [70]

---

## 10. Handling Missing Values

`NaN` (Not a Number) represents missing or undefined values in datasets [64, 65]. 

### Detecting Missing Values
* **`isna()`:** Returns Boolean mask where `True` marks missing values [71, 72].
* **`notna()`:** Returns Boolean mask where `True` marks populated values [72].
```python
# Check total nulls per column
print(obesity_data.isna().sum())

# Check total non-nulls per column
print(obesity_data.notna().sum())
``` [72, 73]

### Filling Missing Values (`fillna()`)
Replace null values with meaningful data [74]. Requires assignment or `inplace=True` [74, 75].
```python
# Example: Injecting NaNs to practice filling
import numpy as np
obesity_data.loc[0, 'age'] = np.nan
obesity_data.loc[0, 'smoke'] = np.nan

# 1. Fill ALL missing cells in the entire DataFrame with 0
obesity_data.fillna(0, inplace=True)

# 2. Fill single column with static scalar
obesity_data['age'].fillna(20, inplace=True)

# 3. Fill column with column Mean (Highly common in ML preprocessing)
obesity_data['age'].fillna(obesity_data['age'].mean(), inplace=True)

# 4. Fill multiple different columns with unique values
obesity_data.fillna(
    value={
        'smoke': 'no', 
        'family_history': 'no'
    }, 
    inplace=True
)

# 5. Interpolate: Fill values using neighbor patterns/mean (useful for sequential data)
obesity_data['age'].interpolate(inplace=True)
``` [73, 74, 75, 76, 77, 78, 79]

### Removing Missing Values (`dropna()`)
Completely drop rows or columns containing missing values [80, 81].
```python
# Drop any row containing at least one missing value
obesity_data.dropna(axis=0, inplace=True)

# Drop any column containing at least one missing value
obesity_data.dropna(axis=1, inplace=True)
``` [80, 81]

---

## 11. Data Aggregation

### Grouping Data using `groupby()`
Group row records by unique category values to analyze statistics within subgroups (e.g., comparing height across genders) [81, 82].
```python
# 1. Group by gender and find mean height
print(obesity_data.groupby('gender')['height'].mean())

# 2. Group by gender and find median weight
print(obesity_data.groupby('gender')['weight'].median())

# 3. Apply multiple aggregations at once using .agg()
aggregated_stats = obesity_data.groupby('gender')['height'].agg(['sum', 'mean'])
print(aggregated_stats)
``` [82, 83]