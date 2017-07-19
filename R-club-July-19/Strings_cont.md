# Strings and Regular Expressions cont.
CB  
Tuesday, July 18, 2017  


## Chapter 14 Strings

### 14.4 Tools

#### 14.4.1 Detect matches

`str_detect()` can find matches within character vectors.

```r
x <- c("apple", "banana", "pear")
str_detect(x, "e")
```

```
## [1]  TRUE FALSE  TRUE
```

Logical vectors are useful for counting matches.

```r
sum(str_detect(words, "^t"))
```

```
## [1] 65
```

```r
mean(str_detect(words, "[aeiou]$"))
```

```
## [1] 0.2765306
```

Regular expressions can be broken up into smaller pieces to make them easier to work with and understand rather than using one long and complex regular expression.

`str_detect()` within a subsetting function works the same as `str_subset()`.

```r
words[str_detect(words, "x$")]
```

```
## [1] "box" "sex" "six" "tax"
```

```r
str_subset(words, "x$")
```

```
## [1] "box" "sex" "six" "tax"
```

`str_detect()` can also be used in conjunction with `filter()`.

```r
df <- tibble(
  word = words,
  i = seq_along(word)
)
df %>% 
  filter(str_detect(words, "x$"))
```

```
## # A tibble: 4 × 2
##    word     i
##   <chr> <int>
## 1   box   108
## 2   sex   747
## 3   six   772
## 4   tax   841
```

`str_count()` is like `str_detect()` but counts the number of matches within the string.

```r
x <- c("apple", "banana", "pear")
str_count(x, "a")
```

```
## [1] 1 3 1
```

`str_count()` can be used with `mutate()`. 

```r
df %>% 
  mutate(
    vowels = str_count(word, "[aeiou]"),
    consonants = str_count(word, "[^aeiou]")
  )
```

```
## # A tibble: 980 × 4
##        word     i vowels consonants
##       <chr> <int>  <int>      <int>
## 1         a     1      1          0
## 2      able     2      2          2
## 3     about     3      3          2
## 4  absolute     4      4          4
## 5    accept     5      2          4
## 6   account     6      3          4
## 7   achieve     7      4          3
## 8    across     8      2          4
## 9       act     9      1          2
## 10   active    10      3          3
## # ... with 970 more rows
```

String matches don't overlap.

```r
str_count("abababa", "aba")
```

```
## [1] 2
```

```r
str_view_all("abababa", "aba")
```

