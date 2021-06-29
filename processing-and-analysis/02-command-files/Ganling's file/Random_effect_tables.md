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
###Import and read the data

```r
#data_2s <- readRDS(paste0(metadata_dir, "master_2s_small_deid_scaled.rds"))
data_2s <- readRDS("G:/Shared drives/CEG Two-Stage Exams Analysis/Ganling/two-stage-2/original-data/master_2s_small_deid_scaled.rds")
```
##Data wrangling
####Creating unique rows and remove the ones that are not necessary for this data model

```r
data_2s_extras_removed = subset(data_2s, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)
View(data_2s_unique)
data_2s_exam_score <- data_2s_unique %>% 
  select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, experiment1, ta_sect, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc)
  
data_2s_exam_score_true <- na.omit(data_2s_exam_score)
```
##Make 'ta_sect'unique across different data values
####First, we need to split the data from into year 16 and year 17 

```r
data_2s_exam_score_true$ta_sect <- as.character(data_2s_exam_score_true$ta_sect)

#next, split the dataframe in half
master_unique_2016 <- subset(data_2s_exam_score_true, course_fullid != "CHEM_142_A_2017_4" & course_fullid != "CHEM_142_B_2017_4")
master_unique_2017 <- subset(data_2s_exam_score_true, course_fullid != "CHEM_142_A_2016_4" & course_fullid != "CHEM_142_B_2016_4")
```
####Use the vectorization function in R, we can add '_16' and '_17'at the end of each TA section 

```r
master_unique_2016$ta_sect <- paste0(master_unique_2016$ta_sect, "_16")
master_unique_2017$ta_sect <- paste0(master_unique_2017$ta_sect, "_17")
```
##recombine the dataframes together and make the 'ta_sect' factors again

```r
master_true <- rbind(master_unique_2016, master_unique_2017)
master_true$ta_sect <- as.factor(master_true$ta_sect)
```
####I would like to create a model that could implement the random effects based on the TA sections

```r
#First, include sex_id as a control variable
mod1 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + sex_id + (1|ta_sect), 
           data=master_true)
summary(mod1)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + sex_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63478.9
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5638 -0.6262  0.0977  0.7010  2.8624 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  17.33    4.163  
##  Residual             333.49   18.262  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.433e+02  4.940e+00 -29.006
## experiment1EXPERIMENTAL    -9.145e-01  9.319e-01  -0.981
## satmath                     1.369e-01  4.159e-03  32.920
## satverbal                   3.686e-02  4.147e-03   8.887
## mastered_topics_initial_kc  2.986e-01  1.434e-02  20.820
## high_sch_gpa                3.301e+01  1.264e+00  26.112
## sex_idFemale               -3.762e+00  4.611e-01  -8.159
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.108                                   
## satmath     -0.133  0.010                            
## satverbal   -0.110  0.005 -0.615                     
## mstrd_tpc__  0.025  0.003 -0.170 -0.021              
## high_sch_gp -0.868  0.001 -0.094 -0.085 -0.019       
## sex_idFemal  0.026  0.002  0.245 -0.146  0.045 -0.144
```

```r
AIC(mod1)
```

```
## [1] 63496.9
```

```r
#Include eop_id as a control variable
mod2 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + eop_id + (1|ta_sect), 
           data=master_true)
summary(mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + eop_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63539.7
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4240 -0.6272  0.0978  0.7032  2.9610 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  18.03    4.246  
##  Residual             336.18   18.335  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.457e+02  5.191e+00 -28.064
## experiment1EXPERIMENTAL    -9.507e-01  9.473e-01  -1.004
## satmath                     1.471e-01  4.132e-03  35.595
## satverbal                   3.326e-02  4.164e-03   7.987
## mastered_topics_initial_kc  3.027e-01  1.440e-02  21.027
## high_sch_gpa                3.181e+01  1.262e+00  25.196
## eop_idEOP                   1.299e+00  5.811e-01   2.236
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.097                                   
## satmath     -0.193  0.004                            
## satverbal   -0.145  0.002 -0.556                     
## mstrd_tpc__  0.034  0.004 -0.191 -0.020              
## high_sch_gp -0.860 -0.002 -0.040 -0.092 -0.016       
## eop_idEOP   -0.295 -0.025  0.200  0.146 -0.037  0.099
```

```r
AIC(mod2)
```

```
## [1] 63557.7
```

