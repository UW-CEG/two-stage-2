---
title: "TA_Random_Effects_Interaction"
author: "Ganling"
date: "5/9/2021"
output: 
  html_document:
    keep_md: yes
---
### Setting up the data set 
#### Read libraries 

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
library(fs) # file system operations functions
library(readxl)
library(janitor)
```

```
## 
## Attaching package: 'janitor'
```

```
## The following objects are masked from 'package:stats':
## 
##     chisq.test, fisher.test
```

```r
library(knitr)
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
#### Load the data set

```r
original_data_dir   <- here("original-data", "/")
importable_data_dir <- here("processing-and-analysis", "01-importable-data", "/")
analysis_data_dir   <- here("processing-and-analysis", "03-analysis-data", "/")

df <- readRDS("G:/Shared drives/CEG Two-Stage Exams Analysis/Ganling/two-stage-2/original-data/master_2s_small_deid_scaled.rds")
```
### data wrangling 
#### creating data frames with unique `ta_sect` 

```r
# df2 <- df %>% select(course_fullid)

# df3 <- df2 %>% mutate(year = str_extract(course_fullid, "\\d\\d\\d\\d")) "\\d{4}"

# df4 <- df3 %>% 
  # mutate(year = ifelse(str_detect(course_fullid, "2016"), 
                       # paste(year, "A", sep = "_"),
                       # paste(year, "B", sep = "_")))

# df5 <- df3 %>% 
  # mutate(year2 = ifelse(str_detect(course_fullid, "2017"), 
                       # paste(year, "A", sep = "_"),
                       # paste(year, "B", sep = "_")))
```
#### NOTE: I am trying to add A and B after the year, but I can't seem to make it work 
#### Now using my orginal method.Creating unique rows and remove the ones that are not necessary for this data model

```r
data_2s_extras_removed = subset(df, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)
View(data_2s_unique)
data_2s_exam_score <- data_2s_unique %>% 
  select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, experiment1, ta_sect, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc)
  
data_2s_exam_score_true <- na.omit(data_2s_exam_score)
```
### Make `ta_sect` unique across different data values
#### First, we need to split the data from into year 16 and year 17 

```r
data_2s_exam_score_true$ta_sect <- as.character(data_2s_exam_score_true$ta_sect)

#next, split the dataframe in half
master_unique_2016 <- subset(data_2s_exam_score_true, course_fullid != "CHEM_142_A_2017_4" & course_fullid != "CHEM_142_B_2017_4")
master_unique_2017 <- subset(data_2s_exam_score_true, course_fullid != "CHEM_142_A_2016_4" & course_fullid != "CHEM_142_B_2016_4")
```
#### Use the vectorization function in R, we can add `_16` and `_17` at the end of each TA section 

```r
master_unique_2016$ta_sect <- paste0(master_unique_2016$ta_sect, "_16")
master_unique_2017$ta_sect <- paste0(master_unique_2017$ta_sect, "_17")
```
#### recombine the dataframes together and make the `ta_sect` factors again

```r
master_true <- rbind(master_unique_2016, master_unique_2017)
master_true$ta_sect <- as.factor(master_true$ta_sect)
```
### Interaction Models
#### Creating interaction models based on the demographics, `sex_id`, `eop_id`, `fgn_id` and `urm_id`

```r
# Without random effect but set `sex_id` as an interacting variable
mod1 <- lm(finalexam ~ experiment1*sex_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa, 
           data=master_true)
summary(mod1)
```

```
## 
## Call:
## lm(formula = finalexam ~ experiment1 * sex_id + satmath + satverbal + 
##     mastered_topics_initial_kc + high_sch_gpa, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -90.495 -11.839   1.845  12.953  54.056 
## 
## Coefficients:
##                                        Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                          -1.453e+02  4.914e+00 -29.566  < 2e-16 ***
## experiment1EXPERIMENTAL              -1.777e+00  6.586e-01  -2.698  0.00699 ** 
## sex_idFemale                         -4.748e+00  6.694e-01  -7.093 1.44e-12 ***
## satmath                               1.390e-01  4.164e-03  33.379  < 2e-16 ***
## satverbal                             3.794e-02  4.168e-03   9.101  < 2e-16 ***
## mastered_topics_initial_kc            2.951e-01  1.429e-02  20.655  < 2e-16 ***
## high_sch_gpa                          3.321e+01  1.267e+00  26.216  < 2e-16 ***
## experiment1EXPERIMENTAL:sex_idFemale  1.483e+00  8.848e-01   1.676  0.09379 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.7 on 7313 degrees of freedom
## Multiple R-squared:  0.4287,	Adjusted R-squared:  0.4282 
## F-statistic:   784 on 7 and 7313 DF,  p-value: < 2.2e-16
```

