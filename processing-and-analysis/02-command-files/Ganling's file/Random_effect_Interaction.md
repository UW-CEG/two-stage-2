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
## REML criterion at convergence: 63537.4
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4212 -0.6244  0.0975  0.7056  2.9636 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  18.08    4.253  
##  Residual             336.20   18.336  
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.457e+02  5.192e+00 -28.058
## experiment1EXPERIMENTAL           -1.079e+00  9.774e-01  -1.104
## eop_idEOP                          9.549e-01  8.585e-01   1.112
## satmath                            1.470e-01  4.133e-03  35.574
## satverbal                          3.322e-02  4.165e-03   7.976
## mastered_topics_initial_kc         3.029e-01  1.440e-02  21.033
## high_sch_gpa                       3.183e+01  1.263e+00  25.197
## experiment1EXPERIMENTAL:eop_idEOP  5.879e-01  1.080e+00   0.544
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ep_EOP satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.095                                                   
## eop_idEOP       -0.205  0.162                                            
## satmath         -0.193  0.009           0.150                            
## satverbal       -0.145  0.006           0.111 -0.555                     
## mstrd_tpc__      0.034 -0.002          -0.042 -0.191 -0.020              
## high_sch_gp     -0.859 -0.011           0.039 -0.041 -0.092 -0.015       
## e1EXPERIMENTAL:  0.007 -0.242          -0.736 -0.021 -0.017  0.023  0.037
```

```r
AIC(mod4)
```

```
## [1] 63557.41
```

```r
AIC(mod3, mod4)
```

```
##      df      AIC
## mod3  9 63732.78
## mod4 10 63557.41
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
## REML criterion at convergence: 63524.1
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4572 -0.6310  0.0948  0.6958  3.0257 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  17.72    4.21   
##  Residual             335.65   18.32   
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.380e+02  5.161e+00 -26.729
## experiment1EXPERIMENTAL           -2.565e-03  9.811e-01  -0.003
## fgn_idFGN                          3.298e-01  7.574e-01   0.435
## satmath                            1.436e-01  4.090e-03  35.120
## satverbal                          2.867e-02  4.266e-03   6.719
## mastered_topics_initial_kc         3.048e-01  1.438e-02  21.198
## high_sch_gpa                       3.121e+01  1.257e+00  24.826
## experiment1EXPERIMENTAL:fgn_idFGN -3.351e+00  9.803e-01  -3.419
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL fg_FGN satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.098                                                   
## fgn_idFGN       -0.183  0.212                                            
## satmath         -0.177  0.012           0.105                            
## satverbal       -0.173  0.006           0.178 -0.538                     
## mstrd_tpc__      0.031  0.002          -0.022 -0.189 -0.021              
## high_sch_gp     -0.847 -0.012          -0.012 -0.056 -0.095 -0.013       
## e1EXPERIMENTAL: -0.025 -0.284          -0.695  0.001  0.015  0.003  0.049
```

```r
AIC(mod6)
```

```
## [1] 63544.11
```

```r
AIC(mod5, mod6)
```

```
##      df      AIC
## mod5  9 63713.53
## mod6 10 63544.11
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
## REML criterion at convergence: 63532.9
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.4077 -0.6256  0.0950  0.7050  2.9569 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  18.32    4.28   
##  Residual             335.98   18.33   
## Number of obs: 7321, groups:  ta_sect, 110
## 
## Fixed effects:
##                                     Estimate Std. Error t value
## (Intercept)                       -1.458e+02  5.139e+00 -28.372
## experiment1EXPERIMENTAL           -1.227e+00  9.684e-01  -1.267
## urm_idURM                          2.828e-01  1.025e+00   0.276
## satmath                            1.470e-01  4.125e-03  35.634
## satverbal                          3.259e-02  4.127e-03   7.896
## mastered_topics_initial_kc         3.021e-01  1.440e-02  20.972
## high_sch_gpa                       3.203e+01  1.267e+00  25.276
## experiment1EXPERIMENTAL:urm_idURM  2.449e+00  1.333e+00   1.837
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ur_URM satmth stvrbl mst___ hgh_s_
## ex1EXPERIMENTAL -0.095                                                   
## urm_idURM       -0.147  0.126                                            
## satmath         -0.185  0.011           0.142                            
## satverbal       -0.120  0.003           0.035 -0.579                     
## mstrd_tpc__      0.037  0.003          -0.041 -0.194 -0.018              
## high_sch_gp     -0.868 -0.011           0.033 -0.038 -0.099 -0.018       
## e1EXPERIMENTAL: -0.040 -0.178          -0.733 -0.016  0.010  0.005  0.064
```

```r
AIC(mod8)
```

```
## [1] 63552.88
```

```r
AIC(mod7, mod8)
```

```
##      df      AIC
## mod7  9 63731.76
## mod8 10 63552.88
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
AIC(mod9)
```

```
## [1] 63734.08
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
## REML criterion at convergence: 63598.7
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.2889 -0.6247  0.1003  0.6965  2.8416 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.2     3.347  
##  Residual             342.0    18.494  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                              Estimate Std. Error t value
## (Intercept)                -1.422e+02  4.948e+00 -28.737
## experiment1EXPERIMENTAL    -1.073e+00  4.423e-01  -2.426
## satmath                     1.463e-01  4.033e-03  36.274
## satverbal                   3.382e-02  4.107e-03   8.235
## mastered_topics_initial_kc  3.020e-01  1.434e-02  21.062
## high_sch_gpa                3.107e+01  1.253e+00  24.803
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satmth stvrbl mst___
## e1EXPERIMEN -0.072                            
## satmath     -0.147  0.032                     
## satverbal   -0.111  0.007 -0.603              
## mstrd_tpc__  0.022  0.001 -0.190 -0.013       
## high_sch_gp -0.876  0.001 -0.059 -0.105 -0.008
```

```r
AIC(mod10)
```

```
## [1] 63614.71
```

```r
AIC(mod9,mod10)
```

```
##       df      AIC
## mod9   7 63734.08
## mod10  8 63614.71
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
## sex_mod1  9 63662.44
## sex_mod2 10 63494.22
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
## sex_mod3   10 63474.82
## sex_mod3.a  9 63475.91
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
## sex_idFemale               -3.762e+00  4.611e-01  -8.159
## satmath                     1.369e-01  4.159e-03  32.920
## satverbal                   3.686e-02  4.147e-03   8.887
## mastered_topics_initial_kc  2.986e-01  1.434e-02  20.820
## high_sch_gpa                3.301e+01  1.264e+00  26.112
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE sx_dFm satmth stvrbl mst___
## e1EXPERIMEN -0.108                                   
## sex_idFemal  0.026  0.002                            
## satmath     -0.133  0.010  0.245                     
## satverbal   -0.110  0.005 -0.146 -0.615              
## mstrd_tpc__  0.025  0.003  0.045 -0.170 -0.021       
## high_sch_gp -0.868  0.001 -0.144 -0.094 -0.085 -0.019
```
#### Finding the best fit model for `eop_id`

```r
# Making a model that has fixed effects only with `lm()`
eop_mod1 <- lm(finalexam ~ experiment1 * eop_id + experiment1 + eop_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa, data = master_true)

