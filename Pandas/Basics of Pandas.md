---
title: Basics of Pandas
source: https://www.kaggle.com/learn/pandas
created: 2026-07-02
tags:
  - pandas
  - python
---
# Creating, Reading and Writing

## Getting started

To use pandas, we can import it as follows:

```python
import pandas as pd
```

## Creating data

There are two core objects in pandas: the **DataFrame** and the **Series**.


### DataFrame

A DataFrame is a table. It contains an array of individual _entries_, each of which has a certain _value_. Each entry corresponds to a row (or _record_) and a _column_.

For example, consider the following simple DataFrame:

```python
pd.DataFrame({'Yes': [50, 21], 'No': [131, 2]})
```

|     | Yes | No  |
| --- | --- | --- |
| 0   | 50  | 131 |
| 1   | 21  | 2   |
In this example, the "0, No" entry has the value of 131. The "0, Yes" entry has a value of 50, and so on.

DataFrame entries are not limited to integers. You can store any data type in them.

We are using the `pd.DataFrame()` constructor to generate these DataFrame objects. The syntax for declaring a new one is a dictionary whose keys are the column names ( `Yes` and `No` in above example) , and whose values are a list of entries. This is the standard way of constructing a new DataFrame.

The dictionary-list constructor assigns values to the _column labels_, but just uses an ascending count from 0 (0, 1, 2, 3, ...) for the _row labels_. Sometimes this is OK, but oftentimes we will want to assign these labels ourselves.

The list of row labels used in a DataFrame is known as an **Index**. We can assign values to it by using an `index` parameter in our constructor:

```python
pd.DataFrame({'Bob': ['I liked it.', 'It was awful.'], 
              'Sue': ['Pretty good.', 'Bland.']},
             index=['Product A', 'Product B'])
```

Output:

|           | Bob           | Sue          |
| --------- | ------------- | ------------ |
| Product A | I liked it.   | Pretty good. |
| Product B | It was awful. | Bland.       |
### Series

A Series, by contrast, is a sequence of data values. If a DataFrame is a table, a Series is a list. And in fact you can create one with nothing more than a list:

```python
pd.Series([1, 2, 3, 4, 5])
```

Output:

| 0   | 1   |
| --- | --- |
| 1   | 2   |
| 2   | 3   |
| 3   | 4   |
| 4   | 5   |
A Series is, in essence, a single column of a DataFrame. So you can assign row labels to the Series the same way as before, using an `index` parameter. However, a Series does not have a column name, it only has one overall `name`:

```python
pd.Series([30, 35, 40], index=['2015 Sales', '2016 Sales', '2017 Sales'], name='Product A')
```

|            | Product A |
| ---------- | --------- |
| 2015 Sales | 30        |
| 2016 Sales | 35        |
| 2017 Sales | 40        |
The Series and the DataFrame are intimately related. It's helpful to think of a DataFrame as actually being just a bunch of Series "_glued together_".

## Reading data files

Data can be stored in any number of different forms and formats. The most widely used is the CSV files (excel sheets).

We can use the `pd.read_csv` function to read csv data into a DataFrame.

```python
wine_reviews = pd.read_csv("../input/wine-reviews/winemag-data-130k-v2.csv")
```

We can use the `shape` attribute to check how large the resulting DataFrame is:

```python
wine_reviews.shape
```

```
(129971, 14)
```

We can examine the contents of the resultant DataFrame using the `head()` command, which by default (you can change it) grabs the first five rows:

```python
wine_reviews.head()
```

The `pd.read_csv()` function is well-endowed, with over 30 optional parameters you can specify. By default, pandas don't pick up any index but uses an unnamed index which starts from 0 to number of rows. We can make sure that pandas use any column as any index using `index_col`

```python
wine_reviews = pd.read_csv("../input/wine-reviews/winemag-data-130k-v2.csv", index_col=0)

wine_reviews.head()
```
## Writing to Disk

We can also create a csv file from a DataFrame.

```python
animals.to_csv("cows_and_goats.csv")
```

