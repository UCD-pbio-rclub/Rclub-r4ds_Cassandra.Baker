# Data Import
CB  
Thursday, June 1, 2017  


## Chapter 11 Data import

### 11.1 Introduction

Being able to import data into R is essential for using R to work with real data. For now, `readr` will be used to load files into R. 

### 11.2 Getting started

`readr` functions can be used to convert files into data frames. 

* `read_csv()`: read comma delimited files
* `read_csv2()`: read semicolon delimited files
* `read_tsv()`: read tab delimited files
* `read_delim()`: read files with any delimiter
* `read_fwf()`: read fixed width files (use `fwf_widths()` or `fwf_positions()` to specify fields by width or position)
* `read_table()`: read files with white space between columns
* `read_log()`: read Apache log files

For now, focus on `read_csv()` since csv files are very common. The first argument specifies the path to the file. 
```
heights <- read_csv("data/heights.csv")
```

When this is run, it will show the names and types of all columns within the file.

`read_csv()` can also be used to create a csv file within the code. 

```r
read_csv("a, b, c
         1, 2, 3
         4, 5, 6")
```

```
## # A tibble: 2 × 3
##       a     b     c
##   <int> <int> <int>
## 1     1     2     3
## 2     4     5     6
```

By default, `read_csv()` will read the first row as column names. If there is metadata at the beginning of a file, you can skip the first `n` rows with `skip = n`. Alternatively, you can drop lines starting with a character, like `#`, with `comment = "#"`. If the data does not have column names, `col_names = FALSE` will tell R label the columns sequentially instead of naming them with the first row. 

```r
read_csv("1, 2, 3 \n 4, 5, 6", col_names = FALSE)
```

```
## # A tibble: 2 × 3
##      X1    X2    X3
##   <int> <int> <int>
## 1     1     2     3
## 2     4     5     6
```

Or you can provide a separate vector of column names for R to use.

```r
read_csv("1, 2, 3 \n 4, 5, 6", col_names = c("x", "y", "z"))
```

```
## # A tibble: 2 × 3
##       x     y     z
##   <int> <int> <int>
## 1     1     2     3
## 2     4     5     6
```

Value(s) that represent missing values in the file can be specified with `na`.

```r
read_csv("a, b, c \n 1, 2, .", na = ".")
```

```
## # A tibble: 1 × 3
##       a     b     c
##   <int> <int> <chr>
## 1     1     2  <NA>
```

To read in more complicated files, it's important to understand how columns are parsed in `readr` and turned into vectors.

#### 11.2.1 Compared to base R

Using `read_csv()` has some benefits compared to the base version, `read.csv()`. 

* faster (could also try `data.table::fread()`)
* makes tibbles
* doesn't convert character vectors to factors
* doesn't use row names
* doesn't mess up column names
* more reproducible (isn't affected by operating system or environment)

#### 11.2.2 Exercises

1. `read_delim()` could be used to read files with fields separated by "|".

2. It appears that `read_csv()` and `read_tsv()` have all arguments in common. This includes `delim`, `quote`, `escape_backslash`, `escape_double`, `col_names`, `col_types`, `locale`, `na`, `quoted_na`, `trim_ws`, `n_max`, `guess_max`, `progress`. 

3. `file` and `col_positions` seem to be the most important arguments to `read_fwf()`. Neither of these arguments have a default value and are thus required for the function to run.

4. Read the below into a data frame 
```
"x, y \n 1,`a,b`"
```

The `quote` argument would need to be specified to read this into a data frame. Since you have to use `read_delim()` for this, you also need to specify `delim`. 

```r
read_delim("x, y \n 1,'a,b'", delim = ",", quote = "'")
```

```
## # A tibble: 1 × 2
##       x ` y `
##   <chr> <chr>
## 1     1   a,b
```

5. Identify code errors

```r
read_csv("a, b \n 1, 2, 3 \n 4, 5, 6")
```

```
## Warning: 2 parsing failures.
## row col  expected    actual         file
##   1  -- 2 columns 3 columns literal data
##   2  -- 2 columns 3 columns literal data
```

```
## # A tibble: 2 × 2
##       a     b
##   <int> <int>
## 1     1     2
## 2     4     5
```

There are three columns of observations and only two column names. This produces a data frame with only two columns and doesn't include the third column of observations.


```r
read_csv("a, b, c \n 1, 2 \n 1, 2, 3, 4")
```

```
## Warning: 2 parsing failures.
## row col  expected    actual         file
##   1  -- 3 columns 2 columns literal data
##   2  -- 3 columns 4 columns literal data
```

```
## # A tibble: 2 × 3
##       a     b     c
##   <int> <int> <int>
## 1     1     2    NA
## 2     1     2     3
```

Similar to above, the number of column names doesn't match the number of observations in each row. There are three column names, but only two observations in the first row and then four observations in the second row. The third observation in the row with only two specified observations is filled in with "NA". As above, the fourth observation in the row with four observations is not included in the data frame.


```r
read_csv("a, b \n\"1")
```

```
## Warning: 2 parsing failures.
## row col                     expected    actual         file
##   1  a  closing quote at end of file           literal data
##   1  -- 2 columns                    1 columns literal data
```

```
## # A tibble: 1 × 2
##       a     b
##   <int> <chr>
## 1     1  <NA>
```

It's unclear what the above is intented to achieve. If it's trying to have the 1 surrounded by quotes in the data frame, then the code is missing a space and an additional closing quote. 

```r
read_csv("a, b \n \"1\"")
```

```
## Warning: 1 parsing failure.
## row col  expected    actual         file
##   1  -- 2 columns 1 columns literal data
```

```
## # A tibble: 1 × 2
##       a     b
##   <chr> <chr>
## 1   "1"  <NA>
```


```r
read_csv("a, b \n 1, 2 \n a, b")
```

```
## # A tibble: 2 × 2
##       a     b
##   <chr> <chr>
## 1     1     2
## 2     a     b
```

Again, it's unclear what this is intended to look like. As far as I can tell, this doesn't have any obvious errors. Maybe they wanted "a,b" then "1,2" then "a,b" all in one column? `read_delim()` would need to be used to accomplish this.

```r
read_delim("'a,b'\n'1,2'\n'a,b'", delim = ",", quote = "'")
```

```
## # A tibble: 2 × 1
##   `a,b`
##   <chr>
## 1   1,2
## 2   a,b
```



```r
read_csv("a;b \n 1;3")
```

```
## # A tibble: 1 × 1
##   `a;b`
##   <chr>
## 1   1;3
```

It seems like `;` was used as the delimiter instead of `,`.

```r
read_csv("a, b \n 1, 3")
```

```
## # A tibble: 1 × 2
##       a     b
##   <int> <int>
## 1     1     3
```

### 11.3 Parsing a vector
