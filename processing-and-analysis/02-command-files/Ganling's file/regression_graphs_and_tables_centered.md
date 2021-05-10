---
title: "Regression_graphs_and_tables_centered"
author: "Ganling"
date: "5/10/2021"
output: 
  html_document:
    keep_md: yes
---
# Load the libraries

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
# Copy data from `original-data` to `01-importable-data`

```r
# CFC: Copy rds file from the `original_data_dir` to the `importable_data_dir`.
copy_from <- paste0(original_data_dir, "master_2s_small_deid_scaled.rds")
copy_to <- paste0(importable_data_dir, "master_2s_small_deid_scaled.rds")
# CFC: If the file already exists in the target directory, the file will not be 
# overwritten and this command will return `FALSE`.
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```
# read in the data file 

```r
data_2s <- readRDS(paste0(importable_data_dir, "master_2s_small_deid_scaled.rds"))
#now lets do some data wrangling to make sure everything runs
data_2s_extras_removed = subset(data_2s, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)
data_2s_exam_score <- data_2s_unique %>% 
  select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc, experiment1)
  
data_2s_exam_score_true <- na.omit(data_2s_exam_score)
#making sure the data control and experiment groups are leveled for further regression analysis
data_2s_leveled <- data_2s_exam_score_true %>% 
    mutate(experiment1 = factor(experiment1, 
        levels = c("CONTROL", "EXPERIMENTAL")))
```
#Now, I want to use regression model to see the exam score difference between experiement year and the non experiment year

```r
exam1_regression <- data_2s_leveled %>%
  select(experiment1, exam1)
#now lets find mean and median scores of exam 1
exam1_regression_stats <- exam1_regression %>%
  group_by(experiment1) %>%
  summarize(exam1_median = median(exam1), 
            exam1_mean = mean(exam1))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table for exam 1 
exam1_model <- lm(exam1 ~ experiment1, data = exam1_regression)
get_regression_table(exam1_model)
```

```
## # A tibble: 2 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  68.7      0.291    236.         0    68.1     69.2 
## 2 experiment1EXPERIMENTAL    -2.50     0.393     -6.38       0    -3.28    -1.74
```

```r
#making a data set frame for exam 2 regression
exam2_regression <- data_2s_leveled %>%
  select(experiment1, exam2)
#Same process for exam 2
exam2_regression_stats <- exam2_regression %>%
  group_by(experiment1) %>%
  summarize(exam2_median = median(exam2), 
            exam2_mean = mean(exam2))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table
exam2_model <- lm(exam2 ~ experiment1, data = exam2_regression)
get_regression_table(exam2_model)
```

```
## # A tibble: 2 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  76.9      0.347     221.        0     76.2    77.6 
## 2 experiment1EXPERIMENTAL    -9.64     0.469     -20.6       0    -10.6    -8.72
```

```r
#making a data set frame for final exam regression
finalexam_regression <- data_2s_leveled %>%
  select(experiment1, finalexam)
#Same process for exam 2
finalexam_regression_stats <- finalexam_regression %>%
  group_by(experiment1) %>%
  summarize(finalexam_median = median(finalexam), 
            finalexam_mean = mean(finalexam))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table
finalexam_model <- lm(finalexam ~ experiment1, data = finalexam_regression)
get_regression_table(finalexam_model)
```

```
## # A tibble: 2 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 108.       0.429    251.         0   107.     109.  
## 2 experiment1EXPERIMENTAL    -2.27     0.579     -3.92       0    -3.41    -1.13
```
# now I would like to see the interaction between the scores when we control for sex

```r
exam1_regression_sex <- data_2s_leveled %>%
  select(experiment1, exam1, sex_id)

#now lets get the regression table for exam 1 control for sex
exam1_sex_model <- lm(exam1 ~ experiment1 + sex_id, data = exam1_regression_sex)
get_regression_table(exam1_sex_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  70.9      0.361    197.         0    70.2     71.6 
## 2 experiment1EXPERIMENTAL    -2.50     0.39      -6.40       0    -3.26    -1.73
## 3 sex_idFemale               -4.10     0.39     -10.5        0    -4.87    -3.34
```

