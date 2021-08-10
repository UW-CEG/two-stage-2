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
df2 <- df %>% select(course_fullid)

df3 <- df2 %>% mutate(year = str_extract(course_fullid, "\\d\\d\\d\\d")) 

df4 <- df3 %>% 
  mutate(year = ifelse(str_detect(course_fullid, "2016"), 
                       paste(year, "16", sep = "_"),
                       paste(year, "17", sep = "_")))
```
#### NOTE: I am trying to add A and B after the year, but I can't seem to make it work 
#### Now using my orginal method.Creating unique rows and remove the ones that are not necessary for this data model

```r
data_2s_extras_removed = subset(df, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)

data_2s_exam_score <- data_2s_unique %>% 
  select(two_stage_id, exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, experiment1, ta_sect, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc)

data_2s_exam_score_true_2 <- unique(data_2s_exam_score)
data_2s_exam_score_true <- na.omit(data_2s_exam_score_true_2)
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
## -91.108 -12.022   1.739  13.031  54.166 
## 
## Coefficients:
##                                        Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                          -1.435e+02  9.109e+00 -15.758  < 2e-16 ***
## experiment1EXPERIMENTAL              -1.920e+00  1.225e+00  -1.568    0.117    
## sex_idFemale                         -4.904e+00  1.227e+00  -3.998 6.60e-05 ***
## satmath                               1.365e-01  7.744e-03  17.627  < 2e-16 ***
## satverbal                             4.204e-02  7.801e-03   5.388 7.89e-08 ***
## mastered_topics_initial_kc            3.129e-01  2.726e-02  11.477  < 2e-16 ***
## high_sch_gpa                          3.228e+01  2.366e+00  13.646  < 2e-16 ***
## experiment1EXPERIMENTAL:sex_idFemale  1.597e+00  1.651e+00   0.967    0.334    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.89 on 2126 degrees of freedom
## Multiple R-squared:  0.4305,	Adjusted R-squared:  0.4287 
## F-statistic: 229.6 on 7 and 2126 DF,  p-value: < 2.2e-16
```

```r
AIC(mod1)
```

```
## [1] 18607.93
```

```r
#include `sex_id` as an interacting variable
mod2 <- lmer(finalexam ~ experiment1*sex_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + (1|ta_sect),
           data=master_true)
