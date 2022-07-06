---
title: "Regression Model"
author: "Ganling"
date: "5/1/2021"
output: 
  html_document:
    keep_md: yes
---
#Load the libraries

```r
library(here)
```

```
## Warning: package 'here' was built under R version 4.0.4
```

```
## here() starts at G:/Shared drives/CEG Two-Stage Exams Analysis/Ganling/two-stage-2
```

```r
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------- tidyverse 1.3.0 --
```

```
## v ggplot2 3.3.2     v purrr   0.3.4
## v tibble  3.0.4     v dplyr   1.0.2
## v tidyr   1.1.2     v stringr 1.4.0
## v readr   1.4.0     v forcats 0.5.0
```

```
## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(skimr)
library(moderndive)
library(ggplot2)
library(dplyr)
library(readr)
```
#Load the data using the directory 

```r
proj_dir <- here()
#add original original data directory sting
original_data_dir <- here("original-data", "/")
#add numbers to importable and analysis data directory strings
importable_data_dir <- here("processing-and-analysis", "01-importable-data", "/")
#add analysis directory string
analysis_data_dir <- here("processing-and-analysis", "03-analysis-data", "/")
metadata_dir <- here("original-data", "metadata", "/")
```

Copy data from `original-data` to `01-importable-data`


```r
# CFC: Copy rds file from the `original_data_dir` to the `importable_data_dir`.
copy_from <- paste0(original_data_dir, "master_2s_small_deidentified.rds")
copy_to <- paste0(importable_data_dir, "master_2s_small_deidentified.rds")
# CFC: If the file already exists in the target directory, the file will not be 
# overwritten and this command will return `FALSE`.
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```


```r
## [1] FALSE
```

Read in data.


```r
#CFC: IMport data set from `importable_data_dir`
data_2s <- readRDS("G:/Shared drives/CEG Two-Stage Exams Analysis/Ganling/two-stage-2/processing-and-analysis/01-importable-data/master_2s_small_deidentified.rds")
#data_2s <- readRDS(paste0(importable_data_dir, "master_2s_small_deidenitified.rds")) (still does not work for some reason)
```

#Before starting to make historgrams and boxplots, I would like to remove all the duplicating rows and make a new data frame


```r
data_2s_extras_removed = subset(data_2s, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)
View(data_2s_unique)
data_2s_exam_score <- data_2s_unique %>% 
  select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc)
  
data_2s_exam_score_true <- na.omit(data_2s_exam_score)

data_2s_leveled <- data_2s_exam_score_true %>% 
    mutate(course_fullid = factor(course_fullid, 
        levels = c("CHEM_142_A_2016_4", "CHEM_142_B_2016_4", 
                   "CHEM_142_A_2017_4", "CHEM_142_B_2017_4")))
```
#Now, I want to use regression model to see the exam score difference between experiement year and the non experiment year

```r
exam1_regression <- data_2s_leveled %>%
  select(course_fullid, exam1)
#now lets find mean and median scores of exam 1
exam1_regression_stats <- exam1_regression %>%
  group_by(course_fullid) %>%
  summarize(exam1_median = median(exam1), 
            exam1_mean = mean(exam1))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table for exam 1 
exam1_model <- lm(exam1 ~ course_fullid, data = exam1_regression)
get_regression_table(exam1_model)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 68.7       0.42    163.      0        67.8     69.5 
## 2 course_fullidCHEM_142_~   -0.015     0.582    -0.026   0.979    -1.16     1.13
## 3 course_fullidCHEM_142_~   -2.43      0.56     -4.34    0        -3.53    -1.33
## 4 course_fullidCHEM_142_~   -2.60      0.564    -4.61    0        -3.70    -1.49
```