```r
#Include fgn_id as a control variable
mod3 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + fgn_id + (1|ta_sect), 
           data=master_true)
summary(mod3)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + fgn_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63537.6
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4363 -0.6350  0.1009  0.7023  2.9650 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  17.83    4.222  
##  Residual             336.12   18.334  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.384e+02  5.164e+00 -26.800
## experiment1EXPERIMENTAL    -9.531e-01  9.431e-01  -1.011
## satmath                     1.436e-01  4.093e-03  35.097
## satverbal                   2.889e-02  4.269e-03   6.767
## mastered_topics_initial_kc  3.049e-01  1.439e-02  21.193
## high_sch_gpa                3.142e+01  1.257e+00  25.005
## fgn_idFGN                  -1.468e+00  5.453e-01  -2.692
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.110                                   
## satmath     -0.177  0.012                            
## satverbal   -0.173  0.011 -0.538                     
## mstrd_tpc__  0.031  0.003 -0.189 -0.021              
## high_sch_gp -0.847  0.002 -0.056 -0.096 -0.013       
## fgn_idFGN   -0.279  0.021  0.146  0.263 -0.028  0.031
```

```r
AIC(mod3)
```

```
## [1] 63555.58
```

```r
#Include urm_id as a control variable
mod4 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + urm_id + (1|ta_sect), 
           data=master_true)
summary(mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + urm_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63538.7
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4178 -0.6303  0.1014  0.7110  2.9500 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  18.05    4.249  
##  Residual             336.15   18.334  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.454e+02  5.135e+00 -28.320
## experiment1EXPERIMENTAL    -9.122e-01  9.475e-01  -0.963
## satmath                     1.471e-01  4.125e-03  35.663
## satverbal                   3.251e-02  4.128e-03   7.877
## mastered_topics_initial_kc  3.020e-01  1.441e-02  20.960
## high_sch_gpa                3.188e+01  1.265e+00  25.205
## urm_idURM                   1.662e+00  6.977e-01   2.382
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___ hgh_s_
## e1EXPERIMEN -0.103                                   
## satmath     -0.186  0.008                            
## satverbal   -0.120  0.005 -0.579                     
## mstrd_tpc__  0.037  0.004 -0.194 -0.018              
## high_sch_gp -0.868  0.000 -0.037 -0.100 -0.019       
## urm_idURM   -0.259 -0.006  0.192  0.062 -0.055  0.118
```

```r
AIC(mod4)
```

```
## [1] 63556.66
```

```r
#
```
####This is what I got previously by changing out different independent variables, now I wnat to compare only the models with and without random effects

```r
#First, test that the treatment impacts scores
mod1 <- lm(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa, 
           data=master_true)
summary(mod1)
```

```
## 
## Call:
## lm(formula = finalexam ~ experiment1 + satmath + satverbal + 
##     mastered_topics_initial_kc + high_sch_gpa, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.732 -12.006   2.003  12.879  54.983 
## 
## Coefficients:
##                              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                -1.440e+02  4.937e+00 -29.176  < 2e-16 ***
## experiment1EXPERIMENTAL    -9.347e-01  4.418e-01  -2.116   0.0344 *  
## satmath                     1.478e-01  4.053e-03  36.463  < 2e-16 ***
## satverbal                   3.253e-02  4.142e-03   7.853 4.64e-15 ***
## mastered_topics_initial_kc  3.007e-01  1.434e-02  20.967  < 2e-16 ***
## high_sch_gpa                3.151e+01  1.258e+00  25.049  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.79 on 7315 degrees of freedom
## Multiple R-squared:  0.4228,	Adjusted R-squared:  0.4224 
## F-statistic:  1072 on 5 and 7315 DF,  p-value: < 2.2e-16
```

```r
AIC(mod1)
```

```
## [1] 63734.08
```

```r
#Include random effects (clustering)
mod2 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + (1|ta_sect), 
           data=master_true)
summary(mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63545.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4295 -0.6318  0.1016  0.7018  2.9276 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  18.05    4.249  
##  Residual             336.36   18.340  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.423e+02  4.962e+00 -28.672
## experiment1EXPERIMENTAL    -8.978e-01  9.476e-01  -0.948
## satmath                     1.452e-01  4.050e-03  35.860
## satverbal                   3.190e-02  4.121e-03   7.742
## mastered_topics_initial_kc  3.039e-01  1.439e-02  21.117
## high_sch_gpa                3.153e+01  1.257e+00  25.091
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___
## e1EXPERIMEN -0.109                            
## satmath     -0.144  0.009                     
## satverbal   -0.108  0.005 -0.604              
## mstrd_tpc__  0.024  0.003 -0.187 -0.015       
## high_sch_gp -0.874  0.001 -0.061 -0.108 -0.012
```

```r
AIC(mod2)
```

```
## [1] 63561.45
```

```r
AIC(mod1, mod2)
```

```
##      df      AIC
## mod1  7 63734.08
## mod2  8 63561.45
```
####Again, the change of AIC is much bigger than 2