# Add in random effects for `ta_sect`
eop_mod2 <- lmer(finalexam ~ experiment1 * eop_id + experiment1 + eop_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

AIC(eop_mod1, eop_mod2)
```

```
##          df      AIC
## eop_mod1  9 63732.78
## eop_mod2 10 63557.41
```

```r
# remove experiment1 * dem_id from the best model predicted 
eop_mod3 <- lmer(finalexam ~ experiment1 + eop_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = FALSE)

AIC(eop_mod2, eop_mod3)
```

```
##          df      AIC
## eop_mod2 10 63557.41
## eop_mod3  9 63537.26
```

```r
#Refit model 3 with lmer and REML = FALSE
eop_mod4 <- lmer(finalexam ~ experiment1 + eop_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

AIC(eop_mod3, eop_mod4)
```

```
##          df      AIC
## eop_mod3  9 63537.26
## eop_mod4  9 63557.70
```
#### Finding the best fit model for `fgn_id`

```r
# Making a model that has fixed effects only with `lm()`
fgn_mod1 <- lm(finalexam ~ experiment1 * fgn_id + experiment1 + fgn_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa, data = master_true)

# Add in random effects for `ta_sect`
fgn_mod2 <- lmer(finalexam ~ experiment1 * fgn_id + experiment1 + fgn_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

AIC(fgn_mod1, fgn_mod2)
```

```
##          df      AIC
## fgn_mod1  9 63713.53
## fgn_mod2 10 63544.11
```

```r
# remove experiment1 * dem_id from the best model predicted 
fgn_mod3 <- lmer(finalexam ~ experiment1 + fgn_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = FALSE)

AIC(fgn_mod2, fgn_mod3)
```

```
##          df      AIC
## fgn_mod2 10 63544.11
## fgn_mod3  9 63535.00
```

```r
#Refit model 3 with lmer and REML = FALSE
fgn_mod4 <- lmer(finalexam ~ experiment1 + fgn_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

AIC(fgn_mod3, fgn_mod4)
```

```
##          df      AIC
## fgn_mod3  9 63535.00
## fgn_mod4  9 63555.58
```
#### Finding the best fit model for `urm_id`

```r
# Making a model that has fixed effects only with `lm()`
urm_mod1 <- lm(finalexam ~ experiment1 * urm_id + experiment1 + urm_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa, data = master_true)

# Add in random effects for `ta_sect`
urm_mod2 <- lmer(finalexam ~ experiment1 * urm_id + experiment1 + urm_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = TRUE)

AIC(urm_mod1, urm_mod2)
```

```
##          df      AIC
## urm_mod1  9 63731.76
## urm_mod2 10 63552.88
```

```r
# remove experiment1 * dem_id from the best model predicted 
urm_mod3 <- lmer(finalexam ~ experiment1 + urm_id + satmath + satverbal + mastered_topics_initial_kc +  high_sch_gpa + (1 | ta_sect), data = master_true, REML = FALSE)

AIC(fgn_mod2, fgn_mod3)
```

```
##          df      AIC
## fgn_mod2 10 63544.11
## fgn_mod3  9 63535.00
```

```r
#Refit model 3 with lmer and REML = FALSE
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
## urm_idURM                   1.662e+00  6.977e-01   2.382
## satmath                     1.471e-01  4.125e-03  35.663
## satverbal                   3.251e-02  4.128e-03   7.877
## mastered_topics_initial_kc  3.020e-01  1.441e-02  20.960
## high_sch_gpa                3.188e+01  1.265e+00  25.205
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE ur_URM satmth stvrbl mst___
## e1EXPERIMEN -0.103                                   
## urm_idURM   -0.259 -0.006                            
## satmath     -0.186  0.008  0.192                     
## satverbal   -0.120  0.005  0.062 -0.579              
## mstrd_tpc__  0.037  0.004 -0.055 -0.194 -0.018       
## high_sch_gp -0.868  0.000  0.118 -0.037 -0.100 -0.019
```
#### For each demographic identifier, the best fitting model is the one with random effects and without interactions (model 3). The AIC value decreases by adding random effects and removing interaction. 









