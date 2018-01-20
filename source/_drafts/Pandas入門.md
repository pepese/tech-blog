---
title: Pandas入門
tags:
- Python
- Pandas
id: pandas-basics
---

# Series

一次元配列。

# DataFrame

二次元配列。

https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html

## 基本操作

列へのアクセス。  
一列取得した場合は **Series** になる。

```
df.Attribute
df["Attribute"]
df["Attribute", "Attribute"]
```

## 属性（Attributes）

|属性|説明|
|:---|:---|
|T|転置行列|
|at|Fast label-based scalar accessor|
|axes|行ラベルと列ラベルの情報|
|blocks|Internal property, property synonym for as_blocks()|
|columns|列ラベル|
|dtypes|Return the dtypes in this object.|
|empty|True if NDFrame is entirely empty [no items], meaning any of the axes are of length 0.|
|ftypes|Return the ftypes (indication of sparse/dense and dtype) in this object.|
|iat|Fast integer location scalar accessor.|
|iloc|Purely integer-location based indexing for selection by position.|
|index|行ラベル|
|is_copy||
|ix|A primarily label-location based indexer, with integer position fallback.（非推奨）|
|loc|Purely label-location based indexer for selection by label.|
|ndim|Number of axes / array dimensions 次元|
|shape|行と列のサイズ|
|size|行列の要素数（行サイズは `len(df.index)` ）|
|style|Property returning a Styler object containing methods for building a styled HTML representation for the DataFrame.|
|values|Numpy representation of NDFrame|

### `at` 、 `iat` 、 `loc` 、 `iloc`

- 単独の値（スカラー）にアクセスするのが `at` と `iat`
- 単独の値（スカラー）および複数の値（ベクトル）にアクセスするのが `loc` と `iloc`
- 行ラベル（行名）、列ラベル（列名）で位置を指定するのが `at` と `loc`
- 行番号、列番号で位置を指定するのが `iat` と `iloc`
- 処理速度は `at` と `iat` のほうが `loc` と `iloc` よりも高速

なお、 `DataFrame.get_value()` 、 `DataFrame.ix[]` もあるが、どちらも最新のバージョンでは非推奨（Deprecated）になっている。これから新しく書くコードには `at` , `iat` , `loc` , `iloc` を使うほうがいいだろう。

`at` は行ラベルと列ラベルで位置を指定する。

```
print(df.at[1, 'age'])
print(df.at[3, 'state'])
df.at[1, 'age'] = 60
```

`iat` は行番号と列番号で位置を指定する。行番号・列番号は `0` はじまり。

```
print(df.iat[1, 1])
print(df.iat[3, 2])
```

`loc` はスライス `x:y` やリスト `[x, y]` でデータの範囲・位置を指定する。参照される値は pandas.Series または pandas.DataFrame になる。  
列の指定を省略すると、行全体の参照になる。  
列全体を参照したい場合は、行の指定を `:` （全体のスライス）にする。

```
print(df.loc[1:3, 'age':'point'])
print(type(df.loc[1:3, 'age':'point']))

print(df.loc[[1, 3], ['age', 'point']])
print(type(df.loc[1:3, ['age', 'point']]))

df.loc[1:3, 'age'] = [20, 30, 40]
```

## メソッド（Methods）

