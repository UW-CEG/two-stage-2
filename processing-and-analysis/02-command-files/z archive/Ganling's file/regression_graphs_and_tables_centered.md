---
title: "Regression_graphs_and_tables_centered"
author: "Ganling"
date: "5/10/2021"
output: 
  html_document:
    keep_md: yes
---
### Load the libraries

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
### Load the data using the directory 

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
### Copy data from `original-data` to `01-importable-data`

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
### read in the data file 

```r
data_2s <- readRDS(paste0(importable_data_dir, "master_2s_small_deid_scaled.rds"))
#now lets do some data wrangling to make sure everything runs
data_2s_extras_removed = subset(data_2s, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)
data_2s_exam_score <- data_2s_unique %>% 
  select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc, experiment1, satm_c, satv_c, aleksikc_c, hsgpa_c, satm_z, satv_z, aleksikc_z, hsgpa_z, final_z, final_c)
  
data_2s_exam_score_true <- na.omit(data_2s_exam_score)
#making sure the data control and experiment groups are leveled for further regression analysis
data_2s_leveled <- data_2s_exam_score_true %>% 
    mutate(experiment1 = factor(experiment1, 
        levels = c("CONTROL", "EXPERIMENTAL")))
```
### Now, I want to use regression model to see the exam score difference between experiement year and the non experiment year

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
### now I would like to see the interaction between the scores when we control for sex

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
### control for eop status 

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
### Now we look for fgn status

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
### URM status and score interaction

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
### Next, I need to do multiple regression tables for two numerical values 

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
### I need to do multiple regression graphs for two sat verbal 

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
### Aleks initial knowledge check score and exam scores 

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
### High school GPA and exam score 

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


## Final Exam Score C Regression Tables
### 1. final_c and Experiment

```r
final_c_regression <- data_2s_leveled %>%
  select(experiment1, final_c)
#now lets find mean and median scores of final exam C
final_c_regression_stats <- final_c_regression %>%
  group_by(experiment1) %>%
  summarize(exam1_median = median(final_c), 
            exam1_mean = mean(final_c))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table for final exam 
exam1_model <- lm(final_c ~ experiment1, data = final_c_regression)
get_regression_table(exam1_model)
```

```
## # A tibble: 2 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 -0.079     0.429    -0.183   0.855    -0.92    0.763
## 2 experiment1EXPERIMENTAL   -1.00      0.579    -1.73    0.084    -2.14    0.133
```
### 2. final score controlled for satm, satv, alekskc, and high school gpa

```r
final_c_regression_2 <- data_2s_leveled %>%
  select(experiment1, final_c, satm_c, satv_c, aleksikc_c, hsgpa_c, sex_id, eop_id, urm_id, fgn_id)

#now lets get the regression table for exam 1 
final_c_model_controlled <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c, data = final_c_regression_2)
get_regression_table(final_c_model_controlled)
```

```
## # A tibble: 6 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.571     0.328      1.74   0.082   -0.072    1.21 
## 2 experiment1EXPERIMENTAL   -0.805     0.441     -1.83   0.068   -1.67     0.059
## 3 satm_c                     0.148     0.004     36.6    0        0.14     0.156
## 4 satv_c                     0.032     0.004      7.80   0        0.024    0.04 
## 5 aleksikc_c                 0.301     0.014     21.0    0        0.273    0.329
## 6 hsgpa_c                   31.5       1.26      25.1    0       29.0     34.0
```
### 3. Controlling sex_id

```r
final_c_model_controlled_sex <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + sex_id, data = final_c_regression_2)
get_regression_table(final_c_model_controlled_sex)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  2.69      0.409      6.57   0        1.89     3.50 
## 2 experiment1EXPERIMENTAL   -0.805     0.439     -1.83   0.067   -1.66     0.055
## 3 satm_c                     0.139     0.004     33.5    0        0.131    0.147
## 4 satv_c                     0.038     0.004      9.03   0        0.029    0.046
## 5 aleksikc_c                 0.295     0.014     20.7    0        0.267    0.323
## 6 hsgpa_c                   33.1       1.26      26.2    0       30.7     35.6  
## 7 sex_idFemale              -3.94      0.46      -8.57   0       -4.84    -3.04
```
### 4. Controlling eop_id