```r
#creating a model for exam 2
exam2_regression_sex <- data_2s_leveled %>%
  select(experiment1, exam2, sex_id)

#now lets get the regression table for exam 2 control for sex
exam2_sex_model <- lm(exam2 ~ experiment1 + sex_id, data = exam2_regression_sex)
get_regression_table(exam2_sex_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  80.3      0.429     187.        0    79.5     81.2 
## 2 experiment1EXPERIMENTAL    -9.63     0.463     -20.8       0   -10.5     -8.72
## 3 sex_idFemale               -6.23     0.464     -13.4       0    -7.14    -5.32
```

```r
finalexam_regression_sex <- data_2s_leveled %>%
  select(experiment1, finalexam, sex_id)

#now lets get the regression table for final exam control for sex
finalexam_sex_model <- lm(finalexam ~ experiment1 + sex_id, data = finalexam_regression_sex)
get_regression_table(finalexam_sex_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 112.       0.531    211.         0   111.     113.  
## 2 experiment1EXPERIMENTAL    -2.25     0.573     -3.93       0    -3.38    -1.13
## 3 sex_idFemale               -7.14     0.574    -12.4        0    -8.26    -6.01
```
# control for eop status 

```r
exam1_regression_eop <- data_2s_leveled %>%
  select(experiment1, exam1, eop_id)

#now lets get the regression table for exam 1 control for sex
exam1_eop_model <- lm(exam1 ~ experiment1 + eop_id, data = 
  exam1_regression_eop)
get_regression_table(exam1_eop_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  70.3      0.299    235.         0    69.7     70.9 
## 2 experiment1EXPERIMENTAL    -2.09     0.385     -5.42       0    -2.84    -1.33
## 3 eop_idEOP                  -8.25     0.457    -18.0        0    -9.14    -7.35
```

```r
exam2_regression_eop <- data_2s_leveled %>%
  select(experiment1, exam2, eop_id)

#now lets get the regression table for exam 1 control for sex
exam2_eop_model <- lm(exam2 ~ experiment1 + eop_id, data = exam2_regression_eop)
get_regression_table(exam2_eop_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  79.1      0.355     223.        0    78.4     79.8 
## 2 experiment1EXPERIMENTAL    -9.08     0.457     -19.9       0    -9.98    -8.18
## 3 eop_idEOP                 -11.1      0.542     -20.4       0   -12.1    -10.0
```

```r
finalexam_regression_eop <- data_2s_leveled %>%
  select(experiment1, finalexam, eop_id)

#now lets get the regression table for exam 1 control for sex
finalexam_eop_model <- lm(finalexam ~ experiment1 + eop_id, data = finalexam_regression_eop)
get_regression_table(finalexam_eop_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 110.       0.44     251.     0       110.    111.   
## 2 experiment1EXPERIMENTAL    -1.61     0.566     -2.84   0.004    -2.72   -0.501
## 3 eop_idEOP                 -13.0      0.673    -19.3    0       -14.3   -11.7
```
# Now we look for fgn status

```r
exam1_regression_fgn <- data_2s_leveled %>%
  select(experiment1, exam1, fgn_id)

#now lets get the regression table for exam 1 control for fgn status
exam1_fgn_model <- lm(exam1 ~ experiment1 + fgn_id, data = exam1_regression_fgn)
get_regression_table(exam1_fgn_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  71.4      0.307    233.         0    70.8     72.0 
## 2 experiment1EXPERIMENTAL    -2.69     0.38      -7.09       0    -3.44    -1.95
## 3 fgn_idFGN                  -9.53     0.42     -22.7        0   -10.4     -8.70
```

