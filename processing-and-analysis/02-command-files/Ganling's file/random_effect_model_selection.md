---
title: "random_effect_model_selection"
author: "Ganling"
date: "8/15/2021"
output: 
  html_document:
    keep_md: yes
---


## Set working directories

```r
original_data_dir   <- here("original-data", "/")
importable_data_dir <- here("processing-and-analysis", "01-importable-data", "/")
analysis_data_dir   <- here("processing-and-analysis", "03-analysis-data", "/")
```

## Import data set
### Copy data from `original-data` to `01-importable-data`

```r
copy_from <- paste0(original_data_dir, "master_2s_small_deid_scaled.rds")
copy_to <- paste0(importable_data_dir, "master_2s_small_deid_scaled.rds")
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```
### Import master data set

```r
df <- readRDS(paste0(importable_data_dir, "master_2s_small_deid_scaled.rds"))
```

## Data wrangling 
### Creating unique rows and omit NA's

```r
df_removed <- subset(df, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
df_unique_1 <- df_removed %>% 
select(two_stage_id, exam1, exam2, finalexam, course_fullid, ta_sect, ver, sex_id, urm_id, eop_id, fgn_id, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc, experiment1, satm_c, satv_c, aleksikc_c, hsgpa_c, final_c, satm_z, satv_z, aleksikc_z, hsgpa_z, final_z)
df_unique_2 <- unique(df_unique_1)
df_unique <- na.omit(df_unique_2)
```
### Make `ta_sect`  unique

```r
df_true <- df_unique %>% 
  mutate(ta_sect = ifelse(str_detect(course_fullid, "2016"), paste(ta_sect, "16", sep = "_"), paste(ta_sect, "17", sep = "_")))
```
## Model selection 
### Model selection on `sex_id`
#### Sex Model 1: fixed effect

```r
sex_mod1 <- lm(final_c ~ experiment1 * sex_id + 
                 experiment1 + sex_id + 
                 satm_c + satv_c + aleksikc_c + hsgpa_c, 
               data = df_true)
summary(sex_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * sex_id + experiment1 + sex_id + 
##     satm_c + satv_c + aleksikc_c + hsgpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -91.108 -12.022   1.739  13.031  54.166 
## 
## Coefficients:
##                                       Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                           3.004037   0.889780   3.376 0.000748 ***
## experiment1EXPERIMENTAL              -1.781175   1.225909  -1.453 0.146388    
## sex_idFemale                         -4.903694   1.226516  -3.998 6.60e-05 ***
## satm_c                                0.136511   0.007744  17.627  < 2e-16 ***
## satv_c                                0.042035   0.007801   5.388 7.89e-08 ***
## aleksikc_c                            0.312862   0.027259  11.477  < 2e-16 ***
## hsgpa_c                              32.280639   2.365641  13.646  < 2e-16 ***
## experiment1EXPERIMENTAL:sex_idFemale  1.596556   1.650592   0.967 0.333523    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.89 on 2126 degrees of freedom
## Multiple R-squared:  0.4298,	Adjusted R-squared:  0.4279 
## F-statistic: 228.9 on 7 and 2126 DF,  p-value: < 2.2e-16
```
#### Sex model 2: random effect 

```r
sex_mod2 <- lmer(final_c ~ experiment1 * sex_id + 
                   experiment1 + sex_id +
                   satm_c + satv_c + aleksikc_c + hsgpa_c + 
                   (1 | ta_sect),
                 data = df_true, REML = TRUE)
summary(sex_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * sex_id + experiment1 + sex_id + satm_c +  
##     satv_c + aleksikc_c + hsgpa_c + (1 | ta_sect)
##    Data: df_true
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
##                                       Estimate Std. Error t value
## (Intercept)                           2.981059   0.965244   3.088
## experiment1EXPERIMENTAL              -1.784357   1.328872  -1.343
## sex_idFemale                         -4.849198   1.226551  -3.954
## satm_c                                0.136279   0.007741  17.604
## satv_c                                0.041517   0.007791   5.329
## aleksikc_c                            0.314237   0.027304  11.509
## hsgpa_c                              32.197024   2.362880  13.626
## experiment1EXPERIMENTAL:sex_idFemale  1.598991   1.652722   0.967
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL sx_dFm satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.721                                                   
## sex_idFemal     -0.681  0.482                                            
## satm_c          -0.070 -0.003           0.175                            
## satv_c           0.051 -0.029          -0.127 -0.611                     
## aleksikc_c      -0.013  0.009           0.029 -0.171 -0.037              
## hsgpa_c          0.087 -0.029          -0.131 -0.084 -0.101 -0.026       
## e1EXPERIMENTAL:  0.491 -0.684          -0.711  0.006  0.024  0.000  0.039
```

