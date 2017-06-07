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
# a
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
# a modified
read_csv("a, b, c \n 1, 2, 3 \n 4, 5, 6")
```

```
## # A tibble: 2 × 3
##       a     b     c
##   <int> <int> <int>
## 1     1     2     3
## 2     4     5     6
```


```r
# b
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
# b modified
read_csv("a, b, c, d \n 1, 2 \n 1, 2, 3, 4")
```

```
## Warning: 1 parsing failure.
## row col  expected    actual         file
##   1  -- 4 columns 2 columns literal data
```

```
## # A tibble: 2 × 4
##       a     b     c     d
##   <int> <int> <int> <int>
## 1     1     2    NA    NA
## 2     1     2     3     4
```


```r
# c
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

It's unclear what the above is intented to achieve. If it's trying to have the 1 surrounded by quotes in the data frame, then the code is missing a space and an additional closing quote. It's also possible that they were trying to set 1 as a character instead of a number.

```r
# c modified
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
read_csv("a, b \n 1", col_types = "cc")
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
## 1     1  <NA>
```


```r
# d
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
# d modified
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
# e
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
# e modified
read_csv("a, b \n 1, 3")
```

```
## # A tibble: 1 × 2
##       a     b
##   <int> <int>
## 1     1     3
```

### 11.3 Parsing a vector

`parse_*()` functions convert character vectors into specialized vectors with a specific format (such as a date). The first argument specifies the vector to parse and the `na` argument specifies missing values.

```r
parse_integer(c("1", "231", ".", "456"), na = ".")
```

```
## [1]   1 231  NA 456
```

A warning will pop up if parsing fails and the values that failed will appear as missing values in the output.

```r
(x <- parse_integer(c("123", "345", "abc", "123.45")))
```

```
## Warning: 2 parsing failures.
## row col               expected actual
##   3  -- an integer                abc
##   4  -- no trailing characters    .45
```

```
## [1] 123 345  NA  NA
## attr(,"problems")
## # A tibble: 2 × 4
##     row   col               expected actual
##   <int> <int>                  <chr>  <chr>
## 1     3    NA             an integer    abc
## 2     4    NA no trailing characters    .45
```

`problems()` will show all parsing failures.

```r
problems(x)
```

```
## # A tibble: 2 × 4
##     row   col               expected actual
##   <int> <int>                  <chr>  <chr>
## 1     3    NA             an integer    abc
## 2     4    NA no trailing characters    .45
```

There are a number of different `parse()` functions.

* `parse_logical()`: parse logical vectors
* `parse_integer()`: parse integer vectors
* `parse_double()`: strictly parse numeric vectors
* `parse_number()`: flexibly parse numeric vectors
* `parse_character()`: parse character vectors (important for character encodings)
* `parse_factor()`: parse factors (categorical variables)
* `parse_datetime()`, `parse_date()`, and `parse_time()`: parse dates and times (these are tricky because the format can vary)

#### 11.3.1 Numbers

Potential difficulties with parsing numbers:

* Different number formats (`.` or `,` as decimal marker)
* Context characters (10%)
* Grouping characters (1,000,000)

`locale` is used to specify different parsing options to adapt to various number formats. The default locale is based on US customs, like R.

```r
parse_double("1.23")
```

```
## [1] 1.23
```

```r
parse_double("1,23", locale = locale(decimal_mark = ","))
```

```
## [1] 1.23
```

`parse_number()` will deal with context characters because it ignores non-numeric characters. This is useful for extracting numeric values without the context characters (like %), but is also good for extracting numeric values within other text.

```r
parse_number("$100")
```

```
## [1] 100
```

```r
parse_number("20%")
```

```
## [1] 20
```

```r
parse_number("It cost $123.45")
```

```
## [1] 123.45
```

Grouping characters are handled with `parse_number()` and `locale()` because `parse_number()` on its own will ignore these characters.

```r
parse_number("$123,456,789")
```

```
## [1] 123456789
```

```r
parse_number("123.456.789", locale = locale(grouping_mark = "."))
```

```
## [1] 123456789
```

```r
parse_number("123'456'789", locale = locale(grouping_mark = "'"))
```

```
## [1] 123456789
```

#### 11.3.2 Strings

There are multiple ways to represent a character string, which makes `parse_character()` more difficult than it seems. 

```r
charToRaw("Hadley")
```

```
## [1] 48 61 64 6c 65 79
```

Characters are encoded from hexadecimal numbers using specific systems. ASCII is useful for encoding English characters, but doesn't work well with other languages. Instead, various standards have been used to encode non-english characters. For example, Latin1 was commonly used for Western European languages and Latin2 was commonly used for Eastern European languages. This could be difficult to work with because you need to know both the values and encoding system to interpret a string. Now, and in readr, UTF-8 is used as the universal standard for encoding characters. When working with older data or data not encoded by UTF-8, the strings will look weird because readr is trying to use UTF-8 by default.

```r
x1 <- "El Ni\xf1o was particularly bad this year"
x2 <- "\x82\xb1\x82\xf1\x82\xc9\x82\xbf\x82\xcd"

parse_character(x1, locale = locale(encoding = "Latin1"))
```