```r
final_c_model_controlled_eop <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + eop_id, data = final_c_regression_2)
get_regression_table(final_c_model_controlled_eop)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.303     0.347     0.875   0.382   -0.376    0.982
## 2 experiment1EXPERIMENTAL   -0.875     0.442    -1.98    0.048   -1.74    -0.009
## 3 satm_c                     0.15      0.004    36.4     0        0.142    0.158
## 4 satv_c                     0.034     0.004     8.07    0        0.026    0.042
## 5 aleksikc_c                 0.3       0.014    21.0     0        0.272    0.328
## 6 hsgpa_c                   31.8       1.26     25.2     0       29.3     34.3  
## 7 eop_idEOP                  1.37      0.574     2.38    0.017    0.24     2.49
```
### 5. Controlling for fgn id

```r
final_c_model_controlled_fgn <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + fgn_id, data = final_c_regression_2)
get_regression_table(final_c_model_controlled_fgn)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  1.05      0.364      2.88   0.004    0.336    1.76 
## 2 experiment1EXPERIMENTAL   -0.835     0.441     -1.89   0.058   -1.70     0.029
## 3 satm_c                     0.146     0.004     35.8    0        0.138    0.154
## 4 satv_c                     0.029     0.004      6.73   0        0.02     0.037
## 5 aleksikc_c                 0.303     0.014     21.1    0        0.274    0.331
## 6 hsgpa_c                   31.4       1.26      25.0    0       28.9     33.8  
## 7 fgn_idFGN                 -1.64      0.546     -3.01   0.003   -2.71    -0.571
```
### 6. controlling for urm_id

```r
final_c_model_controlled_urm <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + urm_id, data = final_c_regression_2)
get_regression_table(final_c_model_controlled_urm)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.361     0.338      1.07   0.285   -0.301    1.02 
## 2 experiment1EXPERIMENTAL   -0.833     0.441     -1.89   0.059   -1.70     0.031
## 3 satm_c                     0.15      0.004     36.4    0        0.142    0.158
## 4 satv_c                     0.033     0.004      7.94   0        0.025    0.041
## 5 aleksikc_c                 0.299     0.014     20.9    0        0.271    0.327
## 6 hsgpa_c                   31.9       1.26      25.2    0       29.4     34.4  
## 7 urm_idURM                  1.76      0.688      2.56   0.01     0.416    3.12
```
## Final Exam Score Z regression tables
### 1. Final z score and experiment

```r
final_z_regression <- data_2s_leveled %>%
  select(experiment1, final_z)
#now lets find mean and median scores of final exam z
final_z_regression_stats <- final_z_regression %>%
  group_by(experiment1) %>%
  summarize(exam1_median = median(final_z), 
            exam1_mean = mean(final_z))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#now lets get the regression table for final exam 
final_z_model <- lm(final_z ~ experiment1, data = final_z_regression)
get_regression_table(final_z_model)
```

```
## # A tibble: 2 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                 -0.003     0.017    -0.184   0.854   -0.037    0.03 
## 2 experiment1EXPERIMENTAL   -0.04      0.023    -1.72    0.086   -0.085    0.006
```
### 2. final score controlled for satm, satv, alekskc, and high school gpa

```r
final_z_regression_2 <- data_2s_leveled %>%
  select(experiment1, final_z, satm_z, satv_z, aleksikc_z, hsgpa_z, sex_id, eop_id, urm_id, fgn_id)

#now lets get the regression table for exam 1 
final_z_model_controlled <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z, data = final_z_regression_2)
get_regression_table(final_z_model_controlled)
```

```
## # A tibble: 6 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.023     0.013      1.78   0.076   -0.002    0.049
## 2 experiment1EXPERIMENTAL   -0.033     0.018     -1.87   0.062   -0.067    0.002
## 3 satm_z                     0.434     0.012     36.5    0        0.411    0.458
## 4 satv_z                     0.088     0.011      7.79   0        0.066    0.11 
## 5 aleksikc_z                 0.2       0.009     21.4    0        0.182    0.219
## 6 hsgpa_z                    0.226     0.009     25.2    0        0.209    0.244
```
### 3. Controlling sex_id