<!--html_preserve--><div id="htmlwidget-12a6078f5735f14536bf" style="width:960px;height:auto;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-12a6078f5735f14536bf">{"x":{"html":"<ul>\n  <li><span class='match'>aba<\/span>b<span class='match'>aba<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Many stringr functions are paired so that one function works with one match and the other function works with all matches.

#### 14.4.2 Exercises

##### 1

1. all words that start or end with `x`

```r
# single regular expression
(single <- str_subset(words, "^x|x$"))
```

```
## [1] "box" "sex" "six" "tax"
```

```r
# multiple str_detect() calls
x_start <- str_detect(words, "^x")
x_end <- str_detect(words, "x$")
(multiple <- words[x_start | x_end])
```

```
## [1] "box" "sex" "six" "tax"
```

```r
identical(single, multiple)
```

```
## [1] TRUE
```

2. all words that start with a vowel and end with a consonant

```r
# single regular expression
single <- str_subset(words, "^[aeiou].*[^aeiou]$")
head(single)
```

```
## [1] "about"   "accept"  "account" "across"  "act"     "actual"
```

```r
# multiple str_detect() calls
vowel_start <- str_detect(words, "^[aeiou]")
consonant_end <- str_detect(words, "[^aeiou]$")
multiple <- words[vowel_start & consonant_end]
head(multiple)
```

```
## [1] "about"   "accept"  "account" "across"  "act"     "actual"
```

```r
identical(single, multiple)
```

```
## [1] TRUE
```

3. words that contain at least one of each vowel

```r
# single regular expression (from stackoverflow)
vowels <- "^(?=.*a)(?=.*e)(?=.*i)(?=.*o)(?=.*u).*$"
(single <-  str_subset(words, vowels))
```

```
## character(0)
```

```r
# multiple str_detect() calls
a <- str_detect(words, "a")
e <- str_detect(words, "e")
i <- str_detect(words, "i")
o <- str_detect(words, "o")
u <- str_detect(words, "u")
(multiple <- words[a & e & i & o & u])
```

```
## character(0)
```

```r
identical(single, multiple)
```

```
## [1] TRUE
```

```r
# did they work?
test <- c("easiloud", "aeiou", "around")
str_subset(test, vowels)
```

```
## [1] "easiloud" "aeiou"
```

```r
a_test <- str_detect(test, "a")
e_test <- str_detect(test, "e")
i_test <- str_detect(test, "i")
o_test <- str_detect(test, "o")
u_test <- str_detect(test, "u")
test[a_test & e_test & i_test & o_test & u_test]
```

```
## [1] "easiloud" "aeiou"
```

##### 2

What word has the highest number of vowels?

```r
df <- tibble(
  word = words
)

df %>% 
  mutate(
    vowels = str_count(word, "[aeiou]")
  ) %>% 
  arrange(desc(vowels)) %>% 
  head()
```

```
## # A tibble: 6 × 2
##          word vowels
##         <chr>  <int>
## 1 appropriate      5
## 2   associate      5
## 3   available      5
## 4   colleague      5
## 5   encourage      5
## 6  experience      5
```

```r
# all words with 5 (most) vowels
df %>% 
  mutate(
    vowels = str_count(word, "[aeiou]")
  ) %>% 
  arrange(desc(vowels)) %>% 
  filter(vowels == 5)
```

```
## # A tibble: 8 × 2
##          word vowels
##         <chr>  <int>
## 1 appropriate      5
## 2   associate      5
## 3   available      5
## 4   colleague      5
## 5   encourage      5
## 6  experience      5
## 7  individual      5
## 8  television      5
```

What word has the highest proportion of vowels? 

```r
df %>% 
  mutate(
    vowels = str_count(word, "[aeiou]"),
    vowel_proportion = vowels/str_length(word)
  ) %>% 
  arrange(desc(vowel_proportion)) %>% 
  head()
```

```
## # A tibble: 6 × 3
##    word vowels vowel_proportion
##   <chr>  <int>            <dbl>
## 1     a      1        1.0000000
## 2  area      3        0.7500000
## 3  idea      3        0.7500000
## 4   age      2        0.6666667
## 5   ago      2        0.6666667
## 6   air      2        0.6666667
```

#### 14.4.3 Extract matches

`str_extract()` will extract matched text, but only the first match.

```r
colors <- c("red", "orange", "yellow", "green", "blue", "purple")
(color_match <- str_c(colors, collapse = "|"))
```

```
## [1] "red|orange|yellow|green|blue|purple"
```

```r
has_color <- str_subset(sentences, color_match)
matches <- str_extract(has_color, color_match)
head(matches)
```

```
## [1] "blue" "blue" "red"  "red"  "red"  "blue"
```

Use `str_extract_all()` to extract all matches.

```r
more <- sentences[str_count(sentences, color_match) > 1]
str_view_all(more, color_match)
```

<!--html_preserve--><div id="htmlwidget-12a6078f5735f14536bf" style="width:960px;height:auto;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-12a6078f5735f14536bf">{"x":{"html":"<ul>\n  <li>It is hard to erase <span class='match'>blue<\/span> or <span class='match'>red<\/span> ink.<\/li>\n  <li>The <span class='match'>green<\/span> light in the brown box flicke<span class='match'>red<\/span>.<\/li>\n  <li>The sky in the west is tinged with <span class='match'>orange<\/span> <span class='match'>red<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_extract_all(more, color_match)
```

```
## [[1]]
## [1] "blue" "red" 
## 
## [[2]]
## [1] "green" "red"  
## 
## [[3]]
## [1] "orange" "red"
```

`simplify = TRUE` will return a matrix instead of a list.

```r
x <- c("a", "a b", "a b c")
str_extract_all(x, "[a-z]", simplify = TRUE)
```

```
##      [,1] [,2] [,3]
## [1,] "a"  ""   ""  
## [2,] "a"  "b"  ""  
## [3,] "a"  "b"  "c"
```

##### 14.4.3.1 Exercises

###### 1 

Only match colors and not colors within other words.

```r
color_match <- "\\b(red|orange|yellow|green|blue|purple)\\b"

more <- sentences[str_count(sentences, color_match) > 1]
str_view_all(more, color_match)
```

<!--html_preserve--><div id="htmlwidget-72f3c36e383eb39eddcc" style="width:960px;height:auto;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-72f3c36e383eb39eddcc">{"x":{"html":"<ul>\n  <li>It is hard to erase <span class='match'>blue<\/span> or <span class='match'>red<\/span> ink.<\/li>\n  <li>The sky in the west is tinged with <span class='match'>orange<\/span> <span class='match'>red<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

###### 2

1. Extract the first word from each sentence

```r
str_extract(sentences, "^\\w+") %>% head()
```

```
## [1] "The"   "Glue"  "It"    "These" "Rice"  "The"
```

2. Extract all words ending in `ing`

```r
ing_words <- "\\w+ing\\b"
has_ing <- str_subset(sentences, ing_words)
str_extract_all(has_ing, ing_words) %>% head()
```

```
## [[1]]
## [1] "spring"
## 
## [[2]]
## [1] "evening"
## 
## [[3]]
## [1] "morning"
## 
## [[4]]
## [1] "winding"
## 
## [[5]]
## [1] "living"
## 
## [[6]]
## [1] "king"
```

3. Extract all plurals

```r
plurals <- "\\w{2,}[b-hj-np-rtv-z]s\\b"
has_plurals <- str_subset(sentences, plurals)
str_extract_all(has_plurals, plurals) %>% head()
```

```
## [[1]]
## [1] "planks"
## 
## [[2]]
## [1] "days"
## 
## [[3]]
## [1] "bowls"
## 
## [[4]]
## [1] "lemons" "makes" 
## 
## [[5]]
## [1] "hogs"
## 
## [[6]]
## [1] "hours"
```

#### 14.4.4 Grouped matches

Extract nouns from sentences.

```r
noun <- "(a|the) ([^ ]+)"

has_noun <- sentences %>% 
  str_subset(noun) %>% 
  head(10)

has_noun %>% 
  str_extract(noun)
```

```
##  [1] "the smooth" "the sheet"  "the depth"  "a chicken"  "the parked"
##  [6] "the sun"    "the huge"   "the ball"   "the woman"  "a helps"
```

`str_match()` splits matches into groups.

```r
has_noun %>% 
  str_match(noun)
```

```
##       [,1]         [,2]  [,3]     
##  [1,] "the smooth" "the" "smooth" 
##  [2,] "the sheet"  "the" "sheet"  
##  [3,] "the depth"  "the" "depth"  
##  [4,] "a chicken"  "a"   "chicken"
##  [5,] "the parked" "the" "parked" 
##  [6,] "the sun"    "the" "sun"    
##  [7,] "the huge"   "the" "huge"   
##  [8,] "the ball"   "the" "ball"   
##  [9,] "the woman"  "the" "woman"  
## [10,] "a helps"    "a"   "helps"
```

`tidyr::extract()` is easier to use with tibbles.

```r
tibble(sentence = sentences) %>% 
  tidyr::extract(
    sentence, c("article", "noun"), "(a|the) ([^ ]+)",
    remove = FALSE
  )
```

```
## # A tibble: 720 × 3
##                                       sentence article    noun
## *                                        <chr>   <chr>   <chr>
## 1   The birch canoe slid on the smooth planks.     the  smooth
## 2  Glue the sheet to the dark blue background.     the   sheet
## 3       It's easy to tell the depth of a well.     the   depth
## 4     These days a chicken leg is a rare dish.       a chicken
## 5         Rice is often served in round bowls.    <NA>    <NA>
## 6        The juice of lemons makes fine punch.    <NA>    <NA>
## 7  The box was thrown beside the parked truck.     the  parked
## 8  The hogs were fed chopped corn and garbage.    <NA>    <NA>
## 9          Four hours of steady work faced us.    <NA>    <NA>
## 10    Large size in stockings is hard to sell.    <NA>    <NA>
## # ... with 710 more rows
```

##### 14.4.4.1 Exercises

1. Pull out numbers and the words that come after them.

```r
number_word <- "\\b(one|two|three|four|five|six|seven|eight|nine|ten) (\\w+)"
has_nw <- sentences %>% 
  str_subset(number_word)
has_nw %>% 
  str_match_all(number_word) %>% 
  head()
```

```
## [[1]]
##      [,1]          [,2]    [,3]   
## [1,] "seven books" "seven" "books"
## 
## [[2]]
##      [,1]      [,2]  [,3] 
## [1,] "two met" "two" "met"
## 
## [[3]]
##      [,1]          [,2]  [,3]     
## [1,] "two factors" "two" "factors"
## 
## [[4]]
##      [,1]          [,2]    [,3]   
## [1,] "three lists" "three" "lists"
## 
## [[5]]
##      [,1]       [,2]    [,3]
## [1,] "seven is" "seven" "is"
## 
## [[6]]
##      [,1]       [,2]  [,3]  
## [1,] "two when" "two" "when"
```

2. Find all contractions and separate the parts around the apostrophe.

```r
contraction <- "([A-Za-z]+)'([a-z]+)"
has_con <- sentences %>% 
  str_subset(contraction)
has_con %>% 
  str_match_all(contraction) %>% 
  head()
```

```
## [[1]]
##      [,1]   [,2] [,3]
## [1,] "It's" "It" "s" 
## 
## [[2]]
##      [,1]    [,2]  [,3]
## [1,] "man's" "man" "s" 
## 
## [[3]]
##      [,1]    [,2]  [,3]
## [1,] "don't" "don" "t" 
## 
## [[4]]
##      [,1]      [,2]    [,3]
## [1,] "store's" "store" "s" 
## 
## [[5]]
##      [,1]        [,2]      [,3]
## [1,] "workmen's" "workmen" "s" 
## 
## [[6]]
##      [,1]    [,2]  [,3]
## [1,] "Let's" "Let" "s"
```

#### 14.4.5 Replacing matches

`str_replace()` will replace matches with something new.

```r
x <- c("apple", "pear", "banana")
str_replace(x, "[aeiou]", "-")
```

```
## [1] "-pple"  "p-ar"   "b-nana"
```

```r
str_replace_all(x, "[aeiou]", "-")
```

```
## [1] "-ppl-"  "p--r"   "b-n-n-"
```

```r
x <- c("1 house", "2 cars", "3 people")
str_replace_all(x, c("1" = "one", "2" = "two", "3" = "three"))
```

```
## [1] "one house"    "two cars"     "three people"
```

Backreferences can also be used for replacements.

```r
sentences %>% 
  str_replace("([^ ]+) ([^ ]+) ([^ ]+)", "\\1 \\3 \\2") %>% 
  head(3)
```

```
## [1] "The canoe birch slid on the smooth planks." 
## [2] "Glue sheet the to the dark blue background."
## [3] "It's to easy tell the depth of a well."
```

##### 14.4.5.1 Exercises

1. Replace forward slashes with backslashes

```r
x <- c("rat/mouse", "cat/dog", "bird/bat")
str_replace(x, "/", "\\")
```

```
## [1] "ratmouse" "catdog"   "birdbat"
```

```r
str_replace(x, "/", "\\\\")
```

```
## [1] "rat\\mouse" "cat\\dog"   "bird\\bat"
```

```r
# can't get just one backslash as a replacement
```

2. Use `replace_all()` for `str_to_lower()`

```r
# not sure how to translate to the correct lowercase character
#sentences %>% 
 # str_replace_all("([A-Z])", "") %>% 
  #head()
```

3.  Switch first and last letter in `words`

```r
head(words)
```

```
## [1] "a"        "able"     "about"    "absolute" "accept"   "account"
```

```r
str_replace(words, "^(\\w)(\\w*)(\\w)$", "\\3\\2\\1") %>% 
  head()
```

```
## [1] "a"        "ebla"     "tboua"    "ebsoluta" "tccepa"   "tccouna"
```
WOrds with the same first and last letter would still be words.

#### 14.4.6 Splitting

`str_split()` will split strings into pices. 

```r
sentences %>% 
  head(3) %>% 
  str_split(" ")
```

```
## [[1]]
## [1] "The"     "birch"   "canoe"   "slid"    "on"      "the"     "smooth" 
## [8] "planks."
## 
## [[2]]
## [1] "Glue"        "the"         "sheet"       "to"          "the"        
## [6] "dark"        "blue"        "background."
## 
## [[3]]
## [1] "It's"  "easy"  "to"    "tell"  "the"   "depth" "of"    "a"     "well."
```

```r
x <- "This is a sentence. This is another sentence."
str_view_all(x, boundary("word"))
```

<!--html_preserve--><div id="htmlwidget-12a6078f5735f14536bf" style="width:960px;height:auto;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-12a6078f5735f14536bf">{"x":{"html":"<ul>\n  <li><span class='match'>This<\/span> <span class='match'>is<\/span> <span class='match'>a<\/span> <span class='match'>sentence<\/span>. <span class='match'>This<\/span> <span class='match'>is<\/span> <span class='match'>another<\/span> <span class='match'>sentence<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

##### 14.4.6.1 Exercises

1. Split a string into individual components

```r
x <- "apples, pears, and bananas"
str_split(x, boundary("word"))
```

```
## [[1]]
## [1] "apples"  "pears"   "and"     "bananas"
```

2. Splitting by `boundary("word")` will remove punctuation when separating words, while splitting by `" "` includes the punctuation with the words they are next to.

3. Splitting with an empty string splits on each character. 

```r
str_split(x, "")
```

```
## [[1]]
##  [1] "a" "p" "p" "l" "e" "s" "," " " "p" "e" "a" "r" "s" "," " " "a" "n"
## [18] "d" " " "b" "a" "n" "a" "n" "a" "s"
```

#### 14.4.7 Find matches

`str_locate()` gives the positions of each match, which is useful for dealing with situations where other functions aren't working well.

### 14.5 Other types of pattern

`regexp()` parameters can control pattern matching.

`fixed()` matches exactly what is specified on a byte level. This may be faster than `regexp()` for simple matches.
`coll()` matches strings with collation rules and uses `locale` to compare characters. This is slower than `regexp()` and `fixed()`. 

#### 14.5.1 Exercises

1. To find all strings containing `\` with `regex()`, you would need to escape the backslash and make sure it is being recognized as a literal backslash. This would be easier to find with `fixed()` because it could recognize `\` without the need for escaping the special character.

2. Five most common words in `sentences`

```r
words <- str_split(sentences, boundary("word"), simplify = TRUE)
# how to match all words that are the same and then count?
```

