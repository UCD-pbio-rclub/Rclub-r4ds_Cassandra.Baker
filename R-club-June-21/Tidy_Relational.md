# Tidy Data and Relational Data
CB  
Tuesday, June 20, 2017  


## Chapter 12 Tidy Data

### 12.5 Missing Values

Data can be missing explicitly or implicitly. Even changing the format of a data frame may make implicit missing data explicit.


```r
stocks <- tibble(
  year   = c(2015, 2015, 2015, 2015, 2016, 2016, 2016),
  qtr    = c(   1,    2,    3,    4,    2,    3,    4),
  return = c(1.88, 0.59, 0.35,   NA, 0.92, 0.17, 2.66)
)

stocks %>% 
  spread(year, return)
```

```
## # A tibble: 4 × 3
##     qtr `2015` `2016`
## * <dbl>  <dbl>  <dbl>
## 1     1   1.88     NA
## 2     2   0.59   0.92
## 3     3   0.35   0.17
## 4     4     NA   2.66
```

`na.rm = TRUE` will remove explicit missing values.

```r
stocks %>% 
  spread(year, return) %>% 
  gather(year, return, `2015`:`2016`, na.rm = TRUE)
```

```
## # A tibble: 6 × 3
##     qtr  year return
## * <dbl> <chr>  <dbl>
## 1     1  2015   1.88
## 2     2  2015   0.59
## 3     3  2015   0.35
## 4     2  2016   0.92
## 5     3  2016   0.17
## 6     4  2016   2.66
```

`complete()` will insert explicit missing values by column combinations.

```r
stocks %>% 
  complete(year, qtr)
```

```
## # A tibble: 8 × 3
##    year   qtr return
##   <dbl> <dbl>  <dbl>
## 1  2015     1   1.88
## 2  2015     2   0.59
## 3  2015     3   0.35
## 4  2015     4     NA
## 5  2016     1     NA
## 6  2016     2   0.92
## 7  2016     3   0.17
## 8  2016     4   2.66
```

A value can be carried down to following missing values with `fill()`. 

```r
treatment <- tribble(
  ~ person,           ~ treatment, ~ response,
  "Derrick Whitmore", 1,           7,
  NA,                 2,          10, 
  NA,                 3,           9,
  "Katherine Burke",  1,           4
)

treatment %>% 
  fill(person)
```

```
## # A tibble: 4 × 3
##             person treatment response
##              <chr>     <dbl>    <dbl>
## 1 Derrick Whitmore         1        7
## 2 Derrick Whitmore         2       10
## 3 Derrick Whitmore         3        9
## 4  Katherine Burke         1        4
```

#### 12.5.1 Exercises

1. The `fill` arguments in both `spread()` and `complete()` replace missing values with another specified value. However, `fill` in `spread()` will fill in one specific value for all missing values, while `fill` in `complete()` will fill in missing values differently for each column with values specified by column. 

2. `direction` within `fill()` will specify if a value should be applied to the missing values either above or below it.

### 12.6 Case Study

Tidying the who data one step at a time:

```r
(who1 <- who %>% 
  gather(new_sp_m014:newrel_f65, key = "key", value = "cases", na.rm = TRUE))
```

```
## # A tibble: 76,046 × 6
##        country  iso2  iso3  year         key cases
## *        <chr> <chr> <chr> <int>       <chr> <int>
## 1  Afghanistan    AF   AFG  1997 new_sp_m014     0
## 2  Afghanistan    AF   AFG  1998 new_sp_m014    30
## 3  Afghanistan    AF   AFG  1999 new_sp_m014     8
## 4  Afghanistan    AF   AFG  2000 new_sp_m014    52
## 5  Afghanistan    AF   AFG  2001 new_sp_m014   129
## 6  Afghanistan    AF   AFG  2002 new_sp_m014    90
## 7  Afghanistan    AF   AFG  2003 new_sp_m014   127
## 8  Afghanistan    AF   AFG  2004 new_sp_m014   139
## 9  Afghanistan    AF   AFG  2005 new_sp_m014   151
## 10 Afghanistan    AF   AFG  2006 new_sp_m014   193
## # ... with 76,036 more rows
```