```r
final_z_model_controlled_sex <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + sex_id, data = final_z_regression_2)
get_regression_table(final_z_model_controlled_sex)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.108     0.016      6.63   0        0.076    0.14 
## 2 experiment1EXPERIMENTAL   -0.033     0.017     -1.88   0.061   -0.067    0.001
## 3 satm_z                     0.408     0.012     33.4    0        0.384    0.432
## 4 satv_z                     0.103     0.011      9.02   0        0.08     0.125
## 5 aleksikc_z                 0.196     0.009     21.0    0        0.178    0.215
## 6 hsgpa_z                    0.238     0.009     26.3    0        0.22     0.256
## 7 sex_idFemale              -0.158     0.018     -8.62   0       -0.193   -0.122
```
### 4. Controlling eop_id

```r
final_z_model_controlled_eop <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + eop_id, data = final_z_regression_2)
get_regression_table(final_z_model_controlled_eop)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.013     0.014     0.923   0.356   -0.014    0.04 
## 2 experiment1EXPERIMENTAL   -0.035     0.018    -2.02    0.043   -0.07    -0.001
## 3 satm_z                     0.44      0.012    36.2     0        0.416    0.464
## 4 satv_z                     0.092     0.011     8.06    0        0.07     0.115
## 5 aleksikc_z                 0.199     0.009    21.3     0        0.181    0.218
## 6 hsgpa_z                    0.228     0.009    25.3     0        0.211    0.246
## 7 eop_idEOP                  0.053     0.023     2.33    0.02     0.008    0.098
```
### 5. Controlling fgn_id

```r
final_z_model_controlled_fgn <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + fgn_id, data = final_z_regression_2)
get_regression_table(final_z_model_controlled_fgn)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.042     0.014      2.93   0.003    0.014    0.071
## 2 experiment1EXPERIMENTAL   -0.034     0.018     -1.94   0.053   -0.068    0    
## 3 satm_z                     0.429     0.012     35.7    0        0.406    0.453
## 4 satv_z                     0.079     0.012      6.71   0        0.056    0.102
## 5 aleksikc_z                 0.201     0.009     21.5    0        0.183    0.219
## 6 hsgpa_z                    0.225     0.009     25.1    0        0.208    0.243
## 7 fgn_idFGN                 -0.066     0.022     -3.04   0.002   -0.109   -0.023
```
### 6. Controlling urm_id

```r
final_z_model_controlled_urm <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + urm_id, data = final_z_regression_2)
get_regression_table(final_z_model_controlled_sex)
```

```
## # A tibble: 7 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.108     0.016      6.63   0        0.076    0.14 
## 2 experiment1EXPERIMENTAL   -0.033     0.017     -1.88   0.061   -0.067    0.001
## 3 satm_z                     0.408     0.012     33.4    0        0.384    0.432
## 4 satv_z                     0.103     0.011      9.02   0        0.08     0.125
## 5 aleksikc_z                 0.196     0.009     21.0    0        0.178    0.215
## 6 hsgpa_z                    0.238     0.009     26.3    0        0.22     0.256
## 7 sex_idFemale              -0.158     0.018     -8.62   0       -0.193   -0.122
```
## Final Exam Score C interaction Tables
### 1. Final c score and sex id interaction

```r
final_c_regression_sex <- data_2s_leveled %>%
  select(experiment1, final_c, satm_c, satv_c, aleksikc_c, hsgpa_c, sex_id, eop_id, urm_id, fgn_id)

#now lets get the regression table for exam 1 
final_c_model_sex <- lm(final_c ~ experiment1*sex_id, data = final_c_regression_2)
get_regression_table(final_c_model_sex)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  3.46      0.637     5.44    0         2.21    4.71 
## 2 experiment1EXPERIMENTAL   -0.227     0.86     -0.264   0.792    -1.91    1.46 
## 3 sex_idFemale              -6.38      0.855    -7.47    0        -8.06   -4.71 
## 4 experiment1EXPERIMENTA~   -1.37      1.15     -1.18    0.236    -3.63    0.895
```
### 2. Final c score and sex id interaction controlling other variables