```r
AIC(sex_mod1, sex_mod2)
```

```
##          df      AIC
## sex_mod1  9 18607.93
## sex_mod2 10 18616.32
```
* Since the AIC value went up after adding random effect, we can ignore the random effect.  

#### Sex model 3: no interaction or random effect

```r
sex_mod3 <- lm(final_c ~ experiment1 + sex_id + 
                 satm_c + satv_c + aleksikc_c + hsgpa_c, 
               data = df_true)
AIC(sex_mod1, sex_mod3)
```

```
##          df      AIC
## sex_mod1  9 18607.93
## sex_mod3  8 18606.86
```

```r
summary(sex_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + sex_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -91.508 -12.144   1.755  13.216  53.693 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              2.543555   0.751699   3.384 0.000728 ***
## experiment1EXPERIMENTAL -0.899323   0.819536  -1.097 0.272610    
## sex_idFemale            -4.059707   0.861958  -4.710 2.64e-06 ***
## satm_c                   0.136451   0.007744  17.620  < 2e-16 ***
## satv_c                   0.041862   0.007799   5.368 8.84e-08 ***
## aleksikc_c               0.312845   0.027258  11.477  < 2e-16 ***
## hsgpa_c                 32.182409   2.363424  13.617  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.89 on 2127 degrees of freedom
## Multiple R-squared:  0.4295,	Adjusted R-squared:  0.4279 
## F-statistic: 266.9 on 6 and 2127 DF,  p-value: < 2.2e-16
```
* The AIC value decreased for `sex_mod3`
* The interaction between `experiemnt1` and `sex_id` should not be included

#### Sex model 3.a: No `experiment1` 

```r
sex_mod3.a <- lm(final_c ~ sex_id + 
                 satm_c + satv_c + aleksikc_c + hsgpa_c, 
               data = df_true)
AIC(sex_mod3, sex_mod3.a)
```

```
##            df      AIC
## sex_mod3    8 18606.86
## sex_mod3.a  7 18606.07
```

```r
summary(sex_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -91.939 -11.946   1.852  13.219  53.231 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   2.074749   0.618527   3.354  0.00081 ***
## sex_idFemale -4.068857   0.861959  -4.720 2.51e-06 ***
## satm_c        0.136497   0.007744  17.625  < 2e-16 ***
## satv_c        0.041659   0.007797   5.343 1.01e-07 ***
## aleksikc_c    0.313288   0.027257  11.494  < 2e-16 ***
## hsgpa_c      32.168783   2.363505  13.611  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.89 on 2128 degrees of freedom
## Multiple R-squared:  0.4292,	Adjusted R-squared:  0.4279 
## F-statistic:   320 on 5 and 2128 DF,  p-value: < 2.2e-16
```
* The AIC value slightly decreased...
* Since sex_mod3.a is slightly smaller, we keep this model

#### Sex model 3.b: Remove `sex_id`

```r
sex_mod3.b <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.b)
```

```
##            df      AIC
## sex_mod3.a  7 18606.07
## sex_mod3.b  6 18626.30
```

```r
summary(sex_mod3.b)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.727 -12.063   2.107  13.116  54.575 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -0.099879   0.414793  -0.241     0.81    
## satm_c       0.145876   0.007522  19.392  < 2e-16 ***
## satv_c       0.035867   0.007738   4.635 3.78e-06 ***
## aleksikc_c   0.318807   0.027367  11.649  < 2e-16 ***
## hsgpa_c     30.478179   2.347861  12.981  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2129 degrees of freedom
## Multiple R-squared:  0.4232,	Adjusted R-squared:  0.4222 
## F-statistic: 390.6 on 4 and 2129 DF,  p-value: < 2.2e-16
```
* Since the AIC value increased, `sex_id` should be kept in the model

#### Sex model 3.c: Remove `satv_c`

```r
sex_mod3.c <- lm(final_c ~ sex_id + 
                 satm_c + aleksikc_c + hsgpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.c)
```