```r
#making a data set frame for exam 2 regression
exam2_regression <- data_2s_leveled %>%
  select(course_fullid, exam2)
#Same process for exam 2
exam2_regression_stats <- exam2_regression %>%
  group_by(course_fullid) %>%
  summarize(exam2_median = median(exam2), 
            exam2_mean = mean(exam2))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table
exam2_model <- lm(exam2 ~ course_fullid, data = exam2_regression)
get_regression_table(exam2_model)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  75.4      0.501    151.         0    74.4     76.4 
## 2 course_fullidCHEM_142_~     2.81     0.694      4.04       0     1.44     4.17
## 3 course_fullidCHEM_142_~    -8.44     0.667    -12.7        0    -9.75    -7.13
## 4 course_fullidCHEM_142_~    -7.92     0.672    -11.8        0    -9.23    -6.60
```

```r
#making a data set frame for final exam regression
finalexam_regression <- data_2s_leveled %>%
  select(course_fullid, finalexam)
#Same process for exam 2
finalexam_regression_stats <- finalexam_regression %>%
  group_by(course_fullid) %>%
  summarize(finalexam_median = median(finalexam), 
            finalexam_mean = mean(finalexam))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table
finalexam_model <- lm(finalexam ~ course_fullid, data = finalexam_regression)
get_regression_table(finalexam_model)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                106.        0.619   172.      0       105.    107.   
## 2 course_fullidCHEM_142_~    3.28      0.858     3.82    0         1.60    4.96 
## 3 course_fullidCHEM_142_~   -1.53      0.825    -1.85    0.064    -3.14    0.091
## 4 course_fullidCHEM_142_~    0.425     0.831     0.512   0.609    -1.20    2.05
```
#now I would like to see the interaction between the scores when we control for sex

```r
exam1_regression_sex <- data_2s_leveled %>%
  select(course_fullid, exam1, sex_id)

#now lets get the regression table for exam 1 control for sex
exam1_sex_model <- lm(exam1 ~ course_fullid * sex_id, data = exam1_regression_sex)
get_regression_table(exam1_sex_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  66.5      0.571    116.     0        65.4    67.6  
## 2 course_fullidCHEM_142_~     1.08     0.778      1.40   0.163    -0.44    2.61 
## 3 course_fullidCHEM_142_~    -1.76     0.755     -2.33   0.02     -3.24   -0.279
## 4 course_fullidCHEM_142_~    -2.93     0.757     -3.87   0        -4.42   -1.45 
## 5 sex_idMale                  4.68     0.835      5.60   0         3.04    6.32 
## 6 course_fullidCHEM_142_~    -2.14     1.16      -1.84   0.066    -4.42    0.145
## 7 course_fullidCHEM_142_~    -1.30     1.12      -1.17   0.242    -3.49    0.88 
## 8 course_fullidCHEM_142_~     1.07     1.12       0.95   0.342    -1.14    3.27
```

```r
exam2_regression_sex <- data_2s_leveled %>%
  select(course_fullid, exam2, sex_id)

#now lets get the regression table for exam 2 control for sex
exam2_sex_model <- lm(exam2 ~ course_fullid * sex_id, data = exam2_regression_sex)
get_regression_table(exam2_sex_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 73.5       0.676   109.      0       72.2      74.8 
## 2 course_fullidCHEM_142_~    2.61      0.922     2.84    0.005    0.807     4.42
## 3 course_fullidCHEM_142_~   -9.00      0.895   -10.1     0      -10.8      -7.24
## 4 course_fullidCHEM_142_~  -10.3       0.897   -11.5     0      -12.1      -8.53
## 5 sex_idMale                 4.16      0.99      4.20    0        2.22      6.10
## 6 course_fullidCHEM_142_~    0.848     1.38      0.615   0.538   -1.85      3.55
## 7 course_fullidCHEM_142_~    1.42      1.32      1.07    0.283   -1.17      4.01
## 8 course_fullidCHEM_142_~    5.68      1.33      4.27    0        3.07      8.30
```