Above we have a DataFrame stored in `animals` variable which we are saving to disk with a name `cows_and_goast.csv` 

# Indexing, Selecting & Assigning

## Native accessors

Native Python objects provide good ways of indexing data. Pandas carries all of these over, which helps make it easy to start with.

Consider a `reviews` DataFrame with columns like `country`, `description`, `points`.

In Python, we can access the property of an object by accessing it as an attribute. A `book` object, for example, might have a `title` property, which we can access by calling `book.title`. Columns in a pandas DataFrame work in much the same way.

Hence to access the `country` property of `reviews` we can use:

```python
reviews.country
```

If we have a Python dictionary, we can access its values using the indexing `[]` operator. We can do the same with columns in a DataFrame:

```python
reviews['country']
```

These are the two ways of selecting a specific Series out of a DataFrame. Neither of them is more or less syntactically valid than the other, but the indexing operator `[]` does have the advantage that it can handle column names with reserved characters in them (e.g. if we had a `country providence` column, `reviews.country providence` wouldn't work).

If we had to access anything under `conutry` column, we can use another `[]` operator.

```python
reviews['country'][0]
```

## Indexing in pandas

Indexing in pandas work similarly like python, you can use the `.` or `[` operator for accessing.

### Indexing-based selection

Pandas indexing works in one of two paradigms. The first is **index-based selection**: selecting data based on its numerical position in the data. `iloc` follows this paradigm.

To select the first row of the data in a DataFrame, we may use the following:

```python
reviews.iloc[0]
```

Both `loc` and `iloc` are row-first, column-second. This is the opposite of what we do in native python, which is column-first, row-second. 

This means that it's marginally easier to retrieve rows, and marginally harder to get retrieve columns. To get a column with `iloc`, we can do the following:

```python
reviews.iloc[:, 0]
```

On its own, the `:` operator, which also comes from native Python, means "everything". When combined with other selectors, however, it can be used to indicate a range of values. For example, to select the `country` column from just the first, second, and third row, we would do:

```python
reviews.iloc[:3, 0]
```

Or, to select just the second and third entries, we would do:

```python
reviews.iloc[1:3, 0]
```

It's also possible to pass a list:

```python
reviews.iloc[[0, 1, 2], 0]
```

We can also use negative indexes which will start counting from the back.

### Label-based selection

The second paradigm for attribute selection is the one followed by the `loc` operator: **label-based selection**. In this paradigm, it's the data index value, not its position, which matter.

For example, to get the first entry in `reviews`, we would now do the following:

```python
reviews.loc[0, 'country']
```

`iloc` is conceptually simpler than `loc` because it ignores the dataset's indices. When we use `iloc` we treat the dataset like a big matrix. `loc`, by contrast, uses the information in the indices to do its work. Since your dataset usually has meaningful indices, it's usually easier to do things using `loc` instead. For example, here's one operation that's much easier using `loc`:

```python
reviews.loc[:, ['taster_name', 'taster_twitter_handle', 'points']]
```

#### Choosing between `loc` and `iloc`

These two methods use different indexing schemas.

`iloc` uses the Python stdlib indexing scheme, where the first element of the range is included and the last one excluded. So `0:10` will select entries `0,...,9`. `loc`, meanwhile, indexes inclusively. So `0:10` will select entries `0,...,10`.

This is particularly confusing when the DataFrame index is a simple numerical list, e.g. `0,...,1000`. In this case `df.iloc[0:1000]` will return 1000 entries, while `df.loc[0:1000]` return 1001 of them! To get 1000 elements using `loc`, you will need to go one lower and ask for `df.loc[0:999]`.

Otherwise, the semantics of using `loc` are the same as those for `iloc`.

## Manipulating the index

We can use the `set_index` method converts one or more existing columns into the row index (labels) of a DataFrame.

```python
reviews.set_index('title')
```

This is useful if we come up with an index for the dataset which is better than the current one.

## Conditional selection

We can conditionally select columns based on some condition. Say in the above example we want to select all `country` where `country` is `Italy`.

```python
reviews.country == 'Italy'
```

This operation produced a Series of `True`/`False` booleans based on the `country` of each record. This result can then be used inside of `loc` to select the relevant data:

```python
reviews.loc[reviews.country == 'Italy']
```

To use multiple conditions, we can use `&` or `|` operator according to our use case.

```python
reviews.loc[(reviews.country == 'Italy') & (reviews.points >= 90)]

reviews.loc[(reviews.country == 'Italy') | (reviews.points >= 90)]
```

Pandas comes with a few built-in conditional selectors, two of which we will highlight here.

The first is `isin`. `isin` is lets you select data whose value "is in" a list of values. For example, here's how we can use it to select wines only from Italy or France:

```python
reviews.loc[reviews.country.isin(['Italy', 'France'])]
```

The second is `isnull` (and its companion `notnull`). These methods let you highlight values which are (or are not) empty (`NaN`). For example, to filter out wines lacking a price tag in the dataset, here's what we would do:

```python
reviews.loc[reviews.price.notnull()]
```

## Assigning Data

Going the other way, assigning data to a DataFrame is easy. You can assign either a constant value:

```python
reviews['critic'] = 'everyone'
```

Or with an iterable of values:

```python
reviews['index_backwards'] = range(len(reviews), 0, -1)
```

You can even re-assign values for columns that are already existing.

In case of iterables, the length of the iterable should be same of the column that you are assigning.

## Summary Functions and Maps

### Summary functions

Pandas provides many simple "summary functions" (not an official name) which restructure the data in some useful way. For example, consider the `describe()` method:

```python
reviews.points.describe()
```

The `describe()` method calculates summary statistics for a `DataFrame` or `Series`. It drops missing (`NaN`) values automatically before calculating metrics.

By default, `describe` only analyzes numeric columns. It returns a new DataFrame containing eight primary rows:
- **count**: Number of non-empty values.
- **mean**: Mathematical average of the values.
- **std**: Standard deviation (spread of data).
- **min**: Lowest value in the column.
- **25%**: The 25th percentile (first quartile).
- **50%**: The 50th percentile (median).
- **75%**: The 75th percentile (third quartile).
- **max**: Highest value in the column. 

If you run `describe` on non-numeric data, it returns completely different metrics:
- **count**: Number of non-empty values.
- **unique**: Total number of distinct values.
- **top**: The most frequently occurring value (mode).
- **freq**: Frequency of that most common value.

To see mean of a column we can use `mean` function:

```python
reviews.points.mean()
```

To see list of unique values we can use the `unique` function:

```python
reviews.taster_name.unique()
```

To see a list of unique values and how often they occur in the dataset, we can use the `value_counts()` method:

```python
reviews.taster_name.value_counts()
```

### Maps

A **map** is a term, borrowed from mathematics, for a function that takes one set of values and "maps" them to another set of values. There are cases when we need to transform data from one format to another.

There are two mapping methods that you will use often.

**`map()`** is the first one. For example, suppose that we want to get the mean of some data and re mean them around 0 (the value which is equal to mean will be 0, others will be `curr_value`- `mean_value`).

```python
review_points_mean = reviews.points.mean()
reviews.points.map(lambda p: p - review_points_mean)
```

The function you pass to `map()` should expect a single value from the Series (a point value, in the above example), and return a transformed version of that value. `map()` returns a new Series where all the values have been transformed by your function.

**`apply()`** is the equivalent method if we want to transform a whole DataFrame by calling a custom method on each row.

```python
def remean_points(row):
	row.points = row.points - review_points_mean
	return row
	
reviews.apply(remean_points, axis='columns')
```

If we had called `reviews.apply()` with `axis='index'`, then instead of passing a function to transform each row, we would need to give a function to transform each _column_.

Note that `map()` and `apply()` return new, transformed Series and DataFrames, respectively. They don't modify the original data they're called on.

Pandas provides many common mapping operations as build-ins. For example, here's a faster way of remeaning our points column:

```python
review_points_mean = reviews.points.mean()
reviews.points - review_points_mean
```

In this code we are performing an operation between a lot of values on the left-hand side (everything in the Series) and a single value on the right-hand side (the mean value). Pandas looks at this expression and figures out that we must mean to subtract that mean value from every value in the dataset.

Pandas will also understand what to do if we perform these operations between Series of equal length. For example, an easy way of combining country and region information in the dataset would be to do the following:

```python
reviews.country + " - " + reviews.region_1
```

It will produce a output something like this:

```
0     Italy - Etna 
1              NaN
```

These operators are faster than `map()` or `apply()` because they use speed ups built into pandas. All of the standard Python operators (`>`, `<`, `==`, and so on) work in this manner.

However, they are not as flexible as `map()` or `apply()`, which can do more advanced things, like applying conditional logic, which cannot be done with addition and subtraction alone.

## Grouping and Sorting

Maps allows us to transform data in a DataFrame or Series one value at a time for an entire column. However, often we want to group our data, and then do something specific to the group the data is in.

### Grouping 

Grouping means to split our data set and apply some mathematical or logical operation to each group, and combine the results back into a clean DataFrame.

Above we used a function called `value_counts()`. We can replicate what `value_counts()` does by the following:

```python
reviews.groupby('points').points.count()
```

`groupby` created a group of reviews according to `points` and than we are doing some operation on the `points` itself which is `count()`. 

We can use any of the summary functions we've used before with this data. For example, getting the minimum of a group:

```python
reviews.groupby('points').price.min()
```

_Note: Calling `df.groupby()` on its own does not calculate anything. It returns a lazy `DataFrameGroupBy` object. You must attach an operational method (like `.sum()` or `.mean()`) to see computed output._

You can think of each group we generate as being a slice of our DataFrame containing only data with values that match. This DataFrame is accessible to us directly using the `apply()` method, and we can then manipulate the data in any way we see fit.

For example, here's one way of selecting the first column of a group:

```python
reviews.groupby('winery').apply(lambda df: df.title.iloc[0])
```

For even more fine-grained control, we can also group by more than one column. For example:

```python
reviews.groupby(['country', 'province']).apply(lambda df: df.loc[df.points.idxmax()])
```

Another `groupby()` method worth mentioning is `agg()`, which lets you run a bunch of different functions on your DataFrame simultaneously. For example, we can generate a simple statistical summary of the dataset as follows:

```python
reviews.groupby(['country']).price.agg([len, min, max])
```

### Multi-indexes

Till now we have seen only a single-label index regarding Series or DataFrame. But depending on the operation we run, it sometimes will result in what is called a muti-index.

A multi-index differs from a regular index in that it has multiple levels. For example:

```python
countries_reviewed = reviews.groupby(['country', 'province']).description.agg([len])
countries_reviewed
```

We will get a output something like this:

|           |                  | len  |
| --------- | ---------------- | ---- |
| country   | province         |      |
| Argentina | Mendoza Province | 3264 |
|           | Other            | 536  |

```python
mi = countries_reviewed.index
type(mi)
```

Output

```
pandas.core.indexes.multi.MultiIndex
```

Multi-indices have several methods for dealing with their tiered structure which are absent for single-level indices. They also require two levels of labels to retrieve a value.

To convert multi-index method back into a single index we can use `reset_index()` method:

```python
countries_reviewed.reset_index()
```

Our index will go back to normal and look something like this:

|     | country   | province         | len  |
| --- | --------- | ---------------- | ---- |
| 0   | Argentina | Mendoza Province | 3264 |
| 1   | Argentina | Other            | 536  |
## Sorting

Looking at the above index, we can see that grouping returns data in index order, not in value order. 

To get data in the order we want we can use the `sort_values()` method.

```python
countries_reviewed = countries_reviewed.reset_index()
countries_reviewed.sort_values(by='len')
```

`sort_values` defaults to ascending sort. We can change that by passing a additional parameter of `ascending`.

```python
countries_reviewed.sort_values(by='len', ascending=False)
```

To sort by index values, we can use `sort_index()`.

```python
countries_reviewed.sort_index()
```

We can also sort by more than one column at a time:

```python
countries_reviews.sort_values(by=['country', 'len'])
```

## Data Types and Missing Values

### Dtypes

The data type for a column in a DataFrame or a Series is known as the **dtype**.

You can use the `dtype` property to grab the type of a specific column. 

```python
reviews.price.dtype
```

Output:

```
dtype('float64')
```

Alternatively, the `dtypes` property returns the `dtype` of _every_ column in the DataFrame.

```python
reviews.dtypes
```

You can even get data type of the index.

```python
reviews.index.dtype
```

Data types tells us something about how pandas is storing the data internally. `float64` means that it's using 64-bit floating point number; `int64` means a similarly sized integer instead.

One thing to keep in mind is that columns consisting of strings do not get their own type; they instead have `object` type. It is inherited from `numpy`. In newer version of pandas, `string` will come.

It's possible to convert a column of one type into another using `astype()`. For example, we can convert `int64` to `float64`:

```python
reviews.points.astype('float64')
```

Pandas also supports more exotic data types, such as categorical data and timeseries data.

## Missing Data

Entries missing values are given the value of `NaN`, short for "Not a Number". For technical reasons these `NaN` values are always of the `float64` dtype.

Pandas provides some methods specific to missing data. To select `NaN` entries you can use `pd.isnull()` (or its companion `pd.notnull()`). This is meant to be used thusly:

```python
reviews[pd.isnull(reviews.country)]
```

It will output only those rows where `reviews.country` is null or `NaN`.

Replacing missing values is a common operation. Pandas provides a really handy method for this problem: `fillna()`. `fillna()` provides a few different strategies for mitigating such data. For example, we can simply replace each `NaN` with an `"Unknown"`:

```python
reviews.region_2.filna("unknown")
```

It will replace all missing or null values in `region_2` with `unknown`.

Or we could fill each missing value with the first non-null value that appears sometime after the given record in the database. This is known as the backfill strategy.

Alternatively, we may have a non-null value that we would like to replace. We can do so using the `replace` method:

```python
reviews.taster_twitter_handle.replace("@kerinokeefe", "@kerino")
```

## Renaming and Combining

### Renaming

`rename()` lets us change index names and/or column names. 

```python
reviews.rename(columns={'points': 'score'})
```

It will rename `points` column to `score`.

You can even rename indexes as follows:

```python
reviews.rename(index={0: 'firstEntry', 1: 'secondEntry'})
```

What above does is rename index's first row to `firstEntry` and second row to `secondEntry`, rest of the column will remain unchanged.

Both the row index and the column index can have their own `name` attributes. You can use `rename_axis()` to change these names.

```python
reviews.rename_axis('wines', axis='rows').rename_axis('fields', axis='columns')
```

It will produce a output something like this:

| fields                  | rest of the column indexes... |
| ----------------------- | ----------------------------- |
| wines                   |                               |
| rest of the row indexes |                               |
### Combining

We can combine multiple Series or DataFrames using `concat`, `join` and `merge`.

The simplest combining method is `concat`. Given a list of elements, this function will combine at a particular axis. By default it combines vertically (`axis=0`).

```python
canadian_youtube = pd.read_csv("../input/youtube-new/CAvideos.csv")
british_youtube = pd.read_csv("../input/youtube-new/GBvideos.csv")

pd.concat([canadian_youtube, british_youtube])
```

`join` lets us combine DataFrames objects which have an index in common.

```python
left = canadian_youtube.set_index(['title', 'trending_date'])
right = british_youtube.set_index(['title', 'trending_date'])

left.join(right, lsuffix='_CAN', rsuffix='_UK')
```

If `right` doesn't have anything common, it will show `NaN` there. You can also specify column on which to join. `lsuffix` and `rsuffix` will concatenate values after column name.

The `lsuffix` and `rsuffix` parameters are necessary here because the data has the same column names in both British and Canadian datasets. If this wasn't true (because, say, we'd renamed them beforehand) we wouldn't need them.