```r
exam2_regression_fgn <- data_2s_leveled %>%
  select(experiment1, exam2, fgn_id)

#now lets get the regression table for exam 2 control for fgn status
exam2_fgn_model <- lm(exam2 ~ experiment1 + fgn_id, data = exam2_regression_fgn)
get_regression_table(exam2_fgn_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  80.1      0.367     218.        0     79.4    80.8 
## 2 experiment1EXPERIMENTAL    -9.86     0.454     -21.7       0    -10.8    -8.97
## 3 fgn_idFGN                 -11.1      0.503     -22.1       0    -12.1   -10.1
```

```r
finalexam_regression_fgn <- data_2s_leveled %>%
  select(experiment1, finalexam, fgn_id)

#now lets get the regression table for final exam control for fgn status
finalexam_fgn_model <- lm(finalexam ~ experiment1 + fgn_id, data = finalexam_regression_fgn)
get_regression_table(finalexam_fgn_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 112.       0.452    248.         0   111.     113.  
## 2 experiment1EXPERIMENTAL    -2.56     0.559     -4.57       0    -3.65    -1.46
## 3 fgn_idFGN                 -14.4      0.619    -23.2        0   -15.6    -13.2
```
# URM status and score interaction

```r
exam1_regression_urm <- data_2s_leveled %>%
  select(experiment1, exam1, urm_id)

#now lets get the regression table for exam 1 control for sex
exam1_urm_model <- lm(exam1 ~ experiment1 + urm_id, data = exam1_regression_urm)
get_regression_table(exam1_urm_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  69.6      0.296    235.         0    69.0     70.1 
## 2 experiment1EXPERIMENTAL    -2.39     0.388     -6.14       0    -3.15    -1.62
## 3 urm_idURM                  -7.44     0.573    -13.0        0    -8.56    -6.32
```

```r
exam2_regression_urm <- data_2s_leveled %>%
  select(experiment1, exam2, urm_id)

#now lets get the regression table for exam 1 control for sex
exam2_urm_model <- lm(exam2 ~ experiment1 + urm_id, data = exam2_regression_urm)
get_regression_table(exam2_urm_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  78.1      0.353     221.        0     77.4    78.8 
## 2 experiment1EXPERIMENTAL    -9.49     0.462     -20.5       0    -10.4    -8.58
## 3 urm_idURM                  -9.77     0.682     -14.3       0    -11.1    -8.43
```

```r
finalexam_regression_urm <- data_2s_leveled %>%
  select(experiment1, finalexam, urm_id)

#now lets get the regression table for exam 1 control for sex
finalexam_urm_model <- lm(finalexam ~ experiment1 + urm_id, data = finalexam_regression_urm)
get_regression_table(finalexam_urm_model)
```

```
## # A tibble: 3 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 109.       0.435    251.         0   109.    110.   
## 2 experiment1EXPERIMENTAL    -2.07     0.571     -3.62       0    -3.19   -0.951
## 3 urm_idURM                 -12.5      0.842    -14.8        0   -14.1   -10.8
```
# Next, I need to do multiple regression tables for two numerical values 

```r
#math SAT scores and exam 1 score 
exam1_regression_satm <- data_2s_leveled %>% 
  select(experiment1, exam1, satmath)
ggplot(exam1_regression_satm, aes(x = satmath, y = exam1, color = experiment1)) +
  geom_point() +
  labs(x = "SAT Math Score", y = "Exam 1 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
#now lets do it for exam 2 and sat math score
exam2_regression_satm <- data_2s_leveled %>% 
  select(experiment1, exam2, satmath)
ggplot(exam2_regression_satm, aes(x = satmath, y = exam2, color = experiment1)) +
  geom_point() +
  labs(x = "SAT Math Score", y = "Exam 2 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-10-2.png)<!-- -->

```r
#final exam and sat math score
finalexam_regression_satm <- data_2s_leveled %>% 
  select(experiment1, finalexam, satmath)