```
## [1] "El Niño was particularly bad this year"
```

```r
parse_character(x2, locale = locale(encoding = "Shift-JIS"))
```

```
## [1] "<U+3053><U+3093><U+306B><U+3061><U+306F>"
```

Sometimes the encoding system will be specified in data documentation, but not always. To help determine which encoding system was used, you can use `guess_encoding()`. You may also need to try a few different encoding systems before finding the correct one.

```r
guess_encoding(charToRaw(x1))
```

```
## # A tibble: 2 × 2
##     encoding confidence
##        <chr>      <dbl>
## 1 ISO-8859-1       0.46
## 2 ISO-8859-9       0.23
```

```r
guess_encoding(charToRaw(x2))
```

```
## # A tibble: 1 × 2
##   encoding confidence
##      <chr>      <dbl>
## 1   KOI8-R       0.42
```

`guess_encoding()` can be used with either a file path or a vector. 

#### 11.3.3 Factors

Factors are used in R for categorical variables. When using `parse_factor()`, a warning will appear if a value other than the specified categories is present. However, if there are a lot of unexpected values it's probably easier to leave the character vector and clean it up using other methods.

```r
fruit <- c("apple", "banana")
parse_factor(c("apple", "banana", "bananana"), levels = fruit)
```

```
## Warning: 1 parsing failure.
## row col           expected   actual
##   3  -- value in level set bananana
```

```
## [1] apple  banana <NA>  
## attr(,"problems")
## # A tibble: 1 × 4
##     row   col           expected   actual
##   <int> <int>              <chr>    <chr>
## 1     3    NA value in level set bananana
## Levels: apple banana
```

#### 11.3.4 Dates, date-times, and times

`parse_datetime()` expects a date-time in the ISO8601 format, which is organized as year, month, day, hour, minute, second. 

```r
parse_datetime("2010-10-01T2010")
```

```
## [1] "2010-10-01 20:10:00 UTC"
```

```r
parse_datetime("20101010")
```

```
## [1] "2010-10-10 UTC"
```

`parse_date()` expects a date formatted as year, month, day.

```r
parse_date("2010-10-01")
```

```
## [1] "2010-10-01"
```

```r
parse_date("2010/10/01")
```

```
## [1] "2010-10-01"
```

`parse_time()` expects a time formatted as hour, minutes, seconds and an optional am/pm specifier. The hms package provides a class for time data. 

```r
library(hms)
parse_time("01:10 am")
```

```
## 01:10:00
```

```r
parse_time("20:10:01")
```

```
## 20:10:01
```

You can also create a date-time format with certain components.

**Year** 

* `%Y`: 4 digits
* `%y`: 2 digits (00-69 -> 2000-2069, 70-99 -> 1970-1999)

**Month**

* `%m`: 2 digits
* `%b`: abbreviated month name (Jan)
* `%B`: full month name (January)

**Day**

* `%d`: 2 digits
* `%e`: optional leading space

**Time**

* `%H`: 0-23 hour time
* `%I`: 0-12 hour time that requires an am/pm indication with `%p`
* `%p`: AM/PM indication
* `%M`: minutes
* `%S`: integer seconds
* `%OS`: real seconds
* `%Z`: time zone as a name (America/Chicago), but be careful with abbreviations
* `%z`: time zone offset from UTC (+0800)

**Non-digits**

* `%.`: skips one non-digit character
* `%*`: skips any non-digit characters

Test potential formats with example vectors and the parsing functions.

```r
parse_date("01/02/15", "%m/%d/%y")
```

```
## [1] "2015-01-02"
```

```r
parse_date("01/02/15", "%d/%m/%y") 
```

```
## [1] "2015-02-01"
```

```r
parse_date("01/02/15", "%y/%m/%d")
```

```
## [1] "2001-02-15"
```

`%b` and `%B` work with English month names by default, but the language parameters can be changed to recognize month names in other languages. `date_names_langs()` will show the list of built-in languages and you can create your own language with `date_names()`. 

```r
parse_date("1 janvier 2015", "%d %B %Y", locale = locale("fr"))
```

```
## [1] "2015-01-01"
```

#### 11.3.5 Exercises

1. The most important arguments to `locale()` are whichever ones need to be changed from default to work with your data. Otherwise, I think they're equally important. The arguments to `locale()` can be used to specify language, date and time format, decimal and grouping characters, time zone, encoding system, and conversion to ASCII. 

2. `locale` will produce an error when `decimal_mark` and `grouping_mark` are set to the same character. 
```
locale(decimal_mark = ".", grouping_mark = ".")
# Error: `decimal_mark` and `grouping_mark` must be different
```

The default value of `grouping_mark` changes to `.` when you set `decimal_mark` to `,`. 

```r
locale(decimal_mark = ",")
```