summary(mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * sex_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18596.3
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.7243 -0.6224  0.1005  0.6993  2.8682 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   6.69    2.587  
##  Residual             350.30   18.716  
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                                        Estimate Std. Error t value
## (Intercept)                          -1.428e+02  9.115e+00 -15.668
## experiment1EXPERIMENTAL              -1.928e+00  1.328e+00  -1.452
## sex_idFemale                         -4.849e+00  1.227e+00  -3.954
## satmath                               1.363e-01  7.741e-03  17.604
## satverbal                             4.152e-02  7.791e-03   5.329
## mastered_topics_initial_kc            3.142e-01  2.730e-02  11.509
## high_sch_gpa                          3.220e+01  2.363e+00  13.626
## experiment1EXPERIMENTAL:sex_idFemale  1.599e+00  1.653e+00   0.967
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL sx_dFm satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.042                                                   
## sex_idFemal      0.024  0.484                                            
## satmath         -0.143  0.012           0.175                            
## satverbal       -0.097 -0.018          -0.127 -0.611                     
## mstrd_tpc__      0.041  0.003           0.029 -0.171 -0.037              
## high_sch_gp     -0.872 -0.032          -0.131 -0.084 -0.101 -0.026       
## e1EXPERIMENTAL: -0.003 -0.684          -0.711  0.006  0.024  0.000  0.039
```

```r
AIC(mod2)
```

```
## [1] 18616.32
```

```r
AIC(mod1, mod2)
```

```
##      df      AIC
## mod1  9 18607.93
## mod2 10 18616.32
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
## -89.144 -12.218   2.052  13.125  55.655 
## 
## Coefficients:
##                                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                       -1.452e+02  9.610e+00 -15.111  < 2e-16 ***
## experiment1EXPERIMENTAL           -1.083e+00  9.352e-01  -1.158    0.247    
## eop_idEOP                          1.248e+00  1.578e+00   0.791    0.429    
## satmath                            1.477e-01  7.710e-03  19.152  < 2e-16 ***
## satverbal                          3.737e-02  7.828e-03   4.774 1.93e-06 ***
## mastered_topics_initial_kc         3.172e-01  2.739e-02  11.579  < 2e-16 ***
## high_sch_gpa                       3.074e+01  2.360e+00  13.024  < 2e-16 ***
## experiment1EXPERIMENTAL:eop_idEOP -9.189e-02  1.992e+00  -0.046    0.963    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.99 on 2126 degrees of freedom
## Multiple R-squared:  0.4247,	Adjusted R-squared:  0.4228 
## F-statistic: 224.2 on 7 and 2126 DF,  p-value: < 2.2e-16
```

```r
AIC(mod3)
```

```
## [1] 18629.78
```

```r
#include `sex_id` as an interacting variable
mod4 <- lmer(finalexam ~ experiment1*eop_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + (1|ta_sect),
           data=master_true)
summary(mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * eop_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18616.6
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5923 -0.6180  0.1026  0.6874  2.9337 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   7.165   2.677  
##  Residual             353.516  18.802  
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.446e+02  9.608e+00 -15.053
## experiment1EXPERIMENTAL           -1.178e+00  1.077e+00  -1.094
## eop_idEOP                          1.045e+00  1.577e+00   0.663
## satmath                            1.472e-01  7.708e-03  19.097
## satverbal                          3.692e-02  7.814e-03   4.724
## mastered_topics_initial_kc         3.185e-01  2.744e-02  11.604
## high_sch_gpa                       3.076e+01  2.359e+00  13.035
## experiment1EXPERIMENTAL:eop_idEOP  2.777e-01  1.999e+00   0.139
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ep_EOP satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.046                                                   
## eop_idEOP       -0.219  0.268                                            
## satmath         -0.211  0.025           0.172                            
## satverbal       -0.132 -0.002           0.111 -0.546                     
## mstrd_tpc__      0.050 -0.002          -0.040 -0.192 -0.036              
## high_sch_gp     -0.861 -0.026           0.041 -0.028 -0.113 -0.023       
## e1EXPERIMENTAL:  0.014 -0.409          -0.724 -0.032 -0.015  0.018  0.036
```

```r
AIC(mod4)
```

```
## [1] 18636.63
```

```r
AIC(mod3, mod4)
```

```
##      df      AIC
## mod3  9 18629.78
## mod4 10 18636.63
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
## -89.868 -12.137   2.061  13.355  56.957 
## 
## Coefficients:
##                                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                       -1.369e+02  9.520e+00 -14.380  < 2e-16 ***
## experiment1EXPERIMENTAL           -8.684e-02  9.752e-01  -0.089   0.9291    
## fgn_idFGN                          8.706e-02  1.364e+00   0.064   0.9491    
## satmath                            1.438e-01  7.584e-03  18.961  < 2e-16 ***
## satverbal                          3.216e-02  8.024e-03   4.009 6.32e-05 ***
## mastered_topics_initial_kc         3.197e-01  2.735e-02  11.689  < 2e-16 ***
## high_sch_gpa                       3.018e+01  2.348e+00  12.851  < 2e-16 ***
## experiment1EXPERIMENTAL:fgn_idFGN -3.600e+00  1.821e+00  -1.977   0.0482 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.96 on 2126 degrees of freedom
## Multiple R-squared:  0.4262,	Adjusted R-squared:  0.4243 
## F-statistic: 225.6 on 7 and 2126 DF,  p-value: < 2.2e-16
```

```r
AIC(mod5)
```

```
## [1] 18624.2
```

```r
#include `fgn_id` as an interacting variable
mod6 <- lmer(finalexam ~ experiment1*fgn_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + (1|ta_sect),
           data=master_true)
summary(mod6)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * fgn_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18612.2
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6388 -0.6343  0.1108  0.6914  2.9997 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   6.662   2.581  
##  Residual             353.064  18.790  
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.366e+02  9.521e+00 -14.348
## experiment1EXPERIMENTAL           -1.748e-01  1.104e+00  -0.158
## fgn_idFGN                          1.590e-02  1.367e+00   0.012
## satmath                            1.435e-01  7.585e-03  18.920
## satverbal                          3.195e-02  8.015e-03   3.986
## mastered_topics_initial_kc         3.207e-01  2.739e-02  11.708
## high_sch_gpa                       3.019e+01  2.348e+00  12.860
## experiment1EXPERIMENTAL:fgn_idFGN -3.324e+00  1.821e+00  -1.826
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL fg_FGN satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.046                                                   
## fgn_idFGN       -0.184  0.344                                            
## satmath         -0.186  0.018           0.089                            
## satverbal       -0.161 -0.002           0.186 -0.535                     
## mstrd_tpc__      0.045  0.003          -0.019 -0.190 -0.036              
## high_sch_gp     -0.852 -0.027          -0.009 -0.044 -0.115 -0.021       
## e1EXPERIMENTAL: -0.031 -0.474          -0.672  0.016  0.017  0.000  0.045
```

```r
AIC(mod6)
```

```
## [1] 18632.2
```

```r
AIC(mod5, mod6)
```

```
##      df     AIC
## mod5  9 18624.2
## mod6 10 18632.2
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
## -89.083 -12.222   2.064  13.146  55.564 
## 
## Coefficients:
##                                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                       -1.448e+02  9.459e+00 -15.310  < 2e-16 ***
## experiment1EXPERIMENTAL           -1.167e+00  8.837e-01  -1.320    0.187    
## urm_idURM                          1.088e+00  1.875e+00   0.580    0.562    
## satmath                            1.476e-01  7.694e-03  19.182  < 2e-16 ***
## satverbal                          3.657e-02  7.752e-03   4.718 2.54e-06 ***
## mastered_topics_initial_kc         3.165e-01  2.742e-02  11.544  < 2e-16 ***
## high_sch_gpa                       3.083e+01  2.366e+00  13.031  < 2e-16 ***
## experiment1EXPERIMENTAL:urm_idURM  7.337e-01  2.459e+00   0.298    0.765    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.99 on 2126 degrees of freedom
## Multiple R-squared:  0.4247,	Adjusted R-squared:  0.4228 
## F-statistic: 224.2 on 7 and 2126 DF,  p-value: < 2.2e-16
```

```r
AIC(mod7)
```

```
## [1] 18629.57
```

```r
#include `urm_id` as an interacting variable
mod8 <- lmer(finalexam ~ experiment1*urm_id + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + (1|ta_sect),
           data=master_true)
summary(mod8)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 * urm_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18615.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5850 -0.6235  0.1030  0.6866  2.9282 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   7.306   2.703  
##  Residual             353.350  18.798  
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.444e+02  9.463e+00 -15.254
## experiment1EXPERIMENTAL           -1.270e+00  1.035e+00  -1.227
## urm_idURM                          6.917e-01  1.873e+00   0.369
## satmath                            1.471e-01  7.690e-03  19.126
## satverbal                          3.620e-02  7.741e-03   4.676
## mastered_topics_initial_kc         3.178e-01  2.746e-02  11.572
## high_sch_gpa                       3.086e+01  2.365e+00  13.053
## experiment1EXPERIMENTAL:urm_idURM  1.442e+00  2.465e+00   0.585
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ur_URM satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.041                                                   
## urm_idURM       -0.144  0.214                                            
## satmath         -0.198  0.027           0.163                            
## satverbal       -0.104 -0.009           0.022 -0.574                     
## mstrd_tpc__      0.053  0.003          -0.045 -0.196 -0.033              
## high_sch_gp     -0.872 -0.029           0.025 -0.028 -0.121 -0.026       
## e1EXPERIMENTAL: -0.039 -0.308          -0.723 -0.028  0.018  0.005  0.065
```

```r
AIC(mod8)
```

```
## [1] 18635.45
```

```r
AIC(mod7, mod8)
```

```
##      df      AIC
## mod7  9 18629.57
## mod8 10 18635.45
```
#### NOTE: The AIC differences are all greater than 2 when interacting with different demographics.  

### Models without unique `ta_sect`

```r
df5_true <- rename(data_2s_exam_score_true)

#First, test that the treatment impacts scores
mod9 <- lm(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa, 
           data=df5_true)
summary(mod9)
```

```
## 
## Call:
## lm(formula = finalexam ~ experiment1 + satmath + satverbal + 
##     mastered_topics_initial_kc + high_sch_gpa, data = df5_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.284 -12.165   2.009  13.061  55.053 
## 
## Coefficients:
##                              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                -1.420e+02  9.149e+00 -15.518  < 2e-16 ***
## experiment1EXPERIMENTAL    -1.056e+00  8.237e-01  -1.282      0.2    
## satmath                     1.458e-01  7.522e-03  19.384  < 2e-16 ***
## satverbal                   3.609e-02  7.740e-03   4.663 3.31e-06 ***
## mastered_topics_initial_kc  3.183e-01  2.737e-02  11.631  < 2e-16 ***
## high_sch_gpa                3.050e+01  2.348e+00  12.990  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2128 degrees of freedom
## Multiple R-squared:  0.4243,	Adjusted R-squared:  0.423 
## F-statistic: 313.7 on 5 and 2128 DF,  p-value: < 2.2e-16
```

```r
AIC(mod9)
```

```
## [1] 18627.01
```

```r
#Include random effects (clustering)
mod10 <- lmer(finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc + high_sch_gpa + (1|ta_sect), 
           data=df5_true)
summary(mod10)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: df5_true
## 
## REML criterion at convergence: 18620
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4647 -0.6148  0.1089  0.6888  2.8570 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   6.677   2.584  
##  Residual             353.854  18.811  
## Number of obs: 2134, groups:  ta_sect, 57
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.410e+02  9.151e+00 -15.414
## experiment1EXPERIMENTAL    -1.071e+00  8.269e-01  -1.295
## satmath                     1.454e-01  7.504e-03  19.376
## satverbal                   3.679e-02  7.713e-03   4.770
## mastered_topics_initial_kc  3.175e-01  2.736e-02  11.605
## high_sch_gpa                3.022e+01  2.342e+00  12.906
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___
## e1EXPERIMEN -0.053                            
## satmath     -0.158  0.034                     
## satverbal   -0.094 -0.005 -0.598              
## mstrd_tpc__  0.036  0.005 -0.189 -0.030       
## high_sch_gp -0.878 -0.012 -0.047 -0.128 -0.017
```

```r
AIC(mod10)
```

```
## [1] 18635.96
```

```r
AIC(mod9,mod10)
```

```
##       df      AIC
## mod9   7 18627.01
## mod10  8 18635.96
```
#### Values with unique `ta_sect` are: 	
df
<dbl>
AIC
<dbl>
mod1 (linear fit)	7	63734.08		
mod2 (random effect)	63561.45

#### NOTE: When considering random effects with unique `ta_sect`, the AIC value decreases compared to the AIC value without unique `ta_sect`

## Model Secletion
#### Finding the best fit model for `sex_id`

```r
# Making a model that has fixed effects only with `lm()`
sex_mod1 <- lm(finalexam ~ experiment1 * sex_id + experiment1 +
                 sex_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa, 
               data = master_true)

# Add in random effects for `ta_sect`
sex_mod2 <- lmer(finalexam ~ experiment1 * sex_id + experiment1 +
                   sex_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect),
                 data = master_true, REML = TRUE)

AIC(sex_mod1, sex_mod2)
```

```
##          df      AIC
## sex_mod1  9 18607.93
## sex_mod2 10 18616.32
```

```r
# create a model with REML turned off 
sex_mod3 <- lmer(finalexam ~ experiment1 * sex_id + experiment1 +
                   sex_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect),
                 data = master_true, REML = FALSE)

