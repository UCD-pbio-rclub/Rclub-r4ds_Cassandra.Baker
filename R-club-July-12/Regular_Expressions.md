# Strings and Regular Expressions
CB  
Tuesday, July 12, 2017  


## Chapter 14 Strings

### 14.1 Introduction

Regular expressions describe patterns in strings.


```r
library(stringr)
```

### 14.2 String basics


```r
string1 <- "This is a string"
string2 <- 'If I want to include a "quote" inside a string, I use single quotes'

double_quote <- "\"" # or '"'
single_quote <- '\'' # or "'"
backslash <- "\\"
```

The string is different from its printed representation because of escape characters.

```r
x <- c("\"", "\\")
x
```

```
## [1] "\"" "\\"
```

```r
writeLines(x)
```

```
## "
## \
```

Special characters

* `"\n"`: new line
* `"\t"`: tab

`?"'"` will show all special characters. There are also strings to write non-English characters.

```r
x <- "\u00b5"
x
```

```
## [1] "µ"
```

#### 14.2.1 String length


```r
str_length(c("a", "R for data science", NA))
```

```
## [1]  1 18 NA
```

#### 14.2.2 Combining strings

Strings can be combined and the separator can be specified.

```r
str_c("x", "y")
```

```
## [1] "xy"
```

```r
str_c("x", "y", sep = ", ")
```

```
## [1] "x, y"
```

`str_replace_na()` can be used to print NA values as "NA". 

```r
x <- c("abc", NA)
str_c("|-", x, "-|")
```

```
## [1] "|-abc-|" NA
```

```r
str_c("|-", str_replace_na(x), "-|")
```

```
## [1] "|-abc-|" "|-NA-|"
```

Objects of length 0 are dropped.

```r
name <- "Cassandra"
time_of_day <- "evening"
birthday <- FALSE

str_c(
  "Good ", time_of_day, " ", name,
  if (birthday) " and HAPPY BIRTHDAY",
  "."
)
```

```
## [1] "Good evening Cassandra."
```

Vectors of strings can be collapsed into single strings.

```r
str_c(c("x", "y", "z"), collapse = ", ")
```

```
## [1] "x, y, z"
```

#### 14.2.3 Subsetting strings

Parts of a string can be extracted. 

```r
x <- c("Apple", "Banana", "Pear")
str_sub(x, 1, 3)
```

```
## [1] "App" "Ban" "Pea"
```

```r
str_sub(x, -3, -1)
```

```
## [1] "ple" "ana" "ear"
```

```r
str_sub("a", 1, 5)
```

```
## [1] "a"
```

`str_sub()` has an assignment form to modify strings.

```r
str_sub(x, 1, 1) <- str_to_lower(str_sub(x, 1, 1))
x
```

```
## [1] "apple"  "banana" "pear"
```

#### 14.2.4 Locales

Capitalization and sorting are affected by the locale. `str_sort()` and `str_order()` take additional locale arguments for robust analyses. 


```r
str_sort(x, locale = "en")
```

```
## [1] "apple"  "banana" "pear"
```

#### 14.2.5 Exercises

1. `paste()` uses space as a default separator while `paste0()` does not have a default separator. `paste()` and `paste0()` are equivalent to `str_c()`. `paste()` treats NA like a character whereas `str_c()` gives NA as the output if NA is in the input.

```r
paste("x", "y", NA)
```

```
## [1] "x y NA"
```

```r
paste0("x", "y", NA)
```

```
## [1] "xyNA"
```

```r
str_c("x", "y", NA)
```

```
## [1] NA
```

2. `sep` is inserted between the arguments of `str_c()` while `collapse` is inserted between elements of a vector within `str_c()`. 

```r
# separate arguments 
str_c("x", "y", sep = ":")
```

```
## [1] "x:y"
```

```r
str_c("x", "y", collapse = ":")
```

```
## [1] "xy"
```

```r
# vector
str_c(letters, sep = ":")
```

```
##  [1] "a" "b" "c" "d" "e" "f" "g" "h" "i" "j" "k" "l" "m" "n" "o" "p" "q"
## [18] "r" "s" "t" "u" "v" "w" "x" "y" "z"
```

```r
str_c(letters, collapse = ":")
```

```
## [1] "a:b:c:d:e:f:g:h:i:j:k:l:m:n:o:p:q:r:s:t:u:v:w:x:y:z"
```

```r
# both
str_c(1:26, letters, sep = " ", collapse = ", ")
```

```
## [1] "1 a, 2 b, 3 c, 4 d, 5 e, 6 f, 7 g, 8 h, 9 i, 10 j, 11 k, 12 l, 13 m, 14 n, 15 o, 16 p, 17 q, 18 r, 19 s, 20 t, 21 u, 22 v, 23 w, 24 x, 25 y, 26 z"
```

3. It looks like when an even string is divided by 2, R takes the first of the two middle characters. Maybe you could write an ifelse command to take both middle characters if the string has an even number of characters?

```r
x <- c("apple", "banana", "pear")
str_length(x)
```

```
## [1] 5 6 4
```

```r
str_sub(x, str_length(x)/2, str_length(x)/2)
```

```
## [1] "p" "n" "e"
```

4. `str_wrap()` wraps strings into a target width per line. This could be used to modify string visualization for printing onto paper or to fit better on a computer screen.

5. `str_trim()` will remove extra spaces from the start and/or end of strings. `str_pad()` is the opposite of `str_trim()` and will add whitespace to the sides of a string.

6. 

## regexone 

### Tutorial

space: ␣
tab: \t
new line: \n
carriage return: \r
any whitespace: \s
any non-whitespace: \S
any digits: \d
any non-digits: \D
letters and digits: \w
non-alphanumeric characters: \W
any number of: *
any number of, but at least 1: +
optional: ?
different possible characters: |
any character: .
boundary between word and non-word characters: \b
reference captured groups: \0 for entire match, \1 for first group

1: abc
1.5: 123
2: ...\.
3: [cmf]an
4: [^b]og
5: [A-Z][a-z][a-z]
6: waz{2,5}up
7: a+b*c+
8: [1-9]+ files? found\?
9: [1-9]\.\s+abc
10: [A-Za-z]+:[^un]successful$
11: ^(\w+)\.pdf$
12: ^([A-Z][a-z]{2}\s(\d{4}))$
13: ^(\d{3}\d?)x(\d{3}\d?)$
14: ^I love (cats|dogs)$
15: ^.+\.$

### Practice problems

