# Exploratory Data Analysis cont. + Tibble
CB  
Tuesday, May 30, 2017  


## Chapter 7 

### 7.6 Patterns and models

Patterns within data can be informative for determining relationships between variables. 


```r
ggplot(faithful) + 
  geom_point(aes(x = eruptions, y = waiting))
```

![](EDA_Tibble_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

Covariation can be used to reduce uncertainty because one variable can be used to predict values for the second variable.

Models can be used to separate variable effects and remove interfering relationships.


```r
library(modelr)

mod <- lm(log(price) ~ log(carat), diamonds)

diamonds2 <- diamonds %>% 
  add_residuals(mod) %>% 
  mutate(resid = exp(resid))

ggplot(diamonds2) + 
  geom_point(aes(x = carat, y = resid))
```

![](EDA_Tibble_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

Now see the expected relationship between cut and price without the effect of carat on price.

```r
ggplot(diamonds2) + 
  geom_boxplot(aes(x = cut, y = resid))
```

![](EDA_Tibble_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

### 7.7 `ggplot2` calls

Not all arguments need to be specified in `ggplot` functions, as `ggplot` assumes some arguments are in specific locations. For `ggplot()`, the first argument is assumed to be `data` and the second argument is assumed to be `mapping`. For `aes()`, the first argument is assumed to be `x` and the second argument is assumed to be `y`. 
```
ggplot(data = faithful, mapping = aes(x = eruptions)) + 
geom_freqpoly(binwidth = 0.25) 
```
```
ggplot(faithful, aes(eruptions)) + 
geom_freqpoly(binwidth = 0.25)
```

### 7.8 Learning more

Check out the ggplot2 book, *R Graphics Cookbook*, and *Graphical Data Analysis with R* for additional information.