# remove experiment1 * dem_id from the best model predicted 
sex_mod3.a <- lmer(finalexam ~ experiment1 + sex_id + satmath + 
                     satverbal + mastered_topics_initial_kc +
                     high_sch_gpa + (1 | ta_sect), 
                   data = master_true, REML = FALSE)

AIC(sex_mod3, sex_mod3.a)
```

```
##            df      AIC
## sex_mod3   10 18604.59
## sex_mod3.a  9 18603.53
```

```r
# remove `experiment1` because it has the lowest t value 


#Refit model 3 with lmer and REML = TRUE 
sex_mod4 <- lmer(finalexam ~ experiment1 + sex_id + satmath + 
                   satverbal + mastered_topics_initial_kc +  high_sch_gpa + 
                   (1 | ta_sect), data = master_true, REML = TRUE)

summary(sex_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + sex_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18600.1
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.7466 -0.6261  0.1014  0.7026  2.8438 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   6.686   2.586  
##  Residual             350.292  18.716  
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.428e+02  9.114e+00 -15.665
## experiment1EXPERIMENTAL    -1.049e+00  9.688e-01  -1.083
## sex_idFemale               -4.005e+00  8.625e-01  -4.644
## satmath                     1.362e-01  7.741e-03  17.599
## satverbal                   4.134e-02  7.788e-03   5.307
## mastered_topics_initial_kc  3.142e-01  2.730e-02  11.508
## high_sch_gpa                3.211e+01  2.361e+00  13.599
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE sx_dFm satmth stvrbl mst___
## e1EXPERIMEN -0.061                                   
## sex_idFemal  0.031 -0.004                            
## satmath     -0.143  0.022  0.255                     
## satverbal   -0.097 -0.002 -0.156 -0.611              
## mstrd_tpc__  0.041  0.004  0.042 -0.171 -0.037       
## high_sch_gp -0.873 -0.007 -0.147 -0.084 -0.102 -0.026
```
#### NOTE: The AIC value went up after adding in `ta_sect` random effects. Does this mean we no longer need to include the random effects?

#### Finding the best fit model for `eop_id`

```r
# Making a model that has fixed effects only with `lm()`
eop_mod1 <- lm(finalexam ~ experiment1 * eop_id + experiment1 + eop_id
               + satmath + satverbal + mastered_topics_initial_kc +
                 high_sch_gpa, data = master_true)

# Add in random effects for `ta_sect`
eop_mod2 <- lmer(finalexam ~ experiment1 * eop_id + experiment1 +
                   eop_id + satmath + satverbal + mastered_topics_initial_kc +  
                   high_sch_gpa + 
                   (1 | ta_sect), data = master_true, REML = TRUE)

AIC(eop_mod1, eop_mod2)
```

```
##          df      AIC
## eop_mod1  9 18629.78
## eop_mod2 10 18636.63
```

```r
# Turn off REML 
eop_mod3 <- lmer(finalexam ~ experiment1 * eop_id + experiment1 +
                   eop_id + satmath + satverbal + mastered_topics_initial_kc +
                   high_sch_gpa + (1 | ta_sect), data = master_true, REML = FALSE)

# remove experiment1 * dem_id from the best model predicted 
eop_mod3.a <- lmer(finalexam ~ experiment1 + eop_id + satmath + 
                     satverbal + mastered_topics_initial_kc +  
                     high_sch_gpa + (1 | ta_sect), data = master_true, REML = FALSE)

AIC(eop_mod3, eop_mod3.a)
```

```
##            df      AIC
## eop_mod3   10 18625.83
## eop_mod3.a  9 18623.85
```

```r
#Refit model 3 with lmer and REML = FALSE
eop_mod4 <- lmer(finalexam ~ experiment1 + eop_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

summary(eop_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + eop_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18619.9
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5952 -0.6176  0.0996  0.6861  2.9329 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   7.128   2.67   
##  Residual             353.376  18.80   
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.447e+02  9.605e+00 -15.060
## experiment1EXPERIMENTAL    -1.117e+00  9.821e-01  -1.137
## eop_idEOP                   1.204e+00  1.087e+00   1.107
## satmath                     1.472e-01  7.702e-03  19.116
## satverbal                   3.694e-02  7.812e-03   4.728
## mastered_topics_initial_kc  3.184e-01  2.743e-02  11.606
## high_sch_gpa                3.074e+01  2.357e+00  13.042
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE ep_EOP satmth stvrbl mst___
## e1EXPERIMEN -0.044                                   
## eop_idEOP   -0.303 -0.045                            
## satmath     -0.210  0.013  0.216                     
## satverbal   -0.132 -0.010  0.145 -0.547              
## mstrd_tpc__  0.049  0.006 -0.039 -0.192 -0.036       
## high_sch_gp -0.862 -0.012  0.097 -0.027 -0.112 -0.024
```
#### Finding the best fit model for `fgn_id`

```r
# Making a model that has fixed effects only with `lm()`
fgn_mod1 <- lm(finalexam ~ experiment1 * fgn_id + experiment1 
               + fgn_id + satmath + satverbal + mastered_topics_initial_kc 
               +  high_sch_gpa, data = master_true)

# Add in random effects for `ta_sect`
fgn_mod2 <- lmer(finalexam ~ experiment1 * fgn_id + experiment1 + 
                   fgn_id + satmath + satverbal + 
                   mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect),
                 data = master_true, REML = TRUE)

AIC(fgn_mod1, fgn_mod2)
```

```
##          df     AIC
## fgn_mod1  9 18624.2
## fgn_mod2 10 18632.2
```

```r
# turn off REML
fgn_mod3 <- lmer(finalexam ~ experiment1 * fgn_id + experiment1 + 
                   fgn_id + satmath + satverbal + 
                   mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect),
                 data = master_true, REML = FALSE)

# remove experiment1 * dem_id from the best model predicted 
fgn_mod3.a <- lmer(finalexam ~ experiment1 + fgn_id + satmath + 
                   satverbal + mastered_topics_initial_kc + 
                   high_sch_gpa + (1 | ta_sect), data = master_true, 
                 REML = FALSE)

AIC(fgn_mod3, fgn_mod3.a)
```

```
##            df      AIC
## fgn_mod3   10 18621.02
## fgn_mod3.a  9 18622.38
```

```r
#Refit model 3 with lmer and REML = FALSE
fgn_mod4 <- lmer(finalexam ~ experiment1 + fgn_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

summary(fgn_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + fgn_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18618.6
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6124 -0.6326  0.1048  0.6861  2.9396 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   6.981   2.642  
##  Residual             353.233  18.794  
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.371e+02  9.521e+00 -14.403
## experiment1EXPERIMENTAL    -1.129e+00  9.786e-01  -1.154
## fgn_idFGN                  -1.659e+00  1.013e+00  -1.637
## satmath                     1.437e-01  7.588e-03  18.940
## satverbal                   3.219e-02  8.018e-03   4.015
## mastered_topics_initial_kc  3.208e-01  2.741e-02  11.702
## high_sch_gpa                3.038e+01  2.347e+00  12.948
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE fg_FGN satmth stvrbl mst___
## e1EXPERIMEN -0.069                                   
## fgn_idFGN   -0.276  0.038                            
## satmath     -0.186  0.028  0.135                     
## satverbal   -0.161  0.007  0.267 -0.535              
## mstrd_tpc__  0.045  0.003 -0.026 -0.190 -0.036       
## high_sch_gp -0.852 -0.007  0.028 -0.045 -0.116 -0.021
```
#### Finding the best fit model for `urm_id`

```r
# Making a model that has fixed effects only with `lm()`
urm_mod1 <- lm(finalexam ~ experiment1 * urm_id + experiment1 + 
                 urm_id + satmath + satverbal + 
                 mastered_topics_initial_kc +  high_sch_gpa, data = master_true)

# Add in random effects for `ta_sect`
urm_mod2 <- lmer(finalexam ~ experiment1 * urm_id + experiment1 + 
                   urm_id + satmath + satverbal + mastered_topics_initial_kc +
                   high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

AIC(urm_mod1, urm_mod2)
```

```
##          df      AIC
## urm_mod1  9 18629.57
## urm_mod2 10 18635.45
```

```r
#turn off REML
urm_mod3 <- lmer(finalexam ~ experiment1 * urm_id + experiment1 + 
                   urm_id + satmath + satverbal + mastered_topics_initial_kc +
                   high_sch_gpa + (1 | ta_sect), 
                 data = master_true, REML = TRUE)

# remove experiment1 * dem_id from the best model predicted 
urm_mod3.a <- lmer(finalexam ~ experiment1 + urm_id + satmath + 
                     satverbal + mastered_topics_initial_kc +
                     high_sch_gpa + (1 | ta_sect), 
                   data = master_true, REML = FALSE)

AIC(urm_mod3, urm_mod3.a)
```

```
##            df      AIC
## urm_mod3   10 18635.45
## urm_mod3.a  9 18623.76
```

```r
#Because the AIC decreased by 2, we now can remove experiment1
urm_mod3.b <- lmer(finalexam ~ urm_id + satmath + 
                     satverbal + mastered_topics_initial_kc +
                     high_sch_gpa + (1 | ta_sect), 
                   data = master_true, REML = FALSE)

AIC(urm_mod3.a, urm_mod3.b)
```

```
##            df      AIC
## urm_mod3.a  9 18623.76
## urm_mod3.b  8 18623.00
```

```r
#Because it only decreased by 1, experiment 1 is retained. Next try to remove `urm_id`
urm_mod3.c <- lmer(finalexam ~ experiment1 + satmath + 
                     satverbal + mastered_topics_initial_kc +
                     high_sch_gpa + (1 | ta_sect), 
                   data = master_true, REML = FALSE)
AIC(urm_mod3.a, urm_mod3.c)
```

```
##            df      AIC
## urm_mod3.a  9 18623.76
## urm_mod3.c  8 18623.08
```

```r
# AIC value went up, meaning the best model should be urm_mod3.a

#Refit model 3 with lmer and REML = TRUE
urm_mod4 <- lmer(finalexam ~ experiment1 + urm_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

summary(urm_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## finalexam ~ experiment1 + urm_id + satmath + satverbal + mastered_topics_initial_kc +  
##     high_sch_gpa + (1 | ta_sect)
##    Data: master_true
## 
## REML criterion at convergence: 18619.4
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.5930 -0.6225  0.1024  0.6852  2.9233 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   7.106   2.666  
##  Residual             353.376  18.798  
## Number of obs: 2134, groups:  ta_sect, 110
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.441e+02  9.454e+00 -15.247
## experiment1EXPERIMENTAL    -1.084e+00  9.807e-01  -1.105
## urm_idURM                   1.484e+00  1.294e+00   1.147
## satmath                     1.472e-01  7.686e-03  19.153
## satverbal                   3.613e-02  7.739e-03   4.668
## mastered_topics_initial_kc  3.177e-01  2.746e-02  11.570
## high_sch_gpa                3.077e+01  2.359e+00  13.045
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE ur_URM satmth stvrbl mst___
## e1EXPERIMEN -0.055                                   
## urm_idURM   -0.250 -0.014                            
## satmath     -0.199  0.020  0.206                     
## satverbal   -0.103 -0.004  0.050 -0.574              
## mstrd_tpc__  0.053  0.005 -0.059 -0.196 -0.033       
## high_sch_gp -0.872 -0.009  0.104 -0.026 -0.122 -0.026
```
#### The AIC values all went up after adding the random effect. Does it mean we no longer need to include the random effect? Also, what do we need to do for our next step then if we now do not have to consider random effect?