|メソッド|説明|
|:---|:---|
|abs()|Return an object with absolute value taken–only applicable to objects that are all numeric.|
|add(other[, axis, level, fill_value])|Addition of dataframe and other, element-wise (binary operator add).|
|add_prefix(prefix)|Concatenate prefix string with panel items names.|
|add_suffix(suffix)|Concatenate suffix string with panel items names.|
|agg(func[, axis])|Aggregate using callable, string, dict, or list of string/callables|
|aggregate(func[, axis])|Aggregate using callable, string, dict, or list of string/callables|
|align(other[, join, axis, level, copy, ...])|Align two objects on their axes with the|
|all([axis, bool_only, skipna, level])|Return whether all elements are True over requested axis|
|any([axis, bool_only, skipna, level])|Return whether any element is True over requested axis|
|append(other[, ignore_index, verify_integrity])|Append rows of other to the end of this frame, returning a new object.|
|apply(func[, axis, broadcast, raw, reduce, args])|Applies function along input axis of DataFrame.|
|applymap(func)|Apply a function to a DataFrame that is intended to operate elementwise, i.e.|
|as_blocks([copy])|Convert the frame to a dict of dtype -> Constructor Types that each has a homogeneous dtype.|
|as_matrix([columns])|Convert the frame to its Numpy-array representation.|
|asfreq(freq[, method, how, normalize, ...])|Convert TimeSeries to specified frequency.|
|asof(where[, subset])|The last row without any NaN is taken (or the last row without|
|assign(**kwargs)|Assign new columns to a DataFrame, returning a new object (a copy) with all the original columns in addition to the new ones.|
|astype(dtype[, copy, errors])|Cast a pandas object to a specified dtype dtype.|
|at_time(time[, asof])|Select values at particular time of day (e.g.|
|between_time(start_time, end_time[, ...])|Select values between particular times of the day (e.g., 9:00-9:30 AM).|
|bfill([axis, inplace, limit, downcast])|Synonym for DataFrame.fillna(method='bfill')|
|bool()|Return the bool of a single element PandasObject.|
|boxplot([column, by, ax, fontsize, rot, ...])|Make a box plot from DataFrame column optionally grouped by some columns or|
|clip([lower, upper, axis, inplace])|Trim values at input threshold(s).|
|clip_lower(threshold[, axis, inplace])|Return copy of the input with values below given value(s) truncated.|
|clip_upper(threshold[, axis, inplace])|Return copy of input with values above given value(s) truncated.|
|combine(other, func[, fill_value, overwrite])|Add two DataFrame objects and do not propagate NaN values, so if for a|
|combine_first(other)|Combine two DataFrame objects and default to non-null values in frame calling the method.|
|compound([axis, skipna, level])|Return the compound percentage of the values for the requested axis|
|consolidate([inplace])|DEPRECATED: consolidate will be an internal implementation only.|
|convert_objects([convert_dates, ...])|Deprecated.|
|copy([deep])|Make a copy of this objects data.|
|corr([method, min_periods])|Compute pairwise correlation of columns, excluding NA/null values|
|corrwith(other[, axis, drop])|Compute pairwise correlation between rows or columns of two DataFrame objects.|
|count([axis, level, numeric_only])|Return Series with number of non-NA/null observations over requested axis.|
|cov([min_periods])|Compute pairwise covariance of columns, excluding NA/null values|
|cummax([axis, skipna])|Return cumulative max over requested axis.|
|cummin([axis, skipna])|Return cumulative minimum over requested axis.|
|cumprod([axis, skipna])|Return cumulative product over requested axis.|
|cumsum([axis, skipna])|Return cumulative sum over requested axis.|
|describe([percentiles, include, exclude])|Generates descriptive statistics that summarize the central tendency, dispersion and shape of a dataset’s distribution, excluding NaN values.|
|diff([periods, axis])|1st discrete difference of object|
|div(other[, axis, level, fill_value])|Floating division of dataframe and other, element-wise (binary operator truediv).|
|divide(other[, axis, level, fill_value])|Floating division of dataframe and other, element-wise (binary operator truediv).|
|dot(other)|Matrix multiplication with DataFrame or Series objects|
|drop([labels, axis, index, columns, level, ...])|Return new object with labels in requested axis removed.|
|drop_duplicates([subset, keep, inplace])|Return DataFrame with duplicate rows removed, optionally only|
|dropna([axis, how, thresh, subset, inplace])|Return object with labels on given axis omitted where alternately any|
|duplicated([subset, keep])|Return boolean Series denoting duplicate rows, optionally only|
|eq(other[, axis, level])|Wrapper for flexible comparison methods eq|
|equals(other)|Determines if two NDFrame objects contain the same elements.|
|eval(expr[, inplace])|Evaluate an expression in the context of the calling DataFrame instance.|
|ewm([com, span, halflife, alpha, ...])|Provides exponential weighted functions|
|expanding([min_periods, freq, center, axis])|Provides expanding transformations.|
|ffill([axis, inplace, limit, downcast])|Synonym for DataFrame.fillna(method='ffill')|
|fillna([value, method, axis, inplace, ...])|Fill NA/NaN values using the specified method|
|filter([items, like, regex, axis])|Subset rows or columns of dataframe according to labels in the specified index.|
|first(offset)|Convenience method for subsetting initial periods of time series data based on a date offset.|
|first_valid_index()|Return index for first non-NA/null value.|
|floordiv(other[, axis, level, fill_value])|Integer division of dataframe and other, element-wise (binary operator floordiv).|
|from_csv(path[, header, sep, index_col, ...])|Read CSV file (DEPRECATED, please use pandas.read_csv() instead).|
|from_dict(data[, orient, dtype])|Construct DataFrame from dict of array-like or dicts|
|from_items(items[, columns, orient])|Convert (key, value) pairs to DataFrame.|
|from_records(data[, index, exclude, ...])|Convert structured or record ndarray to DataFrame|
|ge(other[, axis, level])|Wrapper for flexible comparison methods ge|
|get(key[, default])|Get item from object for given key (DataFrame column, Panel slice, etc.).|
|get_dtype_counts()|Return the counts of dtypes in this object.|
|get_ftype_counts()|Return the counts of ftypes in this object.|
|get_value(index, col[, takeable])|Quickly retrieve single value at passed column and index|
|get_values()|same as values (but handles sparseness conversions)|
|groupby([by, axis, level, as_index, sort, ...])|Group series using mapper (dict or key function, apply given function to group, return result as series) or by a series of columns.|
|gt(other[, axis, level])|Wrapper for flexible comparison methods gt|
|head([n])|Return the first n rows.|
|hist(data[, column, by, grid, xlabelsize, ...])|Draw histogram of the DataFrame’s series using matplotlib / pylab.|
|idxmax([axis, skipna])|Return index of first occurrence of maximum over requested axis.|
|idxmin([axis, skipna])|Return index of first occurrence of minimum over requested axis.|
|infer_objects()|Attempt to infer better dtypes for object columns.|
|info([verbose, buf, max_cols, memory_usage, ...])|Concise summary of a DataFrame.|
|insert(loc, column, value[, allow_duplicates])|Insert column into DataFrame at specified location.|
|interpolate([method, axis, limit, inplace, ...])|Interpolate values according to different methods.|
|isin(values)|Return boolean DataFrame showing whether each element in the DataFrame is contained in values.|
|isna()|Return a boolean same-sized object indicating if the values are NA.|
|isnull()|Return a boolean same-sized object indicating if the values are NA.|
|items()|Iterator over (column name, Series) pairs.|
|iteritems()|Iterator over (column name, Series) pairs.|
|iterrows()|Iterate over DataFrame rows as (index, Series) pairs.|
|itertuples([index, name])|Iterate over DataFrame rows as namedtuples, with index value as first element of the tuple.|
|join(other[, on, how, lsuffix, rsuffix, sort])|Join columns with other DataFrame either on index or on a key column.|
|keys()|Get the ‘info axis’ (see Indexing for more)|
|kurt([axis, skipna, level, numeric_only])|Return unbiased kurtosis over requested axis using Fisher’s definition of kurtosis (kurtosis of normal == 0.0).|
|kurtosis([axis, skipna, level, numeric_only])|Return unbiased kurtosis over requested axis using Fisher’s definition of kurtosis (kurtosis of normal == 0.0).|
|last(offset)|Convenience method for subsetting final periods of time series data based on a date offset.|
|last_valid_index()|Return index for last non-NA/null value.|
|le(other[, axis, level])|Wrapper for flexible comparison methods le|
|lookup(row_labels, col_labels)|Label-based “fancy indexing” function for DataFrame.|
|lt(other[, axis, level])|Wrapper for flexible comparison methods lt|
|mad([axis, skipna, level])|Return the mean absolute deviation of the values for the requested axis|
|mask(cond[, other, inplace, axis, level, ...])|Return an object of same shape as self and whose corresponding entries are from self where cond is False and otherwise are from other.|
|max([axis, skipna, level, numeric_only])|This method returns the maximum of the values in the object.|
|mean([axis, skipna, level, numeric_only])|Return the mean of the values for the requested axis|
|median([axis, skipna, level, numeric_only])|Return the median of the values for the requested axis|
|melt([id_vars, value_vars, var_name, ...])|“Unpivots” a DataFrame from wide format to long format, optionally|
|memory_usage([index, deep])|Memory usage of DataFrame columns.|
|merge(right[, how, on, left_on, right_on, ...])|Merge DataFrame objects by performing a database-style join operation by columns or indexes.|
|min([axis, skipna, level, numeric_only])|This method returns the minimum of the values in the object.|
|mod(other[, axis, level, fill_value])|Modulo of dataframe and other, element-wise (binary operator mod).|
|mode([axis, numeric_only])|Gets the mode(s) of each element along the axis selected.|
|mul(other[, axis, level, fill_value])|Multiplication of dataframe and other, element-wise (binary operator mul).|
|multiply(other[, axis, level, fill_value])|Multiplication of dataframe and other, element-wise (binary operator mul).|
|ne(other[, axis, level])|Wrapper for flexible comparison methods ne|
|nlargest(n, columns[, keep])|Get the rows of a DataFrame sorted by the n largest values of columns.|
|notna()|Return a boolean same-sized object indicating if the values are not NA.|
|notnull()|Return a boolean same-sized object indicating if the values are not NA.|
|nsmallest(n, columns[, keep])|Get the rows of a DataFrame sorted by the n smallest values of columns.|
|nunique([axis, dropna])|Return Series with number of distinct observations over requested axis.|
|pct_change([periods, fill_method, limit, freq])|Percent change over given number of periods.|
|pipe(func, *args, **kwargs)|Apply func(self, *args, **kwargs)|
|pivot([index, columns, values])|Reshape data (produce a “pivot” table) based on column values.|
|pivot_table([values, index, columns, ...])|Create a spreadsheet-style pivot table as a DataFrame.|
|plot|alias of FramePlotMethods|
|pop(item)|Return item and drop from frame.|
|pow(other[, axis, level, fill_value])|Exponential power of dataframe and other, element-wise (binary operator pow).|
|prod([axis, skipna, level, numeric_only, ...])|Return the product of the values for the requested axis|
|product([axis, skipna, level, numeric_only, ...])|Return the product of the values for the requested axis|
|quantile([q, axis, numeric_only, interpolation])|Return values at the given quantile over requested axis, a la numpy.percentile.|
|query(expr[, inplace])|Query the columns of a frame with a boolean expression.|
|radd(other[, axis, level, fill_value])|Addition of dataframe and other, element-wise (binary operator radd).|
|rank([axis, method, numeric_only, ...])|Compute numerical data ranks (1 through n) along axis.|
|rdiv(other[, axis, level, fill_value])|Floating division of dataframe and other, element-wise (binary operator rtruediv).|
|reindex([labels, index, columns, axis, ...])|Conform DataFrame to new index with optional filling logic, placing NA/NaN in locations having no value in the previous index.|
|reindex_axis(labels[, axis, method, level, ...])|Conform input object to new index with optional filling logic, placing NA/NaN in locations having no value in the previous index.|
|reindex_like(other[, method, copy, limit, ...])|Return an object with matching indices to myself.|
|rename([mapper, index, columns, axis, copy, ...])|Alter axes labels.|
|rename_axis(mapper[, axis, copy, inplace])|Alter the name of the index or columns.|
|reorder_levels(order[, axis])|Rearrange index levels using input order.|
|replace([to_replace, value, inplace, limit, ...])|Replace values given in ‘to_replace’ with ‘value’.|
|resample(rule[, how, axis, fill_method, ...])|Convenience method for frequency conversion and resampling of time series.|
|reset_index([level, drop, inplace, ...])|For DataFrame with multi-level index, return new DataFrame with labeling information in the columns under the index names, defaulting to ‘level_0’, ‘level_1’, etc.|
|rfloordiv(other[, axis, level, fill_value])|Integer division of dataframe and other, element-wise (binary operator rfloordiv).|
|rmod(other[, axis, level, fill_value])|Modulo of dataframe and other, element-wise (binary operator rmod).|
|rmul(other[, axis, level, fill_value])|Multiplication of dataframe and other, element-wise (binary operator rmul).|
|rolling(window[, min_periods, freq, center, ...])|Provides rolling window calculations.|
|round([decimals])|Round a DataFrame to a variable number of decimal places.|
|rpow(other[, axis, level, fill_value])|Exponential power of dataframe and other, element-wise (binary operator rpow).|
|rsub(other[, axis, level, fill_value])|Subtraction of dataframe and other, element-wise (binary operator rsub).|
|rtruediv(other[, axis, level, fill_value])|Floating division of dataframe and other, element-wise (binary operator rtruediv).|
|sample([n, frac, replace, weights, ...])|Returns a random sample of items from an axis of object.|
|select(crit[, axis])|Return data corresponding to axis labels matching criteria|
|select_dtypes([include, exclude])|Return a subset of a DataFrame including/excluding columns based on their dtype.|
|sem([axis, skipna, level, ddof, numeric_only])|Return unbiased standard error of the mean over requested axis.|
|set_axis(labels[, axis, inplace])|Assign desired index to given axis|
|set_index(keys[, drop, append, inplace, ...])|Set the DataFrame index (row labels) using one or more existing columns.|
|set_value(index, col, value[, takeable])|Put single value at passed column and index|
|shift([periods, freq, axis])|Shift index by desired number of periods with an optional time freq|
|skew([axis, skipna, level, numeric_only])|Return unbiased skew over requested axis|
|slice_shift([periods, axis])|Equivalent to shift without copying data.|
|sort_index([axis, level, ascending, ...])|Sort object by labels (along an axis)|
|sort_values(by[, axis, ascending, inplace, ...])|Sort by the values along either axis|
|sortlevel([level, axis, ascending, inplace, ...])|DEPRECATED: use DataFrame.sort_index()|
|squeeze([axis])|Squeeze length 1 dimensions.|
|stack([level, dropna])|Pivot a level of the (possibly hierarchical) column labels, returning a DataFrame (or Series in the case of an object with a single level of column labels) having a hierarchical index with a new inner-most level of row labels.|
|std([axis, skipna, level, ddof, numeric_only])|Return sample standard deviation over requested axis.|
|sub(other[, axis, level, fill_value])|Subtraction of dataframe and other, element-wise (binary operator sub).|
|subtract(other[, axis, level, fill_value])|Subtraction of dataframe and other, element-wise (binary operator sub).|
|sum([axis, skipna, level, numeric_only, ...])|Return the sum of the values for the requested axis|
|swapaxes(axis1, axis2[, copy])|Interchange axes and swap values axes appropriately|
|swaplevel([i, j, axis])|Swap levels i and j in a MultiIndex on a particular axis|
|tail([n])|Return the last n rows.|
|take(indices[, axis, convert, is_copy])|Return the elements in the given positional indices along an axis.|
|to_clipboard([excel, sep])|Attempt to write text representation of object to the system clipboard This can be pasted into Excel, for example.|
|to_csv([path_or_buf, sep, na_rep, ...])|Write DataFrame to a comma-separated values (csv) file|
|to_dense()|Return dense representation of NDFrame (as opposed to sparse)|
|to_dict([orient, into])|Convert DataFrame to dictionary.|
|to_excel(excel_writer[, sheet_name, na_rep, ...])|Write DataFrame to an excel sheet|
|to_feather(fname)|write out the binary feather-format for DataFrames|
|to_gbq(destination_table, project_id[, ...])|Write a DataFrame to a Google BigQuery table.|
|to_hdf(path_or_buf, key, **kwargs)|Write the contained data to an HDF5 file using HDFStore.|
|to_html([buf, columns, col_space, header, ...])|Render a DataFrame as an HTML table.|
|to_json([path_or_buf, orient, date_format, ...])|Convert the object to a JSON string.|
|to_latex([buf, columns, col_space, header, ...])|Render an object to a tabular environment table.|
|to_msgpack([path_or_buf, encoding])|msgpack (serialize) object to input file path|
|to_panel()|Transform long (stacked) format (DataFrame) into wide (3D, Panel) format.|
|to_parquet(fname[, engine, compression])|Write a DataFrame to the binary parquet format.|
|to_period([freq, axis, copy])|Convert DataFrame from DatetimeIndex to PeriodIndex with desired|
|to_pickle(path[, compression, protocol])|Pickle (serialize) object to input file path.|
|to_records([index, convert_datetime64])|Convert DataFrame to record array.|
|to_sparse([fill_value, kind])|Convert to SparseDataFrame|
|to_sql(name, con[, flavor, schema, ...])|Write records stored in a DataFrame to a SQL database.|
|to_stata(fname[, convert_dates, ...])|A class for writing Stata binary dta files from array-like objects|
|to_string([buf, columns, col_space, header, ...])|Render a DataFrame to a console-friendly tabular output.|
|to_timestamp([freq, how, axis, copy])|Cast to DatetimeIndex of timestamps, at beginning of period|
|to_xarray()|Return an xarray object from the pandas object.|
|transform(func, *args, **kwargs)|Call function producing a like-indexed NDFrame|
|transpose(*args, **kwargs)|Transpose index and columns|
|truediv(other[, axis, level, fill_value])|Floating division of dataframe and other, element-wise (binary operator truediv).|
|truncate([before, after, axis, copy])|Truncates a sorted DataFrame/Series before and/or after some particular index value.|
|tshift([periods, freq, axis])|Shift the time index, using the index’s frequency if available.|
|tz_convert(tz[, axis, level, copy])|Convert tz-aware axis to target time zone.|
|tz_localize(tz[, axis, level, copy, ambiguous])|Localize tz-naive TimeSeries to target time zone.|
|unstack([level, fill_value])|Pivot a level of the (necessarily hierarchical) index labels, returning a DataFrame having a new level of column labels whose inner-most level consists of the pivoted index labels.|
|update(other[, join, overwrite, ...])|Modify DataFrame in place using non-NA values from passed DataFrame.|
|var([axis, skipna, level, ddof, numeric_only])|Return unbiased variance over requested axis.|
|where(cond[, other, inplace, axis, level, ...])|Return an object of same shape as self and whose corresponding entries are from self where cond is True and otherwise are from other.|
|xs(key[, axis, level, drop_level])|Returns a cross-section (row(s) or column(s)) from the Series/DataFrame.|

### `groupby`

集約。

### 検索

`[]` 内に `True` または `False` を返す式を指定する。

```
# "A" 列の値が 0 より大きい行を取得
df[df.A > 0]

# 値が 0 より大きい値のみを取得
df[df > 0]
```


# Kaggle Titanic

```
```