ggplot(finalexam_regression_satm, aes(x = satmath, y = finalexam, color = experiment1)) +
  geom_point() +
  labs(x = "SAT Math Score", y = "Final Exam score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-10-3.png)<!-- -->
# I need to do multiple regression graphs for two sat verbal 

```r
#SAT verbal scores and exam 1 score 
exam1_regression_satv <- data_2s_leveled %>% 
  select(experiment1, exam1, satverbal)
ggplot(exam1_regression_satv, aes(x = satverbal, y = exam1, color = experiment1)) +
  geom_point() +
  labs(x = "SAT Verbal Score", y = "Exam 1 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```r
#SAT verbal scores and exam 2 score 
exam2_regression_satv <- data_2s_leveled %>% 
  select(experiment1, exam2, satverbal)
ggplot(exam2_regression_satv, aes(x = satverbal, y = exam2, color = experiment1)) +
  geom_point() +
  labs(x = "SAT Verbal Score", y = "Exam 2 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-11-2.png)<!-- -->

```r
#SAT verbal scores and exam 1 score 
finalexam_regression_satv <- data_2s_leveled %>% 
  select(experiment1, finalexam, satverbal)
ggplot(finalexam_regression_satv, aes(x = satverbal, y = finalexam, color = experiment1)) +
  geom_point() +
  labs(x = "SAT Verbal Score", y = "Final Exam score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-11-3.png)<!-- -->
# Aleks initial knowledge check score and exam scores 

```r
#Aleks initial knowledge check score and exam 1 score 
exam1_regression_satv <- data_2s_leveled %>% 
  select(experiment1, exam1, mastered_topics_initial_kc)
ggplot(exam1_regression_satv, aes(x = mastered_topics_initial_kc, y = exam1, color = experiment1)) +
  geom_point() +
  labs(x = "Aleks Inital Knowledge Check", y = "Exam 1 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

```r
#Aleks initial knowledge check score and exam 2 score 
exam2_regression_satv <- data_2s_leveled %>% 
  select(experiment1, exam2, mastered_topics_initial_kc)
ggplot(exam2_regression_satv, aes(x = mastered_topics_initial_kc, y = exam2, color = experiment1)) +
  geom_point() +
  labs(x = "Aleks Initial Knowledge Check", y = "Exam 2 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-12-2.png)<!-- -->

```r
#Aleks initial knowledge check score and final exam score 
finalexam_regression_satv <- data_2s_leveled %>% 
  select(experiment1, finalexam, mastered_topics_initial_kc)
ggplot(finalexam_regression_satv, aes(x = mastered_topics_initial_kc, y = finalexam, color = experiment1)) +
  geom_point() +
  labs(x = "Aleks Initial Knowledge Check", y = "Final Exam score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-12-3.png)<!-- -->
# High school GPA and exam score 

```r
#High school GPA and exam 1 score 
exam1_regression_high_school_gpa <- data_2s_leveled %>% 
  select(experiment1, exam1, high_sch_gpa)
ggplot(exam1_regression_high_school_gpa, aes(x = high_sch_gpa, y = exam1, color = experiment1)) +
  geom_point() +
  labs(x = "High School GPA", y = "Exam 1 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
#Aleks initial knowledge check score and exam 2 score 
exam2_regression_high_school_gpa <- data_2s_leveled %>% 
  select(experiment1, exam2, high_sch_gpa)
ggplot(exam2_regression_high_school_gpa, aes(x = high_sch_gpa, y = exam2, color = experiment1)) +
  geom_point() +
  labs(x = "High School GPA", y = "Exam 2 score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-13-2.png)<!-- -->

```r
#High school GPA and final exam score 
finalexam_regression_high_school_gpa <- data_2s_leveled %>% 
  select(experiment1, finalexam, high_sch_gpa)
ggplot(finalexam_regression_high_school_gpa, aes(x = high_sch_gpa, y = finalexam, color = experiment1)) +
  geom_point() +
  labs(x = "High School GPA", y = "Final Exam score", color = "Experiment") +
  geom_parallel_slopes(se = FALSE)
```

![](regression_graphs_and_tables_centered_files/figure-html/unnamed-chunk-13-3.png)<!-- -->