```r
AIC(mod1)
```

```
## [1] 63662.44
```

```r
#include `sex_id` as an interacting variable
mod2 <- lmer(finalexam ~ experiment1*sex_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + sex_id + (1|ta_sect),
           data=master_true)
summary(mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * sex_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + sex_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63474.2
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5423 -0.6277  0.0926  0.6971  2.8860 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  17.31    4.161  
##  Residual             333.40   18.259  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                                        Estimate Std. Error t value
## (Intercept)                          -1.432e+02  4.940e+00 -28.983
## experiment1EXPERIMENTAL              -1.773e+00  1.052e+00  -1.686
## sex_idFemale                         -4.614e+00  6.691e-01  -6.896
## satmath                               1.370e-01  4.159e-03  32.933
## satverbal                             3.700e-02  4.148e-03   8.921
## mastered_topics_initial_kc            2.987e-01  1.434e-02  20.829
## high_sch_gpa                          3.307e+01  1.264e+00  26.153
## experiment1EXPERIMENTAL:sex_idFemale  1.559e+00  8.880e-01   1.756
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL sx_dFm satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.102                                                   
## sex_idFemal      0.008  0.338                                            
## satmath         -0.133  0.006           0.165                            
## satverbal       -0.110 -0.005          -0.115 -0.614                     
## mstrd_tpc__      0.025  0.001           0.028 -0.170 -0.021              
## high_sch_gp     -0.867 -0.012          -0.118 -0.094 -0.084 -0.019       
## e1EXPERIMENTAL:  0.014 -0.465          -0.725  0.005  0.020  0.004  0.027
```

```r
AIC(mod2)
```

```
## [1] 63494.22
```

```r
AIC(mod1, mod2)
```

```
##      df      AIC
## mod1  9 63662.44
## mod2 10 63494.22
```

```r
# Without random effect but set `eop_id` as an interacting variable
mod3 <- lm(finalexam ~ experiment1*eop_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa, 
           data=master_true)
summary(mod3)
```

```
## 
## Call:
## lm(formula = finalexam ~ experiment1 * eop_id + satmath + satverbal + 
##     mastered_topics_initial_kc + high_sch_gpa, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.584 -11.995   1.945  12.916  55.647 
## 
## Coefficients:
##                                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                       -1.476e+02  5.172e+00 -28.538  < 2e-16 ***
## experiment1EXPERIMENTAL           -9.559e-01  5.009e-01  -1.908   0.0564 .  
## eop_idEOP                          1.394e+00  8.580e-01   1.625   0.1043    
## satmath                            1.497e-01  4.136e-03  36.189  < 2e-16 ***
## satverbal                          3.399e-02  4.190e-03   8.113 5.76e-16 ***
## mastered_topics_initial_kc         2.996e-01  1.435e-02  20.883  < 2e-16 ***
## high_sch_gpa                       3.179e+01  1.264e+00  25.143  < 2e-16 ***
## experiment1EXPERIMENTAL:eop_idEOP -1.209e-01  1.069e+00  -0.113   0.9100    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.79 on 7313 degrees of freedom
## Multiple R-squared:  0.4232,	Adjusted R-squared:  0.4227 
## F-statistic: 766.5 on 7 and 7313 DF,  p-value: < 2.2e-16
```

```r
AIC(mod3)
```

```
## [1] 63732.78
```