```
## <locale>
## Numbers:  123.456,78
## Formats:  %AD / %AT
## Timezone: UTC
## Encoding: UTF-8
## <date_names>
## Days:   Sunday (Sun), Monday (Mon), Tuesday (Tue), Wednesday (Wed),
##         Thursday (Thu), Friday (Fri), Saturday (Sat)
## Months: January (Jan), February (Feb), March (Mar), April (Apr), May
##         (May), June (Jun), July (Jul), August (Aug), September
##         (Sep), October (Oct), November (Nov), December (Dec)
## AM/PM:  AM/PM
```

The default value of `decimal_mark` changes to `,` when you set `grouping_mark` to `.`. 

```r
locale(grouping_mark = ".")
```

```
## <locale>
## Numbers:  123.456,78
## Formats:  %AD / %AT
## Timezone: UTC
## Encoding: UTF-8
## <date_names>
## Days:   Sunday (Sun), Monday (Mon), Tuesday (Tue), Wednesday (Wed),
##         Thursday (Thu), Friday (Fri), Saturday (Sat)
## Months: January (Jan), February (Feb), March (Mar), April (Apr), May
##         (May), June (Jun), July (Jul), August (Aug), September
##         (Sep), October (Oct), November (Nov), December (Dec)
## AM/PM:  AM/PM
```

3. The `date_format` and `time_format` options specify the format of dates and times respectively. By default, `date_format = "%AD"` and `time_format = "%AT"`. These formatting options may be useful for adjusting the date and time to match conventions in different countries, or different dataframes. For example, if I'm parsing "06-07-2017" and "0210pm" the default date and time formats would need to be adjusted for proper parsing.

```r
# default
parse_date("06-07-2017")
```

```
## Warning: 1 parsing failure.
## row col   expected     actual
##   1  -- date like  06-07-2017
```

```
## [1] NA
```

```r
# modified
parse_date("06-07-2017", locale = locale(date_format = "%m-%d-%Y"))
```

```
## [1] "2017-06-07"
```

```r
# default
parse_time("0210pm")
```

```
## Warning: 1 parsing failure.
## row col   expected actual
##   1  -- time like  0210pm
```

```
## NA
```

```r
# modified
parse_time("0210pm", locale = locale(time_format = "%I%M%p"))
```

```
## 14:10:00
```

4. If I were living in France, I may need a new locale similar to below.

```r
(new_locale <- locale(date_names = "fr", date_format = "%d/%m/%Y", time_format = "%H%M", decimal_mark = ","))
```

```
## <locale>
## Numbers:  123.456,78
## Formats:  %d/%m/%Y / %H%M
## Timezone: UTC
## Encoding: UTF-8
## <date_names>
## Days:   dimanche (dim.), lundi (lun.), mardi (mar.), mercredi (mer.),
##         jeudi (jeu.), vendredi (ven.), samedi (sam.)
## Months: janvier (janv.), février (févr.), mars (mars), avril (avr.), mai
##         (mai), juin (juin), juillet (juil.), août (août),
##         septembre (sept.), octobre (oct.), novembre (nov.),
##         décembre (déc.)
## AM/PM:  AM/PM
```

5. `read_csv()` uses `,` as the delimiter and `read_csv2()` uses `;` as the delimiter.

6. UTF-8 is universally commonly used, but other encoding systems vary by region. ISO 8859 variants and Windows variants are common in a number of regions. In Western Europe, ISO 8859-1 and Windows-1250 are both commonly used, while ISO 8859-2 and Windows-1252 are common in Eastern Europe.  Greek languages use ISO 8859-7; Turkish languages use ISO 8859-9 and Windows-1254; Hebrew uses ISO 8859-8, IBM424, and Windows-1255; and Russian uses Windows-1251. Mac OS Roman is also common in Europe. 

ISO 8859-6 and Windows-1256 are common for Arabic, Windows-1258 is common for Vietnamese, and ISO 8859-11 is common for Thai. JIS X 0208, including Shift JIS, EUC-JP, and ISO 2022-JP, is common for Japanese. Guobiao, including GB 2312, GBK, and GB 18030, is common for Chinese. Big5 and HKSCS are used in Taiwan. KS X 1001, EUC-KR, and ISO 2022-KR are commonly used encoding systems in Korea.

7. Parse the following:

```r
d1 <- "January 1, 2010"
parse_date(d1, "%B %d, %Y")
```

```
## [1] "2010-01-01"
```

```r
d2 <- "2015-Mar-07"
parse_date(d2, "%Y-%b-%d")
```

```
## [1] "2015-03-07"
```

```r
d3 <- "06-Jun-2017"
parse_date(d3, "%d-%b-%Y")
```

```
## [1] "2017-06-06"
```

```r
d4 <- c("August 19 (2015)", "July 1 (2015)")
parse_date(d4, "%B %d (%Y)")
```

```
## [1] "2015-08-19" "2015-07-01"
```

```r
d5 <- "12/30/14"
parse_date(d5, "%m/%d/%y")
```

```
## [1] "2014-12-30"
```

```r
t1 <- "1705"
parse_time(t1, "%H%M")
```

```
## 17:05:00
```

```r
t2 <- "11:15:10.12 PM"
parse_datetime(t2, "%m:%d:%y.%I %p")
```

```
## [1] "2010-11-15 12:00:00 UTC"
```