```r
finalexam_regression_sex <- data_2s_leveled %>%
  select(course_fullid, finalexam, sex_id)

#now lets get the regression table for final exam control for sex
finalexam_sex_model <- lm(finalexam ~ course_fullid * sex_id, data = finalexam_regression_sex)
get_regression_table(finalexam_sex_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                103.        0.838   123.      0      101.      104.  
## 2 course_fullidCHEM_142_~    4.33      1.14      3.79    0        2.09      6.57
## 3 course_fullidCHEM_142_~   -0.6       1.11     -0.541   0.589   -2.77      1.57
## 4 course_fullidCHEM_142_~   -0.465     1.11     -0.418   0.676   -2.64      1.72
## 5 sex_idMale                 7.43      1.23      6.06    0        5.03      9.84
## 6 course_fullidCHEM_142_~   -1.75      1.71     -1.03    0.305   -5.10      1.60
## 7 course_fullidCHEM_142_~   -1.75      1.64     -1.07    0.284   -4.96      1.46
## 8 course_fullidCHEM_142_~    2.50      1.65      1.52    0.129   -0.731     5.74
```
#EOP status and score interaction

```r
exam1_regression_eop <- data_2s_leveled %>%
  select(course_fullid, exam1, eop_id)

#now lets get the regression table for exam 1 control for sex
exam1_eop_model <- lm(exam1 ~ course_fullid * eop_id, data = 
  exam1_regression_eop)
get_regression_table(exam1_eop_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 61.5       0.868    70.8     0        59.8    63.2  
## 2 course_fullidCHEM_142_~    0.222     1.28      0.174   0.862    -2.29    2.73 
## 3 course_fullidCHEM_142_~   -1.05      1.12     -0.938   0.348    -3.23    1.14 
## 4 course_fullidCHEM_142_~   -1.30      1.16     -1.13    0.259    -3.56    0.961
## 5 eop_idnon-EOP              9.30      0.985     9.44    0         7.37   11.2  
## 6 course_fullidCHEM_142_~   -0.838     1.43     -0.585   0.558    -3.64    1.97 
## 7 course_fullidCHEM_142_~   -1.36      1.28     -1.06    0.289    -3.87    1.15 
## 8 course_fullidCHEM_142_~   -1.58      1.31     -1.20    0.228    -4.16    0.992
```

```r
exam2_regression_eop <- data_2s_leveled %>%
  select(course_fullid, exam2, eop_id)

#now lets get the regression table for exam 1 control for sex
exam2_eop_model <- lm(exam2 ~ course_fullid * eop_id, data = exam2_regression_eop)
get_regression_table(exam2_eop_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 66.4        1.03    64.6     0        64.4     68.4 
## 2 course_fullidCHEM_142_~    1.73       1.52     1.14    0.256    -1.25     4.70
## 3 course_fullidCHEM_142_~   -7.28       1.32    -5.51    0        -9.88    -4.69
## 4 course_fullidCHEM_142_~   -6.57       1.37    -4.80    0        -9.26    -3.89
## 5 eop_idnon-EOP             11.6        1.17     9.94    0         9.33    13.9 
## 6 course_fullidCHEM_142_~    0.625      1.70     0.368   0.713    -2.7      3.95
## 7 course_fullidCHEM_142_~   -0.916      1.52    -0.603   0.546    -3.89     2.06
## 8 course_fullidCHEM_142_~   -1.62       1.56    -1.04    0.298    -4.68     1.43
```

```r
finalexam_regression_eop <- data_2s_leveled %>%
  select(course_fullid, finalexam, eop_id)

#now lets get the regression table for exam 1 control for sex
finalexam_eop_model <- lm(finalexam ~ course_fullid * eop_id, data = finalexam_regression_eop)
get_regression_table(finalexam_eop_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  93.8       1.27    73.6     0       91.3     96.3  
## 2 course_fullidCHEM_142_~     7.04      1.88     3.74    0        3.36    10.7  
## 3 course_fullidCHEM_142_~     1.29      1.64     0.788   0.431   -1.92     4.51 
## 4 course_fullidCHEM_142_~     3.90      1.70     2.30    0.022    0.571    7.22 
## 5 eop_idnon-EOP              16.0       1.45    11.1     0       13.2     18.9  
## 6 course_fullidCHEM_142_~    -5.52      2.10    -2.62    0.009   -9.64    -1.40 
## 7 course_fullidCHEM_142_~    -2.93      1.88    -1.55    0.12    -6.62     0.764
## 8 course_fullidCHEM_142_~    -4.35      1.93    -2.25    0.024   -8.13    -0.561
```
#fgn status and exam score 