```r
final_c_regression_sex <- data_2s_leveled %>%
  select(experiment1, final_c, satm_c, satv_c, aleksikc_c, hsgpa_c, sex_id, eop_id, urm_id, fgn_id)

#now lets get the regression table for exam 1 
final_c_model_sex <- lm(final_c ~ experiment1*sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = final_c_regression_2)
get_regression_table(final_c_model_sex)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  3.16      0.49       6.44   0        2.20     4.12 
## 2 experiment1EXPERIMENTAL   -1.65      0.658     -2.51   0.012   -2.94    -0.362
## 3 sex_idFemale              -4.78      0.669     -7.14   0       -6.09    -3.47 
## 4 satm_c                     0.139     0.004     33.5    0        0.131    0.147
## 5 satv_c                     0.038     0.004      9.05   0        0.03     0.046
## 6 aleksikc_c                 0.295     0.014     20.7    0        0.267    0.323
## 7 hsgpa_c                   33.2       1.27      26.2    0       30.7     35.7  
## 8 experiment1EXPERIMENTA~    1.53      0.884      1.73   0.084   -0.207    3.26
```
### 3. Final c score and eop id interaction

```r
final_c_regression_eop <- data_2s_leveled %>%
  select(experiment1, final_c, satm_c, satv_c, aleksikc_c, hsgpa_c, sex_id, eop_id, urm_id, fgn_id)

final_c_model_sex <- lm(final_c ~ experiment1*eop_id, data = final_c_regression_2)
get_regression_table(final_c_model_sex)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  2.64      0.468     5.63    0         1.72     3.55
## 2 experiment1EXPERIMENTAL   -0.588     0.641    -0.916   0.36     -1.84     0.67
## 3 eop_idEOP                -13.6       1.05    -13.0     0       -15.7    -11.6 
## 4 experiment1EXPERIMENTA~    1.11      1.37      0.811   0.418    -1.57     3.79
```
### 4. Final c score and eop id interaction controlling other variables

```r
final_c_regression_eop <- data_2s_leveled %>%
  select(experiment1, final_c, satm_c, satv_c, aleksikc_c, hsgpa_c, sex_id, eop_id, urm_id, fgn_id)

final_c_model_sex <- lm(final_c ~ experiment1*eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = final_c_regression_2)
get_regression_table(final_c_model_sex)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.291     0.369     0.788   0.431   -0.432    1.01 
## 2 experiment1EXPERIMENTAL   -0.852     0.5      -1.70    0.089   -1.83     0.129
## 3 eop_idEOP                  1.43      0.857     1.67    0.095   -0.251    3.11 
## 4 satm_c                     0.15      0.004    36.3     0        0.142    0.158
## 5 satv_c                     0.034     0.004     8.07    0        0.026    0.042
## 6 aleksikc_c                 0.3       0.014    21.0     0        0.272    0.328
## 7 hsgpa_c                   31.8       1.26     25.2     0       29.3     34.3  
## 8 experiment1EXPERIMENTA~   -0.105     1.07     -0.099   0.921   -2.20     1.99
```
### 5.Final C score interacting with fgn status

```r
final_c_model_fgn <- lm(final_c ~ experiment1*fgn_id, data = final_c_regression_2)
get_regression_table(final_c_model_sex)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.291     0.369     0.788   0.431   -0.432    1.01 
## 2 experiment1EXPERIMENTAL   -0.852     0.5      -1.70    0.089   -1.83     0.129
## 3 eop_idEOP                  1.43      0.857     1.67    0.095   -0.251    3.11 
## 4 satm_c                     0.15      0.004    36.3     0        0.142    0.158
## 5 satv_c                     0.034     0.004     8.07    0        0.026    0.042
## 6 aleksikc_c                 0.3       0.014    21.0     0        0.272    0.328
## 7 hsgpa_c                   31.8       1.26     25.2     0       29.3     34.3  
## 8 experiment1EXPERIMENTA~   -0.105     1.07     -0.099   0.921   -2.20     1.99
```
### 6. Final c score and fgn id interaction controlling other variables

```r
final_c_model_fgn <- lm(final_c ~ experiment1*fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = final_c_regression_2)
get_regression_table(final_c_model_fgn)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.477     0.394     1.21    0.226   -0.296    1.25 
## 2 experiment1EXPERIMENTAL    0.214     0.52      0.411   0.681   -0.806    1.23 
## 3 fgn_idFGN                  0.327     0.753     0.434   0.664   -1.15     1.80 
## 4 satm_c                     0.146     0.004    35.8     0        0.138    0.154
## 5 satv_c                     0.029     0.004     6.68    0        0.02     0.037
## 6 aleksikc_c                 0.303     0.014    21.2     0        0.274    0.331
## 7 hsgpa_c                   31.2       1.26     24.8     0       28.7     33.6  
## 8 experiment1EXPERIMENTA~   -3.72      0.98     -3.79    0       -5.64    -1.80
```
### 7.Final C score interacting with urm status

