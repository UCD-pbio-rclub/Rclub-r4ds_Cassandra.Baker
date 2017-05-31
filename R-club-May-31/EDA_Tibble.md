# Exploratory Data Analysis cont. + Tibble
CB  
Tuesday, May 30, 2017  


## Chapter 7 Exploratory Data Analysis

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

## Chapter 8 Workflow: projects

R can be very useful for saving analyses and keeping multiple analyses separate. 

### 8.1 What is real?

It's important to remember that scripts are saved, but the environment in R is not. However, the environment can be recreated from saved scripts. 

You can check to make sure all important pieces of code are included in your script by restarting R and rerunning your script. 

* `Ctrl` + `Shift` + `F10` to restart RStudio
* `Ctrl` + `Shift` + `S` to rerun current script

### 8.2 Where does your analysis live?

It's useful to organize each separate project into its own directory and set that directory as the working directory when working on that project.

### 8.3 Paths and directories

Including an absolute path in a script makes it difficult to share the script with others, as that absolute file path will not work for them. 

### 8.4 RStudio projects

Projects within R are very useful for keeping all files associated with one project together in one place. 

If you save figures from within R using code instead of the GUI, it will be much easier to trace the origin of figure files in the future. Using an R markdown document instead of a script can also be useful for creating a workbook where the script and figures it creates are stored in one place. 

### 8.5 Summary

Be diligent with creating a separate R project for each analysis project. This will make sure all files and information you need for that project are stored together and easy to find. This will also help keep each analysis project separate from other analysis projects. 