```r
who1 %>% 
  count(key)
```

```
## # A tibble: 56 × 2
##             key     n
##           <chr> <int>
## 1   new_ep_f014  1032
## 2  new_ep_f1524  1021
## 3  new_ep_f2534  1021
## 4  new_ep_f3544  1021
## 5  new_ep_f4554  1017
## 6  new_ep_f5564  1017
## 7    new_ep_f65  1014
## 8   new_ep_m014  1038
## 9  new_ep_m1524  1026
## 10 new_ep_m2534  1020
## # ... with 46 more rows
```

```r
(who2 <- who1 %>% 
  mutate(key = stringr::str_replace(key, "newrel", "new_rel")))
```

```
## # A tibble: 76,046 × 6
##        country  iso2  iso3  year         key cases
##          <chr> <chr> <chr> <int>       <chr> <int>
## 1  Afghanistan    AF   AFG  1997 new_sp_m014     0
## 2  Afghanistan    AF   AFG  1998 new_sp_m014    30
## 3  Afghanistan    AF   AFG  1999 new_sp_m014     8
## 4  Afghanistan    AF   AFG  2000 new_sp_m014    52
## 5  Afghanistan    AF   AFG  2001 new_sp_m014   129
## 6  Afghanistan    AF   AFG  2002 new_sp_m014    90
## 7  Afghanistan    AF   AFG  2003 new_sp_m014   127
## 8  Afghanistan    AF   AFG  2004 new_sp_m014   139
## 9  Afghanistan    AF   AFG  2005 new_sp_m014   151
## 10 Afghanistan    AF   AFG  2006 new_sp_m014   193
## # ... with 76,036 more rows
```

```r
(who3 <- who2 %>% 
  separate(key, c("new", "type", "sexage"), sep = "_"))
```

```
## # A tibble: 76,046 × 8
##        country  iso2  iso3  year   new  type sexage cases
## *        <chr> <chr> <chr> <int> <chr> <chr>  <chr> <int>
## 1  Afghanistan    AF   AFG  1997   new    sp   m014     0
## 2  Afghanistan    AF   AFG  1998   new    sp   m014    30
## 3  Afghanistan    AF   AFG  1999   new    sp   m014     8
## 4  Afghanistan    AF   AFG  2000   new    sp   m014    52
## 5  Afghanistan    AF   AFG  2001   new    sp   m014   129
## 6  Afghanistan    AF   AFG  2002   new    sp   m014    90
## 7  Afghanistan    AF   AFG  2003   new    sp   m014   127
## 8  Afghanistan    AF   AFG  2004   new    sp   m014   139
## 9  Afghanistan    AF   AFG  2005   new    sp   m014   151
## 10 Afghanistan    AF   AFG  2006   new    sp   m014   193
## # ... with 76,036 more rows
```

```r
who3 %>% count(new)
```

```
## # A tibble: 1 × 2
##     new     n
##   <chr> <int>
## 1   new 76046
```

```r
who4 <- who3 %>% 
  select(-new, -iso2, -iso3)

(who5 <- who4 %>% 
  separate(sexage, c("sex", "age"), sep = 1))
```

```
## # A tibble: 76,046 × 6
##        country  year  type   sex   age cases
## *        <chr> <int> <chr> <chr> <chr> <int>
## 1  Afghanistan  1997    sp     m   014     0
## 2  Afghanistan  1998    sp     m   014    30
## 3  Afghanistan  1999    sp     m   014     8
## 4  Afghanistan  2000    sp     m   014    52
## 5  Afghanistan  2001    sp     m   014   129
## 6  Afghanistan  2002    sp     m   014    90
## 7  Afghanistan  2003    sp     m   014   127
## 8  Afghanistan  2004    sp     m   014   139
## 9  Afghanistan  2005    sp     m   014   151
## 10 Afghanistan  2006    sp     m   014   193
## # ... with 76,036 more rows
```

Tidying the who data in one pipeline:

```r
who %>% 
  gather(code, value, new_sp_m014:newrel_f65, na.rm = TRUE) %>% 
  mutate(code = stringr::str_replace(code, "newrel", "new_rel")) %>% 
  separate(code, c("new", "var", "sexage")) %>% 
  select(-new, -iso2, -iso3) %>% 
  separate(sexage, c("sex", "age"), sep = 1)
```

```
## # A tibble: 76,046 × 6
##        country  year   var   sex   age value
## *        <chr> <int> <chr> <chr> <chr> <int>
## 1  Afghanistan  1997    sp     m   014     0
## 2  Afghanistan  1998    sp     m   014    30
## 3  Afghanistan  1999    sp     m   014     8
## 4  Afghanistan  2000    sp     m   014    52
## 5  Afghanistan  2001    sp     m   014   129
## 6  Afghanistan  2002    sp     m   014    90
## 7  Afghanistan  2003    sp     m   014   127
## 8  Afghanistan  2004    sp     m   014   139
## 9  Afghanistan  2005    sp     m   014   151
## 10 Afghanistan  2006    sp     m   014   193
## # ... with 76,036 more rows
```

#### 12.6.1 Exercises

1. Removing missing values from this dataset seems reasonable. Missing values are shown as "NA" and there don't appear to be any implicit missing values. NA indicates that the value is missing while 0 indicates that there were no cases within that category.

2. If you neglect the `mutate()` setp, all of the gathered columns are just removed from the data frame.

```r
who %>% 
  gather(code, value, new_sp_m014:newrel_f65, na.rm = TRUE) %>% 
  separate(code, c("new", "var", "sexage")) %>% 
  select(-new, -iso2, -iso3) %>% 
  separate(sexage, c("sex", "age"), sep = 1)
```

```
## Warning: Too few values at 2580 locations: 73467, 73468, 73469, 73470,
## 73471, 73472, 73473, 73474, 73475, 73476, 73477, 73478, 73479, 73480,
## 73481, 73482, 73483, 73484, 73485, 73486, ...
```

```
## # A tibble: 76,046 × 6
##        country  year   var   sex   age value
## *        <chr> <int> <chr> <chr> <chr> <int>
## 1  Afghanistan  1997    sp     m   014     0
## 2  Afghanistan  1998    sp     m   014    30
## 3  Afghanistan  1999    sp     m   014     8
## 4  Afghanistan  2000    sp     m   014    52
## 5  Afghanistan  2001    sp     m   014   129
## 6  Afghanistan  2002    sp     m   014    90
## 7  Afghanistan  2003    sp     m   014   127
## 8  Afghanistan  2004    sp     m   014   139
## 9  Afghanistan  2005    sp     m   014   151
## 10 Afghanistan  2006    sp     m   014   193
## # ... with 76,036 more rows
```

3. The who data documentation specifies that iso2 contains two letter ISO country codes and iso3 containes three letter ISO country codes, which provides the same information as country.

4. 

### 12.7 Non-tidy data

There may be reasons to use formats other than tidy data, but tidy data should be used whenever possible.

## Chapter 13 Relational Data

### 13.1 Introduction

Relational data are multiple data tables that are related to each other. Relations are analyzed between pairs of data tables.

To work with relational data use:

* mutating joins: add new variables to a data frame using matching observations
* filtering joins: filter observations from a data frame based on matching observations
* set operations: treat observations as set elements

### 13.2 nycflights13

Four tibbles are related to the `flights` data frame:

* airlines: carrier names and abbreviations
* airports: airport information and codes
* planes: plain information and tail numbers
* weather: weather per hour for all NYC airports

#### 13.2.1 Exercises

1. You would need the plane identification with `tailnum`, the airports it leaves from and arrives at with `origin` and `dest`, and the locations of those airports with `lat` and `long`. `flights` and `airports` would need to be combined for this.

2. `faa` should connect `origin` with an arrow drawn from `faa` to `origin`.

3. If `weather` included all airports in the USA, it would also define `dest` with `flights`. 

4. You could create a data frame of holidays, when fewer people fly. You may represent this in a data frame with month, day, name, and the organization associated with that holiday. This would connect to `flights` and `weather` with `month` and `day`.
