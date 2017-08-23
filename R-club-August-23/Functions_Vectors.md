# Functions cont. and Vectors
CB  
August 23, 2017  



## Chapter 19 Functions

### 19.5 Function arguments 

Functions include data arguments and detail arguments. Data arguments come first and specify what to use the function on. Detail arguments come later and specify how the function is run. Default values should be specified as the most common value or the safest value (you probably don't want to set `na.rm = TRUE` as the default). 


```r
mean_ci <- function(x, conf = 0.95) {
  se <- sd(x) / sqrt(length(x))
  alpha <- 1 - conf
  mean(x) + se * qnorm(c(alpha / 2, 1 - alpha / 2))
}

x <- runif(100)
mean_ci(x)
```

```
## [1] 0.4139916 0.5287267
```

```r
mean_ci(x, conf = 0.99)
```

```
## [1] 0.3959654 0.5467529
```

#### 19.5.1 Choosing names

Select descriptive or already established names for your function arguments. 

Common names

* `x`, `y`, `z` as vectors
* `w` as a weight vector
* `df` as a data frame
* `i` and `j` as numeric indices (rows and columns)
* `n` as a length or a number of rows
* `p` as a number of columns

#### 19.5.2 Checking values

Set up conditions that will produce errors if function requirements are not met. 


```r
wt_mean <- function(x, w) {
  sum(x * w) / sum(w)
}
wt_var <- function(x, w) {
  mu <- wt_mean(x, w)
  sum(w * (x - mu) ^ 2) / sum(w)
}
wt_sd <- function(x, w) {
  sqrt(wt_var(x, w))
}

wt_mean(1:6, 1:3)
```

```
## [1] 7.666667
```

```r
wt_mean <- function(x, w) {
  if (length(x) != length(w)) {
    stop("`x` and `w` must be the same length", call. = FALSE)
  }
  sum(w * x) / sum(w)
}
```

`stopifnot()` can also be used to check for certain conditions.

```r
wt_mean <- function(x, w) {
  stopifnot(length(x) == length(w))
  sum(w * x) / sum(w)
}

wt_mean(1:6, 6:1)
```

```
## [1] 2.666667
```

```r
wt_mean(1:6, 1:3)
```

```
## Error: length(x) == length(w) is not TRUE
```

#### 19.5.3 Dot-dot-dot (...)

`...` is a special argument that captures all unmatched arguments and can send those arguments on to another function. However, typos will not produce errors in `...`. 


```r
commas <- function(...) stringr::str_c(..., collapse = ", ")
commas(letters[1:10])
```

```
## [1] "a, b, c, d, e, f, g, h, i, j"
```

```r
rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}
rule("Important output")
```

```
## Important output ------------------------------------------------------
```

#### 19.5.4 Lazy evaluation

R uses lazy evaluation, which means arguments are not evaluated until they are needed.

#### 19.5.5 Exercises

**1. What does `commas(letters, collapse = "-")` do? Why?**
It produces an error because `collapse` is already specified in the `str_c()` call of the function instead of being included in the `...` argument. So this code forwards `collapse = "-"` to the `...` argument in addition to the `collapse = ", "` in the `str_c()` function, which ends up specifying `collapse` as two conflicting directives.  

```r
commas(letters, collapse = "-")
```

```
## Error in stringr::str_c(..., collapse = ", "): formal argument "collapse" matched by multiple actual arguments
```

**2. It'd be nice if you could supply multiple characters to the `pad` argument, e.g. `rule("Title", pad = "-+")`. Why doesnt this currently work? How could you fix it?**


```r
rule("Title", pad = "-+")
```

```
## Title -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Currently, this doesn't work because it adds `pad` for the `width`, which ends up being longer than the actual width if there are multiple characters in `pad`. 


```r
rule <- function(..., pad = "-") {
  pad0 <- pad
  title <- paste0(...)
  width <- ((getOption("width") - nchar(title)) - 5) / nchar(pad0)
  cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}

rule("title", pad = "-+")
```

```
## title -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

```r
rule("Important output", pad = "-_+=")
```

```
## Important output -_+=-_+=-_+=-_+=-_+=-_+=-_+=-_+=-_+=-_+=-_+=-_+=-_+=
```

**3. What does the `trim` argument to `mean()` do? When might you use it?**

The `trim` argument to `mean` trims off a symmetrical fraction of observations from each end of the observations before computing the mean. This might be used to calculate the mean without outliers. 


```r
mean(c(1, 40:60, 10,000))
```

```
## [1] 44.20833
```

```r
mean(c(1, 40:60, 10,000), trim = 0.5)
```

```
## [1] 48.5
```

```r
mean(c(40:60, 10,000), trim = 0.5)
```

```
## [1] 49
```

**4. The default value for the `method` argument to `cor()` is `c("pearson", "kendall", "spearman")`. What does that mean? what value is used by default?**

That means that the "pearson", "kendall", or "spearman" methods can be used for the calculation. The "pearson" method is used by default. 

### 19.6 Return values
Two considerations when returning a value from a function.

1. Is the function easier to read if it's returned early?
2. Can the function be pipeable?

#### 19.6.1 Explicit return statements

Use `return()` to return simple solutions early and require less to understand the code.

```
complicated <- function(x, y, z) {
  if (length(x) == 0 || length(y) == 0) {
    return(0)
  }
  # rest of complicated code
}
```

#### 19.6.2 Writing pipeable functions

When working with pipeable functions, be sure to track the input for each step. With transformation functions, the input is modified and that modified input is fed into the next function. With side-effect functions, the input is not modified and is then fed directly into the next function.


```r
show_missings <- function(df) {
  n <- sum(is.na(df))
  cat("Missing values: ", n, "\n", sep = "")
  invisible(df)
}

mtcars %>% 
  show_missings() %>% 
  mutate(mpg = ifelse(mpg < 20, NA, mpg)) %>% 
  show_missings()
```

```
## Missing values: 0
## Missing values: 18
```

### 19.7 Environment

R will use the environment to search for functions within variables that aren't actually defined within the function. 

Also, with R you can actually overwrite existing R symbols and functions, which is useful but also dangerous.

## Chapter 20 Vectors

### 20.1 Introduction


```r
library(tidyverse)
```

### 20.2 Vector basics

Atomic vectors and lists are the main vector types. Vector type and vector length are the two main vector properties.

```r
typeof(letters)
```

```
## [1] "character"
```

```r
x <- list("a", "b", 1:10)
length(x)
```

```
## [1] 3
```

Augmented vectors include metadata within attributes that build on additional behavior.

### 20.3 Important types of atomic vector

#### 20.3.1 Logical

There are three possible values in logical vectors: `FALSE`, `TRUE`, and `NA`. 

```r
1:10 %% 3 == 0
```

```
##  [1] FALSE FALSE  TRUE FALSE FALSE  TRUE FALSE FALSE  TRUE FALSE
```

#### 20.3.2 Numeric

Numeric vectors include integer and double vectors. Numers are doubles by default, but can be designated as integers by adding `L` after them.

```r
typeof(1)
```

```
## [1] "double"
```

```r
typeof(1L)
```

```
## [1] "integer"
```

Doubles are only approximations, and may not provide the expected results with direct comparisons (`==`). `dplyr::near()` can be used with doubles instead.

The only special value for integers is `NA`. For doubles, `NA`, `NaN`, `Inf`, and `-Inf` are all special values. Check for these with `is.finite()`, `is.infinite()`, `is.na()`, and `is.nan()`. 

#### 20.3.3 Character

Each element of a character vector is a string and R only stores each unique vector once, which reduces the memory required for duplicated strings.


```r
x <- "This is a reasonably long string."
pryr::object_size(x)
```

```
## 136 B
```

```r
y <- rep(x, 1000)
pryr::object_size(y)
```

```
## 8.13 kB
```

#### 20.3.4 Missing values

Each type of atomic vector has its own missing value, but normally `NA` will be converted to the correct type.

#### 20.3.5 Exercises

**1. Describe the difference between `is.finite(x)` and `!is.infinite(x)`.**

`is.finite(x)` will check for finite values, while `!is.infinite(x)` will just check for values that are not `Inf`. 

```r
is.finite(NA)
```

```
## [1] FALSE
```

```r
!is.infinite(NA)
```

```
## [1] TRUE
```

**2. Read the source code for `dplyr::near()` (Hint: to see the source code, drop the `()`). How does it work?**

`dplyr::near()` appears to work by setting a tolerance value and then checking if the difference between two number is less than that tolerance value or not.

**3. A logical vector can take 3 possible values. How many possible values can an integer vector take? How many possible values can a double take? Use google to do some research.**

**4. Brainstorm at least four functions that allow you to convert a double to an integer. How do they differ? Be precise.**

**5. What functions from the readr package allow you to turn a string into logical, integer, and double vector?**