```r
final_c_model_urm <- lm(final_c ~ experiment1*fgn_id, data = final_c_regression_2)
get_regression_table(final_c_model_urm)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  3.11      0.492      6.33   0        2.15      4.07
## 2 experiment1EXPERIMENTAL    0.527     0.659      0.8    0.424   -0.765     1.82
## 3 fgn_idFGN                -10.9       0.91     -12.0    0      -12.7      -9.15
## 4 experiment1EXPERIMENTA~   -6.42      1.24      -5.18   0       -8.86     -3.99
```
### 8. Final c score and fgn id interaction controlling other variables

```r
final_c_model_urm <- lm(final_c ~ experiment1*urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = final_c_regression_2)
get_regression_table(final_c_model_urm)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.413     0.35      1.18    0.238   -0.273    1.10 
## 2 experiment1EXPERIMENTAL   -0.929     0.473    -1.96    0.049   -1.86    -0.002
## 3 urm_idURM                  1.34      1.02      1.31    0.189   -0.66     3.34 
## 4 satm_c                     0.15      0.004    36.4     0        0.142    0.158
## 5 satv_c                     0.033     0.004     7.94    0        0.025    0.041
## 6 aleksikc_c                 0.299     0.014    20.9     0        0.271    0.327
## 7 hsgpa_c                   31.9       1.27     25.2     0       29.4     34.4  
## 8 experiment1EXPERIMENTA~    0.741     1.32      0.562   0.574   -1.84     3.32
```
## Final Exam Score z interaction Tables
### 1. Final z score and sex id interaction

```r
final_z_regression_sex <- data_2s_leveled %>%
  select(experiment1, final_z, satm_z, satv_z, aleksikc_z, hsgpa_z, sex_id, eop_id, urm_id, fgn_id)

#now lets get the regression table for final exam 
final_z_model_sex <- lm(final_z ~ experiment1*sex_id, data = final_z_regression_2)
get_regression_table(final_z_model_sex)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.139     0.025     5.47    0        0.089    0.188
## 2 experiment1EXPERIMENTAL   -0.011     0.034    -0.311   0.756   -0.078    0.056
## 3 sex_idFemale              -0.256     0.034    -7.52    0       -0.323   -0.189
## 4 experiment1EXPERIMENTA~   -0.051     0.046    -1.11    0.266   -0.141    0.039
```
### 2. Final z score and sex id interaction controlling other variables

```r
final_z_model_sex <- lm(final_z ~ experiment1*sex_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = final_z_regression_2)
get_regression_table(final_z_model_sex)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.127     0.019      6.49   0        0.088    0.165
## 2 experiment1EXPERIMENTAL   -0.067     0.026     -2.54   0.011   -0.118   -0.015
## 3 sex_idFemale              -0.191     0.027     -7.18   0       -0.243   -0.139
## 4 satm_z                     0.408     0.012     33.4    0        0.385    0.432
## 5 satv_z                     0.103     0.011      9.05   0        0.081    0.125
## 6 aleksikc_z                 0.196     0.009     21.0    0        0.178    0.215
## 7 hsgpa_z                    0.239     0.009     26.4    0        0.221    0.256
## 8 experiment1EXPERIMENTA~    0.061     0.035      1.74   0.083   -0.008    0.13
```
### 3. Final z score and eop id interaction

```r
final_z_model_sex <- lm(final_z ~ experiment1*eop_id, data = final_z_regression_2)
get_regression_table(final_z_model_sex)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.106     0.019     5.67    0        0.069    0.142
## 2 experiment1EXPERIMENTAL   -0.025     0.026    -0.961   0.336   -0.075    0.026
## 3 eop_idEOP                 -0.546     0.042   -13.1     0       -0.628   -0.464
## 4 experiment1EXPERIMENTA~    0.05      0.054     0.927   0.354   -0.056    0.157
```
### 4. Final z score and eop id interaction controlling other variables