```r
exam1_regression_fgn <- data_2s_leveled %>%
  select(course_fullid, exam1, fgn_id)

#now lets get the regression table for exam 1 control for sex
exam1_fgn_model <- lm(exam1 ~ course_fullid * fgn_id, data = exam1_regression_fgn)
get_regression_table(exam1_fgn_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 63.0       0.695    90.6     0       61.6     64.3  
## 2 course_fullidCHEM_142_~    0.624     1.05      0.595   0.552   -1.43     2.68 
## 3 course_fullidCHEM_142_~   -5.49      0.971    -5.66    0       -7.40    -3.59 
## 4 course_fullidCHEM_142_~   -4.38      0.99     -4.43    0       -6.32    -2.44 
## 5 fgn_idnon-FGN              8.61      0.856    10.1     0        6.93    10.3  
## 6 course_fullidCHEM_142_~   -1.92      1.25     -1.54    0.123   -4.36     0.522
## 7 course_fullidCHEM_142_~    3.50      1.17      2.99    0.003    1.20     5.79 
## 8 course_fullidCHEM_142_~    1.56      1.19      1.31    0.19    -0.771    3.88
```

```r
exam2_regression_fgn <- data_2s_leveled %>%
  select(course_fullid, exam2, fgn_id)

#now lets get the regression table for exam 1 control for sex
exam2_fgn_model <- lm(exam2 ~ course_fullid * fgn_id, data = exam2_regression_fgn)
get_regression_table(exam2_fgn_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 68.7        0.83    82.7     0        67.0    70.3  
## 2 course_fullidCHEM_142_~    3.89       1.25     3.11    0.002     1.44    6.34 
## 3 course_fullidCHEM_142_~  -12.9        1.16   -11.1     0       -15.1   -10.6  
## 4 course_fullidCHEM_142_~   -8.46       1.18    -7.16    0       -10.8    -6.14 
## 5 fgn_idnon-FGN             10.2        1.02    10.0     0         8.23   12.2  
## 6 course_fullidCHEM_142_~   -2.71       1.49    -1.82    0.068    -5.62    0.205
## 7 course_fullidCHEM_142_~    5.24       1.40     3.75    0         2.5     7.98 
## 8 course_fullidCHEM_142_~   -0.308      1.42    -0.218   0.828    -3.09    2.47
```

```r
finalexam_regression_fgn <- data_2s_leveled %>%
  select(course_fullid, finalexam, fgn_id)

#now lets get the regression table for exam 1 control for sex
finalexam_fgn_model <- lm(finalexam ~ course_fullid * fgn_id, data = finalexam_regression_fgn)
get_regression_table(finalexam_fgn_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  98.0       1.02     95.9    0       96.0    100.   
## 2 course_fullidCHEM_142_~     4.82      1.54      3.12   0.002    1.79     7.84 
## 3 course_fullidCHEM_142_~    -6.70      1.43     -4.69   0       -9.50    -3.90 
## 4 course_fullidCHEM_142_~    -3.25      1.46     -2.23   0.026   -6.11    -0.398
## 5 fgn_idnon-FGN              12.4       1.26      9.82   0        9.9     14.8  
## 6 course_fullidCHEM_142_~    -3.58      1.83     -1.96   0.051   -7.17     0.01 
## 7 course_fullidCHEM_142_~     6.10      1.72      3.55   0        2.73     9.48 
## 8 course_fullidCHEM_142_~     3.74      1.75      2.14   0.032    0.323    7.17
```
#URM status and score interaction