```
##            df      AIC
## sex_mod3.a  7 18606.07
## sex_mod3.c  6 18632.51
```

```r
summary(sex_mod3.c)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + aleksikc_c + hsgpa_c, 
##     data = df_true)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -89.15 -12.11   1.77  13.55  50.53 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.914127   0.621780   3.078  0.00211 ** 
## sex_idFemale -3.344132   0.856709  -3.903 9.78e-05 ***
## satm_c        0.161831   0.006163  26.260  < 2e-16 ***
## aleksikc_c    0.318442   0.027415  11.615  < 2e-16 ***
## hsgpa_c      33.443352   2.366600  14.131  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 19.01 on 2129 degrees of freedom
## Multiple R-squared:  0.4216,	Adjusted R-squared:  0.4205 
## F-statistic: 387.9 on 4 and 2129 DF,  p-value: < 2.2e-16
```
* The AIC value went up, so `satv_c` should be kept in our model 

#### Sex model 3.d: Remove `aleksikc_c`

```r
sex_mod3.d <- lm(final_c ~ sex_id + 
                 satm_c + satv_c + hsgpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.d)
```

```
##            df      AIC
## sex_mod3.a  7 18606.07
## sex_mod3.d  6 18732.61
```

```r
summary(sex_mod3.d)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + satv_c + hsgpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -80.777 -12.422   2.159  13.460  58.985 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   2.143136   0.637259   3.363 0.000784 ***
## sex_idFemale -4.493814   0.887287  -5.065 4.44e-07 ***
## satm_c        0.151885   0.007859  19.326  < 2e-16 ***
## satv_c        0.044831   0.008028   5.584 2.65e-08 ***
## hsgpa_c      32.871024   2.434380  13.503  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 19.46 on 2129 degrees of freedom
## Multiple R-squared:  0.3938,	Adjusted R-squared:  0.3926 
## F-statistic: 345.7 on 4 and 2129 DF,  p-value: < 2.2e-16
```
* The AIC value went up, `alekikc_c` is kept

#### Sex Model 3.e: remove `hsgpa_c`

```r
sex_mod3.e <- lm(final_c ~ sex_id + 
                 satm_c + satv_c + aleksikc_c , 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.e)
```

```
##            df      AIC
## sex_mod3.a  7 18606.07
## sex_mod3.e  6 18782.20
```

```r
summary(sex_mod3.e)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + satv_c + aleksikc_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -91.611 -12.813   1.731  14.363  53.161 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.178943   0.641076   1.839  0.06605 .  
## sex_idFemale -2.291131   0.888109  -2.580  0.00995 ** 
## satm_c        0.145575   0.008043  18.101  < 2e-16 ***
## satv_c        0.052370   0.008086   6.477 1.16e-10 ***
## aleksikc_c    0.322878   0.028402  11.368  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 19.69 on 2129 degrees of freedom
## Multiple R-squared:  0.3795,	Adjusted R-squared:  0.3784 
## F-statistic: 325.6 on 4 and 2129 DF,  p-value: < 2.2e-16
```
* The AIC value went up, `hsgpa_c` is kept

#### Sex Model 3.f: Remove `satm_c`

```r
sex_mod3.f <- lm(final_c ~ sex_id + 
                 satv_c + aleksikc_c + hsgpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.f)
```

```
##            df      AIC
## sex_mod3.a  7 18606.07
## sex_mod3.f  6 18894.86
```

```r
summary(sex_mod3.f)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satv_c + aleksikc_c + hsgpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -99.299 -12.670   2.299  14.625  55.028 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   3.322915   0.657628   5.053 4.72e-07 ***
## sex_idFemale -7.966903   0.891632  -8.935  < 2e-16 ***
## satv_c        0.125799   0.006598  19.067  < 2e-16 ***
## aleksikc_c    0.396340   0.028732  13.794  < 2e-16 ***
## hsgpa_c      35.756817   2.520151  14.188  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 20.22 on 2129 degrees of freedom
## Multiple R-squared:  0.3459,	Adjusted R-squared:  0.3447 
## F-statistic: 281.5 on 4 and 2129 DF,  p-value: < 2.2e-16
```
* Again, the AIC value went up. `satm_c` should be kept
* `sex_mod3.a` is the best fitting model  





