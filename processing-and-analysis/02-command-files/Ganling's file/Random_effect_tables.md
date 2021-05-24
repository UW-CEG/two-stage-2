---
title: "TA Section Random Effects"
author: "Ganling"
date: "5/9/2021"
output: 
  html_document:
    keep_md: yes
---

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
library(lme4)
```

```
## Loading required package: Matrix
```

```
## 
## Attaching package: 'Matrix'
```

```
## The following objects are masked from 'package:tidyr':
## 
##     expand, pack, unpack
```
### Set working directories

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
### load the data set through here directory

```r
# CFC: Copy rds file from the `original_data_dir` to the `importable_data_dir`.
copy_from <- paste0(original_data_dir, "master_2s_small_deid_scaled")
copy_to <- paste0(importable_data_dir, "master_2s_small_deid_scaled")
# CFC: If the file already exists in the target directory, the file will not be 
# overwritten and this command will return `FALSE`.
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```

```r
#data_2s <- readRDS(paste0(metadata_dir, "master_2s_small_deid_scaled.rds"))
data_2s <- readRDS("G:/Shared drives/CEG Two-Stage Exams Analysis/Ganling/two-stage-2/original-data/master_2s_small_deid_scaled.rds")

data_2s_extras_removed = subset(data_2s, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)
View(data_2s_unique)
data_2s_exam_score <- data_2s_unique %>% 
  select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, experiment1, ta_sect, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc)
  
data_2s_exam_score_true <- na.omit(data_2s_exam_score)
```
###I would like to create a model that could implement the random effects based on the TA sections

```r
#First, include sex_id as a control variable
mod1 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + sex_id + (1|ta_sect), 
           data=data_2s_exam_score_true)
summary(mod1)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + sex_id + (1 | ta_sect)
##    Data: data_2s_exam_score_true
## 
## REML criterion at convergence: 63526.1
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4209 -0.6213  0.0889  0.7073  2.7950 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.13    3.336  
##  Residual             338.69   18.403  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.434e+02  4.926e+00 -29.115
## experiment1EXPERIMENTAL    -1.070e+00  4.401e-01  -2.431
## satmath                     1.376e-01  4.142e-03  33.214
## satverbal                   3.905e-02  4.133e-03   9.448
## mastered_topics_initial_kc  2.961e-01  1.428e-02  20.734
## high_sch_gpa                3.265e+01  1.260e+00  25.907
## sex_idFemale               -3.917e+00  4.595e-01  -8.526
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.072                                   
## satmath     -0.135  0.031                            
## satverbal   -0.114  0.007 -0.615                     
## mstrd_tpc__  0.023  0.001 -0.172 -0.020              
## high_sch_gp -0.871  0.001 -0.093 -0.081 -0.015       
## sex_idFemal  0.029 -0.001  0.247 -0.148  0.048 -0.147
```

```r
AIC(mod1)
```

```
## [1] 63544.08
```

```r
#Include eop_id as a control variable
mod2 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + eop_id + (1|ta_sect), 
           data=data_2s_exam_score_true)
summary(mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + eop_id + (1 | ta_sect)
##    Data: data_2s_exam_score_true
## 
## REML criterion at convergence: 63592.7
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.2840 -0.6169  0.1022  0.7021  2.8831 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.2     3.347  
##  Residual             341.8    18.488  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.457e+02  5.176e+00 -28.150
## experiment1EXPERIMENTAL    -1.127e+00  4.428e-01  -2.546
## satmath                     1.482e-01  4.115e-03  36.016
## satverbal                   3.526e-02  4.153e-03   8.490
## mastered_topics_initial_kc  3.007e-01  1.434e-02  20.963
## high_sch_gpa                3.135e+01  1.258e+00  24.917
## eop_idEOP                   1.328e+00  5.761e-01   2.305
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.053                                   
## satmath     -0.196  0.021                            
## satverbal   -0.149 -0.001 -0.554                     
## mstrd_tpc__  0.032  0.003 -0.194 -0.019              
## high_sch_gp -0.862 -0.005 -0.038 -0.089 -0.012       
## eop_idEOP   -0.294 -0.053  0.199  0.150 -0.039  0.097
```

```r
AIC(mod2)
```

```
## [1] 63610.66
```

```r
#Include fgn_id as a control variable
mod3 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + fgn_id + (1|ta_sect), 
           data=data_2s_exam_score_true)
summary(mod3)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + fgn_id + (1 | ta_sect)
##    Data: data_2s_exam_score_true
## 
## REML criterion at convergence: 63588.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.2991 -0.6294  0.0983  0.6846  2.8815 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.15    3.34   
##  Residual             341.62   18.48   
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.378e+02  5.147e+00 -26.771
## experiment1EXPERIMENTAL    -1.132e+00  4.424e-01  -2.560
## satmath                     1.445e-01  4.074e-03  35.464
## satverbal                   3.032e-02  4.258e-03   7.122
## mastered_topics_initial_kc  3.032e-01  1.433e-02  21.153
## high_sch_gpa                3.096e+01  1.252e+00  24.719
## fgn_idFGN                  -1.685e+00  5.441e-01  -3.096
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.082                                   
## satmath     -0.180  0.038                            
## satverbal   -0.176  0.018 -0.537                     
## mstrd_tpc__  0.029 -0.001 -0.192 -0.020              
## high_sch_gp -0.850  0.002 -0.054 -0.094 -0.009       
## fgn_idFGN   -0.277  0.044  0.145  0.266 -0.028  0.029
```

```r
AIC(mod3)
```

```
## [1] 63606.51
```

```r
#Include urm_id as a control variable
mod4 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + urm_id + (1|ta_sect), 
           data=data_2s_exam_score_true)
summary(mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + urm_id + (1 | ta_sect)
##    Data: data_2s_exam_score_true
## 
## REML criterion at convergence: 63591.4
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.2817 -0.6168  0.1036  0.7015  2.8753 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.21    3.349  
##  Residual             341.76   18.487  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.455e+02  5.121e+00 -28.416
## experiment1EXPERIMENTAL    -1.089e+00  4.421e-01  -2.462
## satmath                     1.483e-01  4.112e-03  36.067
## satverbal                   3.440e-02  4.112e-03   8.366
## mastered_topics_initial_kc  2.998e-01  1.436e-02  20.882
## high_sch_gpa                3.144e+01  1.261e+00  24.934
## urm_idURM                   1.722e+00  6.897e-01   2.497
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.066                                   
## satmath     -0.190  0.029                            
## satverbal   -0.121  0.006 -0.579                     
## mstrd_tpc__  0.037  0.001 -0.198 -0.017              
## high_sch_gp -0.871 -0.001 -0.034 -0.098 -0.015       
## urm_idURM   -0.259 -0.014  0.197  0.057 -0.060  0.118
```

```r
AIC(mod4)
```

```
## [1] 63609.38
```

```r
#
```