```r
exam1_regression_urm <- data_2s_leveled %>%
  select(course_fullid, exam1, urm_id)

#now lets get the regression table for exam 1 control for sex
exam1_urm_model <- lm(exam1 ~ course_fullid * urm_id, data = exam1_regression_urm)
get_regression_table(exam1_urm_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 69.6       0.446   156.      0        68.8    70.5  
## 2 course_fullidCHEM_142_~   -0.35      0.615    -0.569   0.569    -1.56    0.856
## 3 course_fullidCHEM_142_~   -1.96      0.6      -3.26    0.001    -3.13   -0.78 
## 4 course_fullidCHEM_142_~   -2.73      0.594    -4.60    0        -3.90   -1.57 
## 5 urm_idURM                 -7.32      1.23     -5.97    0        -9.73   -4.92 
## 6 course_fullidCHEM_142_~    1.73      1.76      0.985   0.324    -1.71    5.18 
## 7 course_fullidCHEM_142_~   -1.22      1.57     -0.781   0.435    -4.30    1.85 
## 8 course_fullidCHEM_142_~   -0.458     1.72     -0.267   0.79     -3.83    2.91
```

```r
exam2_regression_urm <- data_2s_leveled %>%
  select(course_fullid, exam2, urm_id)

#now lets get the regression table for exam 1 control for sex
exam2_urm_model <- lm(exam2 ~ course_fullid * urm_id, data = exam2_regression_urm)
get_regression_table(exam2_urm_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  76.9      0.53    145.      0       75.9      77.9 
## 2 course_fullidCHEM_142_~     2.10     0.731     2.88    0.004    0.67      3.54
## 3 course_fullidCHEM_142_~    -8.10     0.713   -11.4     0       -9.50     -6.70
## 4 course_fullidCHEM_142_~    -8.40     0.707   -11.9     0       -9.78     -7.01
## 5 urm_idURM                 -11.2      1.46     -7.68    0      -14.1      -8.34
## 6 course_fullidCHEM_142_~     4.33     2.09      2.07    0.038    0.234     8.43
## 7 course_fullidCHEM_142_~     0.43     1.86      0.231   0.818   -3.22      4.09
## 8 course_fullidCHEM_142_~     1.87     2.04      0.913   0.361   -2.14      5.87
```

```r
finalexam_regression_urm <- data_2s_leveled %>%
  select(course_fullid, finalexam, urm_id)

#now lets get the regression table for exam 1 control for sex
finalexam_urm_model <- lm(finalexam ~ course_fullid * urm_id, data = finalexam_regression_urm)
get_regression_table(finalexam_urm_model)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                108.        0.654   166.      0      107.     110.   
## 2 course_fullidCHEM_142_~    1.77      0.903     1.96    0.05     0.004    3.54 
## 3 course_fullidCHEM_142_~   -1.71      0.88     -1.94    0.052   -3.43     0.018
## 4 course_fullidCHEM_142_~   -0.444     0.873    -0.508   0.611   -2.15     1.27 
## 5 urm_idURM                -17.1       1.80     -9.49    0      -20.6    -13.6  
## 6 course_fullidCHEM_142_~   10.4       2.58      4.04    0        5.37    15.5  
## 7 course_fullidCHEM_142_~    4.8       2.30      2.08    0.037    0.288    9.31 
## 8 course_fullidCHEM_142_~    4.08      2.52      1.62    0.106   -0.865    9.02
```
Next, I need to do multiple regression tables for two numerical values 

```r
#math SAT scores and exam 1 score 
exam1_regression_satm <- data_2s_leveled %>% 
  select(course_fullid, exam1, satmath)
ggplot(exam1_regression_satm, aes(x = satmath, y = exam1, color = course_fullid)) +
  geom_point() +
  labs(x = "SAT Math Score", y = "Exam 1 score", color = "Class") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_files/figure-html/unnamed-chunk-12-1.png)<!-- -->