```r
#include `sex_id` as an interacting variable
mod4 <- lmer(finalexam ~ experiment1*eop_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + sex_id + (1|ta_sect),
           data=master_true)
summary(mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * eop_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + sex_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63470.4
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5559 -0.6338  0.0937  0.7025  2.8998 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  17.36    4.167  
##  Residual             333.31   18.257  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.469e+02  5.169e+00 -28.412
## experiment1EXPERIMENTAL           -1.090e+00  9.619e-01  -1.133
## eop_idEOP                          1.041e+00  8.547e-01   1.218
## satmath                            1.388e-01  4.237e-03  32.758
## satverbal                          3.826e-02  4.192e-03   9.128
## mastered_topics_initial_kc         2.975e-01  1.435e-02  20.729
## high_sch_gpa                       3.333e+01  1.271e+00  26.225
## sex_idFemale                      -3.776e+00  4.610e-01  -8.190
## experiment1EXPERIMENTAL:eop_idEOP  5.497e-01  1.075e+00   0.511
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ep_EOP satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.094                                                   
## eop_idEOP       -0.205  0.163                                            
## satmath         -0.181  0.009           0.143                            
## satverbal       -0.148  0.005           0.111 -0.569                     
## mstrd_tpc__      0.035 -0.002          -0.043 -0.174 -0.027              
## high_sch_gp     -0.854 -0.011           0.041 -0.074 -0.069 -0.021       
## sex_idFemal      0.028  0.001          -0.012  0.238 -0.147  0.046 -0.144
## e1EXPERIMENTAL:  0.007 -0.244          -0.736 -0.019 -0.017  0.024  0.037
##                 sx_dFm
## ex1EXPERIMENTAL       
## eop_idEOP             
## satmath               
## satverbal             
## mstrd_tpc__           
## high_sch_gp           
## sex_idFemal           
## e1EXPERIMENTAL:  0.004
```

```r
AIC(mod4)
```

```
## [1] 63492.37
```

```r
AIC(mod3, mod4)
```

```
##      df      AIC
## mod3  9 63732.78
## mod4 11 63492.37
```

```r
# Without random effect but set `fgn_id` as an interacting variable
mod5 <- lm(finalexam ~ experiment1*fgn_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa, 
           data=master_true)
summary(mod5)
```

```
## 
## Call:
## lm(formula = finalexam ~ experiment1 * fgn_id + satmath + satverbal + 
##     mastered_topics_initial_kc + high_sch_gpa, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.318 -12.063   1.875  13.113  56.880 
## 
## Coefficients:
##                                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                       -1.391e+02  5.142e+00 -27.059  < 2e-16 ***
## experiment1EXPERIMENTAL            9.202e-02  5.207e-01   0.177    0.860    
## fgn_idFGN                          3.994e-01  7.543e-01   0.529    0.597    
## satmath                            1.459e-01  4.090e-03  35.662  < 2e-16 ***
## satverbal                          2.889e-02  4.288e-03   6.738 1.73e-11 ***
## mastered_topics_initial_kc         3.020e-01  1.433e-02  21.079  < 2e-16 ***
## high_sch_gpa                       3.115e+01  1.258e+00  24.761  < 2e-16 ***
## experiment1EXPERIMENTAL:fgn_idFGN -3.859e+00  9.810e-01  -3.934 8.44e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.76 on 7313 degrees of freedom
## Multiple R-squared:  0.4247,	Adjusted R-squared:  0.4242 
## F-statistic: 771.3 on 7 and 7313 DF,  p-value: < 2.2e-16
```

```r
AIC(mod5)
```

```
## [1] 63713.53
```

```r
#include `fgn_id` as an interacting variable
mod6 <- lmer(finalexam ~ experiment1*fgn_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + sex_id + (1|ta_sect),
           data=master_true)
summary(mod6)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * fgn_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + sex_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63457.7
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5913 -0.6208  0.0951  0.7011  2.9603 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  17.02    4.125  
##  Residual             332.78   18.242  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.390e+02  5.139e+00 -27.054
## experiment1EXPERIMENTAL           -2.338e-02  9.659e-01  -0.024
## fgn_idFGN                          3.385e-01  7.540e-01   0.449
## satmath                            1.354e-01  4.197e-03  32.253
## satverbal                          3.365e-02  4.291e-03   7.841
## mastered_topics_initial_kc         2.995e-01  1.433e-02  20.900
## high_sch_gpa                       3.269e+01  1.265e+00  25.847
## sex_idFemale                      -3.754e+00  4.606e-01  -8.150
## experiment1EXPERIMENTAL:fgn_idFGN -3.334e+00  9.759e-01  -3.416
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL fg_FGN satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.097                                                   
## fgn_idFGN       -0.183  0.214                                            
## satmath         -0.166  0.012           0.101                            
## satverbal       -0.175  0.006           0.177 -0.551                     
## mstrd_tpc__      0.032  0.002          -0.022 -0.172 -0.028              
## high_sch_gp     -0.842 -0.013          -0.011 -0.089 -0.073 -0.019       
## sex_idFemal      0.026  0.002          -0.002  0.242 -0.142  0.045 -0.144
## e1EXPERIMENTAL: -0.025 -0.287          -0.694  0.000  0.016  0.003  0.049
##                 sx_dFm
## ex1EXPERIMENTAL       
## fgn_idFGN             
## satmath               
## satverbal             
## mstrd_tpc__           
## high_sch_gp           
## sex_idFemal           
## e1EXPERIMENTAL: -0.002
```

