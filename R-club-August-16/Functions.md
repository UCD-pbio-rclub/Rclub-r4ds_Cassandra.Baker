# Functions
CB  
August 14, 2017  



## Chapter 19 Functions

### 19.1 Introduction

Functions are great for automating tasks in a way that makes code easier to understand, easier to use, and less prone to errors.

### 19.2 When should you write a function?

Writing a function would be useful when you use the same piece of code more than twice. This would also help prevent errors where variable names are copied multiple times and then changed in one place but not another.


```r
df <- tibble::tibble(
  a = rnorm(10),
  b = rnorm(10),
  c = rnorm(10),
  d = rnorm(10)
)

df$a <- (df$a - min(df$a, na.rm = TRUE)) /
  (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
```

* How many inputs are there?
In this piece of code, `df$a` is the only input.


```r
x <- df$a
(x - min(x, na.rm = TRUE)) / (max(x, na.rm = TRUE) - min(x,na.rm = TRUE))
```

```
##  [1] 0.26401934 0.69048181 0.60473750 0.08188242 0.50458472 0.00000000
##  [7] 0.59290288 0.27013964 1.00000000 0.53688675
```

* Use named variables to store intermediate calculations so that the code is easier to understand and duplication is easier to perform.


```r
rng <- range(x, na.rm = TRUE)
(x - rng[1]) / (rng[2] - rng[1])
```

```
##  [1] 0.26401934 0.69048181 0.60473750 0.08188242 0.50458472 0.00000000
##  [7] 0.59290288 0.27013964 1.00000000 0.53688675
```

* After simplification, the code can be turned into a function.


```r
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE)
  (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(c(0, 5, 10))
```

```
## [1] 0.0 0.5 1.0
```

**Making a function**

1. Create a name (better if it's informative)
2. List the arguments of the function, for example `function(x, y, z)`
3. Input developed code for the function body sandwiched by `{}`
4. Check the function with a few inputs


```r
rescale01(c(-10, 0, 10))
```

```
## [1] 0.0 0.5 1.0
```

```r
rescale01(c(1, 2, 3, NA, 5))
```

```
## [1] 0.00 0.25 0.50   NA 1.00
```

Use the function instead of copying and pasting code.

```r
df$a <- rescale01(df$a)
df$b <- rescale01(df$b)
df$c <- rescale01(df$c)
df$d <- rescale01(df$d)
```

This can later be further simplified with iteration.

When function requirements change, it's easier to modify the code because it only needs to be changed once.

```r
x <- c(1:10, Inf)
rescale01(x)
```

```
##  [1]   0   0   0   0   0   0   0   0   0   0 NaN
```

```r
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE, finite = TRUE)
  (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(x)
```

```
##  [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556 0.6666667
##  [8] 0.7777778 0.8888889 1.0000000       Inf
```

#### 19.2.1 Exercises

1. `TRUE` is not a parameter to `rescale01()` because it's a parameter of one of the components of `rescale01()` and not the actual function itself. If `x` contained a single missing value and `na.rm = FALSE`, all outputs would be "NA".

```r
rescale02 <- function(x) {
  rng <- range(x, na.rm = FALSE)
  (x - rng[1]) / (rng[2] - rng[1])
}

x <- c(1:10, NA)
rescale02(x)
```

```
##  [1] NA NA NA NA NA NA NA NA NA NA NA
```


2. Rewrite `rescale01()` so `-Inf` is mapped to 0 and `Inf` is mapped to 1. 

```r
rescale03 <- function(x) {
  rng <- range(x, na.rm = TRUE, finite = TRUE)
  y <- (x - rng[1]) / (rng[2] - rng[1])
  y[y == Inf] <- 1
  y[y == -Inf] <- 0
  y
}

x <- c(1:10, Inf, -Inf)
rescale03(x)
```

```
##  [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556 0.6666667
##  [8] 0.7777778 0.8888889 1.0000000 1.0000000 0.0000000
```

3. Create functions.

```r
# calculate fraction of NAs
fraction_NA <- function(x) {
  mean(is.na(x))
}

x <- c(1:10, NA)
fraction_NA(x)
```

```
## [1] 0.09090909
```

```r
# calculate weight of individual components within a vector
vector_weights <- function(x) {
  x / sum(x, na.rm = TRUE)
}

vector_weights(x)
```

```
##  [1] 0.01818182 0.03636364 0.05454545 0.07272727 0.09090909 0.10909091
##  [7] 0.12727273 0.14545455 0.16363636 0.18181818         NA
```

```r
# calculate coefficient of variation
coeff_var <- function(x) {
  sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
}

coeff_var(x)
```

```
## [1] 0.5504819
```

4. Create functions to compute variance and skew of a vector.

```r
# compute variance
variance <- function(x) {
  n <- length(x)
  m <- mean(x)
  (1/(n - 1)) * sum((x - m)^2)
}

x <- c(1:100)
variance(x)
```

```
## [1] 841.6667
```

```r
var(x)
```

```
## [1] 841.6667
```

```r
# compute skew
skew <- function(x) {
  n <- length(x)
  v <- var(x)
  m <- mean(x)
  third.moment <- (1/(n - 2)) * sum((x - m)^3)
  third.moment/(var(x)^(3/2))
}

skew(x)
```

```
## [1] 0
```

5. create a function that returns the number of NAs in two vectors of the same length.

```r
both_na <- function(x, y) {
  missing_x <- is.na(x)
  missing_y <- is.na(y)
  sum(missing_x & missing_y)
}

x <- c(1, NA, 4, 5, NA)
y <- c(NA, NA, 2, 7, 8)
both_na(x, y)
```

```
## [1] 1
```

6. `is_directory` appears to check if the input is a directory and `is_readable` appears to check if a file has read access. Even though these functions are short, they are useful for clarifying the code.

7. Code the entire "Little Bunny Foo Foo" song.
```
foo_foo %>% 
  hop(through = forest) %>% 
  scoop(up = field_mouse) %>% 
  bop(on = head)
```
This seems very confusing to me.