```r
final_z_model_sex <- lm(final_z ~ experiment1*eop_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = final_z_regression_2)
get_regression_table(final_z_model_sex)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.012     0.015     0.799   0.424   -0.017    0.04 
## 2 experiment1EXPERIMENTAL   -0.034     0.02     -1.69    0.091   -0.073    0.005
## 3 eop_idEOP                  0.058     0.034     1.71    0.088   -0.009    0.125
## 4 satm_z                     0.44      0.012    36.2     0        0.416    0.464
## 5 satv_z                     0.092     0.011     8.06    0        0.07     0.115
## 6 aleksikc_z                 0.199     0.009    21.3     0        0.181    0.218
## 7 hsgpa_z                    0.228     0.009    25.3     0        0.211    0.246
## 8 experiment1EXPERIMENTA~   -0.008     0.042    -0.198   0.843   -0.092    0.075
```
### 5.Final z score interacting with fgn status

```r
final_z_model_fgn <- lm(final_z ~ experiment1*fgn_id, data = final_z_regression_2)
get_regression_table(final_z_model_sex)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.012     0.015     0.799   0.424   -0.017    0.04 
## 2 experiment1EXPERIMENTAL   -0.034     0.02     -1.69    0.091   -0.073    0.005
## 3 eop_idEOP                  0.058     0.034     1.71    0.088   -0.009    0.125
## 4 satm_z                     0.44      0.012    36.2     0        0.416    0.464
## 5 satv_z                     0.092     0.011     8.06    0        0.07     0.115
## 6 aleksikc_z                 0.199     0.009    21.3     0        0.181    0.218
## 7 hsgpa_z                    0.228     0.009    25.3     0        0.211    0.246
## 8 experiment1EXPERIMENTA~   -0.008     0.042    -0.198   0.843   -0.092    0.075
```
### 6. Final z score and fgn id interaction controlling other variables

```r
final_z_model_fgn <- lm(final_z ~ experiment1*fgn_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = final_z_regression_2)
get_regression_table(final_z_model_fgn)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.019     0.016     1.22    0.221   -0.012    0.05 
## 2 experiment1EXPERIMENTAL    0.009     0.021     0.412   0.68    -0.032    0.049
## 3 fgn_idFGN                  0.014     0.03      0.463   0.643   -0.045    0.073
## 4 satm_z                     0.429     0.012    35.7     0        0.405    0.452
## 5 satv_z                     0.078     0.012     6.65    0        0.055    0.101
## 6 aleksikc_z                 0.201     0.009    21.5     0        0.183    0.219
## 7 hsgpa_z                    0.224     0.009    24.9     0        0.206    0.242
## 8 experiment1EXPERIMENTA~   -0.151     0.039    -3.87    0       -0.227   -0.074
```
### 7.Final z score interacting with urm status

```r
final_z_model_urm <- lm(final_z ~ experiment1*fgn_id, data = final_z_regression_2)
get_regression_table(final_z_model_urm)
```

```
## # A tibble: 4 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.125     0.02      6.37    0        0.086    0.163
## 2 experiment1EXPERIMENTAL    0.019     0.026     0.738   0.461   -0.032    0.071
## 3 fgn_idFGN                 -0.438     0.036   -12.1     0       -0.509   -0.367
## 4 experiment1EXPERIMENTA~   -0.249     0.049    -5.04    0       -0.346   -0.152
```
### 8. Final z score and fgn id interaction controlling other variables

```r
final_z_model_urm <- lm(final_z ~ experiment1*urm_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = final_z_regression_2)
get_regression_table(final_z_model_urm)
```

```
## # A tibble: 8 x 7
##   term                    estimate std_error statistic p_value lower_ci upper_ci
##   <chr>                      <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
## 1 intercept                  0.017     0.014     1.20    0.229   -0.011    0.044
## 2 experiment1EXPERIMENTAL   -0.037     0.019    -1.98    0.048   -0.074    0    
## 3 urm_idURM                  0.054     0.041     1.33    0.182   -0.025    0.134
## 4 satm_z                     0.44      0.012    36.3     0        0.417    0.464
## 5 satv_z                     0.09      0.011     7.93    0        0.068    0.112
## 6 aleksikc_z                 0.199     0.009    21.2     0        0.18     0.217
## 7 hsgpa_z                    0.229     0.009    25.3     0        0.211    0.247
## 8 experiment1EXPERIMENTA~    0.026     0.052     0.491   0.623   -0.077    0.128
```