```r
AIC(mod6)
```

```
## [1] 63479.71
```

```r
AIC(mod5, mod6)
```

```
##      df      AIC
## mod5  9 63713.53
## mod6 11 63479.71
```

```r
# Without random effect but set `urm_id` as an interacting variable
mod7 <- lm(finalexam ~ experiment1*urm_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa, 
           data=master_true)
summary(mod7)
```

```
## 
## Call:
## lm(formula = finalexam ~ experiment1 * urm_id + satmath + satverbal + 
##     mastered_topics_initial_kc + high_sch_gpa, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.514 -11.973   1.904  12.929  55.571 
## 
## Coefficients:
##                                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                       -1.473e+02  5.113e+00 -28.817  < 2e-16 ***
## experiment1EXPERIMENTAL           -1.064e+00  4.734e-01  -2.247   0.0247 *  
## urm_idURM                          1.145e+00  1.022e+00   1.120   0.2627    
## satmath                            1.497e-01  4.131e-03  36.228  < 2e-16 ***
## satverbal                          3.314e-02  4.148e-03   7.989 1.57e-15 ***
## mastered_topics_initial_kc         2.988e-01  1.436e-02  20.807  < 2e-16 ***
## high_sch_gpa                       3.192e+01  1.268e+00  25.166  < 2e-16 ***
## experiment1EXPERIMENTAL:urm_idURM  9.093e-01  1.319e+00   0.690   0.4905    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.78 on 7313 degrees of freedom
## Multiple R-squared:  0.4233,	Adjusted R-squared:  0.4227 
## F-statistic: 766.8 on 7 and 7313 DF,  p-value: < 2.2e-16
```

```r
AIC(mod7)
```

```
## [1] 63731.76
```

```r
#include `urm_id` as an interacting variable
mod8 <- lmer(finalexam ~ experiment1*urm_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + sex_id + (1|ta_sect),
           data=master_true)
summary(mod8)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * urm_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + sex_id + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 63468.2
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5424 -0.6275  0.0950  0.7080  2.8890 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  17.59    4.194  
##  Residual             333.20   18.254  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.463e+02  5.116e+00 -28.602
## experiment1EXPERIMENTAL           -1.228e+00  9.528e-01  -1.289
## urm_idURM                          9.025e-02  1.021e+00   0.088
## satmath                            1.385e-01  4.241e-03  32.661
## satverbal                          3.738e-02  4.152e-03   9.002
## mastered_topics_initial_kc         2.971e-01  1.436e-02  20.699
## high_sch_gpa                       3.343e+01  1.274e+00  26.246
## sex_idFemale                      -3.713e+00  4.614e-01  -8.046
## experiment1EXPERIMENTAL:urm_idURM  2.347e+00  1.327e+00   1.768
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ur_URM satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.094                                                   
## urm_idURM       -0.147  0.128                                            
## satmath         -0.176  0.010           0.144                            
## satverbal       -0.121  0.003           0.031 -0.591                     
## mstrd_tpc__      0.038  0.003          -0.040 -0.177 -0.024              
## high_sch_gp     -0.862 -0.011           0.029 -0.071 -0.077 -0.024       
## sex_idFemal      0.013  0.000           0.024  0.249 -0.143  0.043 -0.137
## e1EXPERIMENTAL: -0.040 -0.180          -0.732 -0.013  0.009  0.005  0.062
##                 sx_dFm
## ex1EXPERIMENTAL       
## urm_idURM             
## satmath               
## satverbal             
## mstrd_tpc__           
## high_sch_gp           
## sex_idFemal           
## e1EXPERIMENTAL:  0.009
```

```r
AIC(mod8)
```

```
## [1] 63490.15
```

```r
AIC(mod7, mod8)
```

```
##      df      AIC
## mod7  9 63731.76
## mod8 11 63490.15
```
#### NOTE: The AIC differences are all greater than 2 when interacting with different demographics.  








