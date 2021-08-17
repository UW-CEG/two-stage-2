---
title: 'Model Selection on Demographic Interactions'
author: "Jackson Hughes"
date: "8/16/2021"
output: 
  html_document:
    keep_md: yes
---



# Preliminary setup and data wrangling

## Data import

### Set working directories


```r
proj_dir <- here()
original_data_dir <- here("original-data", "/")
importable_data_dir <- here("processing-and-analysis", "01-importable-data", "/")
analysis_data_dir <- here("processing-and-analysis", "03-analysis-data", "/")
metadata_dir <- here("original-data", "metadata", "/")
```

### Copy data from `original-data` to `01-importable-data`


```r
copy_from <- paste0(original_data_dir, "master_2s_small_deid_scaled.rds")
copy_to <- paste0(importable_data_dir, "master_2s_small_deid_scaled.rds")
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```

### Import master dataset


```r
master <- readRDS(paste0(importable_data_dir, "master_2s_small_deid_scaled.rds"))
```

## Data wrangling

### Condense dataset to only include unique rows; omit NAs


```r
master_unique_1 <- master %>% 
  select(two_stage_id, exam1, exam2, finalexam, course_fullid, ta_sect, ver, sex_id, urm_id, eop_id, fgn_id, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc, experiment1, satm_c, satv_c, aleksikc_c, hsgpa_c, final_c, satm_z, satv_z, aleksikc_z, hsgpa_z, final_z)
master_unique_2 <- unique(master_unique_1)
master_unique <- na.omit(master_unique_2)
```

### Make `ta_sect` values unique across `course_fullid`


```r
master_true <- master_unique %>% 
  mutate(ta_sect = ifelse(str_detect(course_fullid, "2016"), paste(ta_sect, "16", sep = "_"), paste(ta_sect, "17", sep = "_")))
```

# Model Selection (based on consultation with Elli Theobald)

## Model selection on `sex_id` interactions

### Fixed-effects only (`sex_id`)


```r
sex_mod1 <- lm(final_c ~ experiment1*sex_id + 
                         experiment1 + sex_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(sex_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * sex_id + experiment1 + sex_id + 
##     satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
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

### Random-intercepts model with `REML = TRUE` (`sex_id`)


```r
sex_mod2 <- lmer(final_c ~ experiment1*sex_id + 
                           experiment1 + sex_id + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c + 
                           (1|ta_sect), 
                           data = master_true, REML = TRUE)
summary(sex_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * sex_id + experiment1 + sex_id + satm_c +  
##     satv_c + aleksikc_c + hsgpa_c + (1 | ta_sect)
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

### AIC comparison of `sex_mod1` and `sex_mod2`


```r
AIC(sex_mod1, sex_mod2)
```

```
##          df      AIC
## sex_mod1  9 18607.93
## sex_mod2 10 18616.32
```

* The AIC *increases* for `sex_mod2`
* Therefore the random effects will not be included

### Model with no `sex_id` interactions or random effects


```r
sex_mod3 <- lm(final_c ~ experiment1 + sex_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(sex_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + sex_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
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

### AIC comparision of `sex_mod1` and `sex_mod3` to see if we should include `sex_id` interactions


```r
AIC(sex_mod1, sex_mod3)
```

```
##          df      AIC
## sex_mod1  9 18607.93
## sex_mod3  8 18606.86
```

* The AIC *decreases* by *less than 2* for `sex_mod3`, so we should proceed with `sex_mod3`, which is the simplest model.

### Removal of parameters one by one


```r
summary(sex_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + sex_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
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

* `experiment1` has the lowest t-value, so let's try removing that first:


```r
sex_mod3.a <- lm(final_c ~ sex_id + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(sex_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_true)
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

```r
AIC(sex_mod3, sex_mod3.a)
```

```
##            df      AIC
## sex_mod3    8 18606.86
## sex_mod3.a  7 18606.07
```

* The AIC *decreases* by *less than 2* for `sex_mod3.a`, so we should keep the simplest model, which is `sex_mod3.a`.
* Let's now try removing `sex_id`.


```r
sex_mod3.b <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(sex_mod3.b)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
##     data = master_true)
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

```r
AIC(sex_mod3.a, sex_mod3.b)
```

```
##            df      AIC
## sex_mod3.a  7 18606.07
## sex_mod3.b  6 18626.30
```

* The AIC *increases* for `sex_mod3.b`, so we should use `sex_mod3.a` as our final model (summarized above). This means that we are retaining the `sex_id` parameter in our final model.

## Model selection on `eop_id` interactions

### Fixed-effects only (`eop_id`)


```r
eop_mod1 <- lm(final_c ~ experiment1*eop_id + 
                         experiment1 + eop_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(eop_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * eop_id + experiment1 + eop_id + 
##     satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.144 -12.218   2.052  13.125  55.655 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.154736   0.674516   0.229    0.819    
## experiment1EXPERIMENTAL           -0.982452   0.935021  -1.051    0.294    
## eop_idEOP                          1.248068   1.577810   0.791    0.429    
## satm_c                             0.147652   0.007710  19.152  < 2e-16 ***
## satv_c                             0.037367   0.007828   4.774 1.93e-06 ***
## aleksikc_c                         0.317201   0.027395  11.579  < 2e-16 ***
## hsgpa_c                           30.743824   2.360475  13.024  < 2e-16 ***
## experiment1EXPERIMENTAL:eop_idEOP -0.091892   1.991752  -0.046    0.963    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.99 on 2126 degrees of freedom
## Multiple R-squared:  0.4239,	Adjusted R-squared:  0.422 
## F-statistic: 223.5 on 7 and 2126 DF,  p-value: < 2.2e-16
```

### Random-intercepts model with `REML = TRUE` (`eop_id`)


```r
eop_mod2 <- lmer(final_c ~ experiment1*eop_id + 
                           experiment1 + eop_id + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c + 
                           (1|ta_sect), 
                           data = master_true, REML = TRUE)
summary(eop_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * eop_id + experiment1 + eop_id + satm_c +  
##     satv_c + aleksikc_c + hsgpa_c + (1 | ta_sect)
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
##                                    Estimate Std. Error t value
## (Intercept)                        0.211877   0.779195   0.272
## experiment1EXPERIMENTAL           -1.072257   1.077210  -0.995
## eop_idEOP                          1.045116   1.577096   0.663
## satm_c                             0.147195   0.007708  19.097
## satv_c                             0.036918   0.007814   4.724
## aleksikc_c                         0.318464   0.027444  11.604
## hsgpa_c                           30.756407   2.359456  13.035
## experiment1EXPERIMENTAL:eop_idEOP  0.277713   1.998735   0.139
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ep_EOP satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.711                                                   
## eop_idEOP       -0.393  0.256                                            
## satm_c          -0.007  0.004           0.172                            
## satv_c          -0.086 -0.019           0.111 -0.546                     
## aleksikc_c       0.024  0.007          -0.040 -0.192 -0.036              
## hsgpa_c         -0.017 -0.024           0.041 -0.028 -0.113 -0.023       
## e1EXPERIMENTAL:  0.289 -0.407          -0.724 -0.032 -0.015  0.018  0.036
```

### AIC comparison of `eop_mod1` and `eop_mod2`


```r
AIC(eop_mod1, eop_mod2)
```

```
##          df      AIC
## eop_mod1  9 18629.78
## eop_mod2 10 18636.63
```

* The AIC *increases* for `eop_mod2`, so we should not include random effects in our model.

### Model with no `eop_id` interactions or random effects


```r
eop_mod3 <- lm(final_c ~ experiment1 + eop_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(eop_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + eop_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.134 -12.213   2.044  13.134  55.664 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.165229   0.634870   0.260    0.795    
## experiment1EXPERIMENTAL -1.002677   0.825700  -1.214    0.225    
## eop_idEOP                1.195204   1.084434   1.102    0.271    
## satm_c                   0.147640   0.007704  19.165  < 2e-16 ***
## satv_c                   0.037361   0.007825   4.775 1.92e-06 ***
## aleksikc_c               0.317219   0.027386  11.583  < 2e-16 ***
## hsgpa_c                 30.747384   2.358660  13.036  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2127 degrees of freedom
## Multiple R-squared:  0.4239,	Adjusted R-squared:  0.4223 
## F-statistic: 260.9 on 6 and 2127 DF,  p-value: < 2.2e-16
```

### AIC comparison of `eop_mod1` and `eop_mod3`


```r
AIC(eop_mod1, eop_mod3)
```

```
##          df      AIC
## eop_mod1  9 18629.78
## eop_mod3  8 18627.79
```

* The AIC *decreases* by 1.99 for `eop_mod3`
* In this case we should choose the simplest model, which is `eop_mod3`.

### Removal of parameters one by one


```r
summary(eop_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + eop_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.134 -12.213   2.044  13.134  55.664 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.165229   0.634870   0.260    0.795    
## experiment1EXPERIMENTAL -1.002677   0.825700  -1.214    0.225    
## eop_idEOP                1.195204   1.084434   1.102    0.271    
## satm_c                   0.147640   0.007704  19.165  < 2e-16 ***
## satv_c                   0.037361   0.007825   4.775 1.92e-06 ***
## aleksikc_c               0.317219   0.027386  11.583  < 2e-16 ***
## hsgpa_c                 30.747384   2.358660  13.036  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2127 degrees of freedom
## Multiple R-squared:  0.4239,	Adjusted R-squared:  0.4223 
## F-statistic: 260.9 on 6 and 2127 DF,  p-value: < 2.2e-16
```

* `eop_id` has the lowest t-value, so let's try removing that first.


```r
eop_mod3.a <- lm(final_c ~ experiment1 + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(eop_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.284 -12.165   2.009  13.061  55.053 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.393484   0.600172   0.656    0.512    
## experiment1EXPERIMENTAL -0.936662   0.823567  -1.137    0.256    
## satm_c                   0.145807   0.007522  19.384  < 2e-16 ***
## satv_c                   0.036092   0.007740   4.663 3.31e-06 ***
## aleksikc_c               0.318332   0.027369  11.631  < 2e-16 ***
## hsgpa_c                 30.496329   2.347753  12.990  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2128 degrees of freedom
## Multiple R-squared:  0.4236,	Adjusted R-squared:  0.4222 
## F-statistic: 312.8 on 5 and 2128 DF,  p-value: < 2.2e-16
```

```r
AIC(eop_mod3, eop_mod3.a)
```

```
##            df      AIC
## eop_mod3    8 18627.79
## eop_mod3.a  7 18627.01
```

* The AIC *decreases* for `eop_mod3.a` by *less than 2*, so we should retain the simplest model, which is `eop_mod3.a`.
* The next lowest t-value is for `experiment1`, so let's try removing that.


```r
eop_mod3.b <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(eop_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.284 -12.165   2.009  13.061  55.053 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.393484   0.600172   0.656    0.512    
## experiment1EXPERIMENTAL -0.936662   0.823567  -1.137    0.256    
## satm_c                   0.145807   0.007522  19.384  < 2e-16 ***
## satv_c                   0.036092   0.007740   4.663 3.31e-06 ***
## aleksikc_c               0.318332   0.027369  11.631  < 2e-16 ***
## hsgpa_c                 30.496329   2.347753  12.990  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2128 degrees of freedom
## Multiple R-squared:  0.4236,	Adjusted R-squared:  0.4222 
## F-statistic: 312.8 on 5 and 2128 DF,  p-value: < 2.2e-16
```

```r
AIC(eop_mod3.a, eop_mod3.b)
```

```
##            df      AIC
## eop_mod3.a  7 18627.01
## eop_mod3.b  6 18626.30
```
* The AIC *decreases* by *less than 2* for `eop_mod3.b`, so we should retain the simplest model, which is `eop_mod3.b`.
* Let's try removing `satv_c`.


```r
eop_mod3.c <- lm(final_c ~ satm_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(eop_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.284 -12.165   2.009  13.061  55.053 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.393484   0.600172   0.656    0.512    
## experiment1EXPERIMENTAL -0.936662   0.823567  -1.137    0.256    
## satm_c                   0.145807   0.007522  19.384  < 2e-16 ***
## satv_c                   0.036092   0.007740   4.663 3.31e-06 ***
## aleksikc_c               0.318332   0.027369  11.631  < 2e-16 ***
## hsgpa_c                 30.496329   2.347753  12.990  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2128 degrees of freedom
## Multiple R-squared:  0.4236,	Adjusted R-squared:  0.4222 
## F-statistic: 312.8 on 5 and 2128 DF,  p-value: < 2.2e-16
```

```r
AIC(eop_mod3.b, eop_mod3.c)
```

```
##            df      AIC
## eop_mod3.b  6 18626.30
## eop_mod3.c  5 18645.73
```
* The AIC *increases* for `eop_mod3.c`, so we will retain the model `eop_mod3.b` as our final model, which is summarized above.

## Model selection on `fgn_id` interactions

### Fixed-effects only (`fgn_id`)


```r
fgn_mod1 <- lm(final_c ~ experiment1*fgn_id + 
                         experiment1 + fgn_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(fgn_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * fgn_id + experiment1 + fgn_id + 
##     satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.868 -12.137   2.061  13.355  56.957 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.357203   0.724784   0.493   0.6222    
## experiment1EXPERIMENTAL            0.066875   0.975523   0.069   0.9454    
## fgn_idFGN                          0.087061   1.364183   0.064   0.9491    
## satm_c                             0.143803   0.007584  18.961  < 2e-16 ***
## satv_c                             0.032165   0.008024   4.009 6.32e-05 ***
## aleksikc_c                         0.319680   0.027349  11.689  < 2e-16 ***
## hsgpa_c                           30.177375   2.348324  12.851  < 2e-16 ***
## experiment1EXPERIMENTAL:fgn_idFGN -3.599994   1.821160  -1.977   0.0482 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.96 on 2126 degrees of freedom
## Multiple R-squared:  0.4254,	Adjusted R-squared:  0.4235 
## F-statistic: 224.9 on 7 and 2126 DF,  p-value: < 2.2e-16
```

### Random-intercepts model with `REML = TRUE` (`fgn_id`)


```r
fgn_mod2 <- lmer(final_c ~ experiment1*fgn_id + 
                           experiment1 + fgn_id + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c + 
                           (1|ta_sect), 
                           data = master_true, REML = TRUE)
summary(fgn_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * fgn_id + experiment1 + fgn_id + satm_c +  
##     satv_c + aleksikc_c + hsgpa_c + (1 | ta_sect)
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
##                                    Estimate Std. Error t value
## (Intercept)                        0.396192   0.818222   0.484
## experiment1EXPERIMENTAL           -0.018109   1.103785  -0.016
## fgn_idFGN                          0.015903   1.367094   0.012
## satm_c                             0.143502   0.007585  18.920
## satv_c                             0.031948   0.008015   3.986
## aleksikc_c                         0.320732   0.027395  11.708
## hsgpa_c                           30.191401   2.347644  12.860
## experiment1EXPERIMENTAL:fgn_idFGN -3.324413   1.820962  -1.826
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL fg_FGN satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.720                                                   
## fgn_idFGN       -0.500  0.332                                            
## satm_c           0.013 -0.002           0.089                            
## satv_c          -0.132 -0.019           0.186 -0.535                     
## aleksikc_c       0.018  0.012          -0.019 -0.190 -0.036              
## hsgpa_c          0.003 -0.025          -0.009 -0.044 -0.115 -0.021       
## e1EXPERIMENTAL:  0.339 -0.475          -0.672  0.016  0.017  0.000  0.045
```

### AIC comparison of `fgn_mod1` and `fgn_mod2`


```r
AIC(fgn_mod1, fgn_mod2)
```

```
##          df     AIC
## fgn_mod1  9 18624.2
## fgn_mod2 10 18632.2
```

* The AIC *increases* for `fgn_mod2`. Therefore we will not include random effects in our model.

### Model with no `fgn_id` interactions or random effects


```r
fgn_mod3 <- lm(final_c ~ experiment1 + fgn_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(fgn_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + fgn_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.418 -12.135   2.157  13.147  55.760 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.902220   0.670752   1.345   0.1787    
## experiment1EXPERIMENTAL -0.968887   0.823423  -1.177   0.2395    
## fgn_idFGN               -1.718930   1.013761  -1.696   0.0901 .  
## satm_c                   0.144073   0.007588  18.987  < 2e-16 ***
## satv_c                   0.032459   0.008028   4.043 5.46e-05 ***
## aleksikc_c               0.319623   0.027367  11.679  < 2e-16 ***
## hsgpa_c                 30.371207   2.347880  12.936  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.97 on 2127 degrees of freedom
## Multiple R-squared:  0.4244,	Adjusted R-squared:  0.4227 
## F-statistic: 261.3 on 6 and 2127 DF,  p-value: < 2.2e-16
```

### AIC comparison of `fgn_mod2` and `fgn_mod3`


```r
AIC(fgn_mod1, fgn_mod3)
```

```
##          df      AIC
## fgn_mod1  9 18624.20
## fgn_mod3  8 18626.12
```

* The AIC *increases* for `fgn_mod3` by *less than 2*, so we will retain the least complex model, which is `fgn_mod3`.

### Removing parameters one by one


```r
summary(fgn_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + fgn_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.418 -12.135   2.157  13.147  55.760 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.902220   0.670752   1.345   0.1787    
## experiment1EXPERIMENTAL -0.968887   0.823423  -1.177   0.2395    
## fgn_idFGN               -1.718930   1.013761  -1.696   0.0901 .  
## satm_c                   0.144073   0.007588  18.987  < 2e-16 ***
## satv_c                   0.032459   0.008028   4.043 5.46e-05 ***
## aleksikc_c               0.319623   0.027367  11.679  < 2e-16 ***
## hsgpa_c                 30.371207   2.347880  12.936  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.97 on 2127 degrees of freedom
## Multiple R-squared:  0.4244,	Adjusted R-squared:  0.4227 
## F-statistic: 261.3 on 6 and 2127 DF,  p-value: < 2.2e-16
```

* `experiment1` has the lowest t-value, so let's remove that first.


```r
fgn_mod3.a <- lm(final_c ~ fgn_id + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(fgn_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ fgn_id + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.874 -12.166   2.057  13.274  55.255 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.384006   0.505957   0.759   0.4480    
## fgn_idFGN   -1.691398   1.013582  -1.669   0.0953 .  
## satm_c       0.144173   0.007588  19.000  < 2e-16 ***
## satv_c       0.032285   0.008027   4.022 5.98e-05 ***
## aleksikc_c   0.320092   0.027367  11.696  < 2e-16 ***
## hsgpa_c     30.354446   2.348048  12.928  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2128 degrees of freedom
## Multiple R-squared:  0.424,	Adjusted R-squared:  0.4226 
## F-statistic: 313.3 on 5 and 2128 DF,  p-value: < 2.2e-16
```

```r
AIC(fgn_mod3, fgn_mod3.a)
```

```
##            df      AIC
## fgn_mod3    8 18626.12
## fgn_mod3.a  7 18625.51
```

* The AIC *decreases* by *less than 2* for `fgn_mod3.a`, so we should retain the simplest model, which is `fgn_mod3.a`.
* Let's now try removing `fgn_id`


```r
fgn_mod3.b <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(fgn_mod3.b)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
##     data = master_true)
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

```r
AIC(fgn_mod3.a, fgn_mod3.b)
```

```
##            df      AIC
## fgn_mod3.a  7 18625.51
## fgn_mod3.b  6 18626.30
```

* The AIC *increased* by *less than 2* for `fgn_mod3.b`, so we should retain the simplest model, which is `fgn_mod3.b`.
* Let's now try removing `satv_c`.


```r
fgn_mod3.c <- lm(final_c ~ satm_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(fgn_mod3.c)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -87.617 -12.297   2.092  13.674  52.278 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.100270   0.414518   0.242    0.809    
## satm_c       0.166767   0.006052  27.558   <2e-16 ***
## aleksikc_c   0.322489   0.027487  11.732   <2e-16 ***
## hsgpa_c     31.869250   2.339770  13.621   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 19.08 on 2130 degrees of freedom
## Multiple R-squared:  0.4174,	Adjusted R-squared:  0.4166 
## F-statistic: 508.7 on 3 and 2130 DF,  p-value: < 2.2e-16
```

```r
AIC(fgn_mod3.b, fgn_mod3.c)
```

```
##            df      AIC
## fgn_mod3.b  6 18626.30
## fgn_mod3.c  5 18645.73
```

* The AIC *increases* for `fgn_mod3.c`, so our final model will be `fgn_mod3.b`, which is summarized above.

## Model selection on `urm_id` interactions

### Fixed-effects only (`urm_id`)


```r
urm_mod1 <- lm(final_c ~ experiment1*urm_id + 
                         experiment1 + urm_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(urm_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * urm_id + experiment1 + urm_id + 
##     satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.083 -12.222   2.064  13.146  55.564 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.270531   0.639985   0.423    0.673    
## experiment1EXPERIMENTAL           -1.061371   0.883669  -1.201    0.230    
## urm_idURM                          1.088201   1.875071   0.580    0.562    
## satm_c                             0.147589   0.007694  19.182  < 2e-16 ***
## satv_c                             0.036569   0.007752   4.718 2.54e-06 ***
## aleksikc_c                         0.316515   0.027419  11.544  < 2e-16 ***
## hsgpa_c                           30.826756   2.365605  13.031  < 2e-16 ***
## experiment1EXPERIMENTAL:urm_idURM  0.733726   2.459212   0.298    0.765    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.99 on 2126 degrees of freedom
## Multiple R-squared:  0.424,	Adjusted R-squared:  0.4221 
## F-statistic: 223.5 on 7 and 2126 DF,  p-value: < 2.2e-16
```

### Random-intercepts model with `REML = TRUE` (`urm_id`)


```r
urm_mod2 <- lmer(final_c ~ experiment1*urm_id + 
                           experiment1 + urm_id + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c + 
                           (1|ta_sect), 
                           data = master_true, REML = TRUE)
summary(urm_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * urm_id + experiment1 + urm_id + satm_c +  
##     satv_c + aleksikc_c + hsgpa_c + (1 | ta_sect)
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
##                                    Estimate Std. Error t value
## (Intercept)                        0.334426   0.751862   0.445
## experiment1EXPERIMENTAL           -1.159838   1.035365  -1.120
## urm_idURM                          0.691720   1.873335   0.369
## satm_c                             0.147076   0.007690  19.126
## satv_c                             0.036199   0.007741   4.676
## aleksikc_c                         0.317822   0.027465  11.572
## hsgpa_c                           30.864730   2.364551  13.053
## experiment1EXPERIMENTAL:urm_idURM  1.441549   2.465417   0.585
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ur_URM satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.722                                                   
## urm_idURM       -0.294  0.205                                            
## satm_c           0.015  0.007           0.163                            
## satv_c          -0.051 -0.024           0.022 -0.574                     
## aleksikc_c       0.022  0.013          -0.045 -0.196 -0.033              
## hsgpa_c         -0.009 -0.027           0.025 -0.028 -0.121 -0.026       
## e1EXPERIMENTAL:  0.218 -0.308          -0.723 -0.028  0.018  0.005  0.065
```

### AIC comparison of `urm_mod1` and `urm_mod2`


```r
AIC(urm_mod1, urm_mod2)
```

```
##          df      AIC
## urm_mod1  9 18629.57
## urm_mod2 10 18635.45
```

* The AIC *increases* for `urm_mod2`. So we will proceed with the model without random effects.

### Model with no `urm_id` interactions or random effects


```r
urm_mod3 <- lm(final_c ~ experiment1 + urm_id + 
                         satm_c + satv_c + aleksikc_c + hsgpa_c, 
                         data = master_true)
summary(urm_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + urm_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.125 -12.216   2.011  13.112  55.503 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.221373   0.618280   0.358    0.720    
## experiment1EXPERIMENTAL -0.966189   0.823896  -1.173    0.241    
## urm_idURM                1.493858   1.290962   1.157    0.247    
## satm_c                   0.147655   0.007689  19.203  < 2e-16 ***
## satv_c                   0.036531   0.007749   4.714 2.58e-06 ***
## aleksikc_c               0.316486   0.027413  11.545  < 2e-16 ***
## hsgpa_c                 30.783457   2.360643  13.040  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2127 degrees of freedom
## Multiple R-squared:  0.424,	Adjusted R-squared:  0.4223 
## F-statistic: 260.9 on 6 and 2127 DF,  p-value: < 2.2e-16
```

### AIC comparison of `urm_mod1` and `urm_mod3`


```r
AIC(urm_mod1, urm_mod3)
```

```
##          df      AIC
## urm_mod1  9 18629.57
## urm_mod3  8 18627.66
```

* The AIC *decreases* by *less than 2* for `urm_mod3`, which indicates that we should retain the simplest model, which is `urm_mod3`. This indicates that we are not keeping interactions from `urm_id` in our model.

### Removal of parameters one by one


```r
summary(urm_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + urm_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.125 -12.216   2.011  13.112  55.503 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.221373   0.618280   0.358    0.720    
## experiment1EXPERIMENTAL -0.966189   0.823896  -1.173    0.241    
## urm_idURM                1.493858   1.290962   1.157    0.247    
## satm_c                   0.147655   0.007689  19.203  < 2e-16 ***
## satv_c                   0.036531   0.007749   4.714 2.58e-06 ***
## aleksikc_c               0.316486   0.027413  11.545  < 2e-16 ***
## hsgpa_c                 30.783457   2.360643  13.040  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2127 degrees of freedom
## Multiple R-squared:  0.424,	Adjusted R-squared:  0.4223 
## F-statistic: 260.9 on 6 and 2127 DF,  p-value: < 2.2e-16
```

* `urm_id` has the lowest t-value, so we'll try removing that parameter first.


```r
urm_mod3.a <- lm(final_c ~ experiment1 + 
                           satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(urm_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -89.284 -12.165   2.009  13.061  55.053 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.393484   0.600172   0.656    0.512    
## experiment1EXPERIMENTAL -0.936662   0.823567  -1.137    0.256    
## satm_c                   0.145807   0.007522  19.384  < 2e-16 ***
## satv_c                   0.036092   0.007740   4.663 3.31e-06 ***
## aleksikc_c               0.318332   0.027369  11.631  < 2e-16 ***
## hsgpa_c                 30.496329   2.347753  12.990  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.98 on 2128 degrees of freedom
## Multiple R-squared:  0.4236,	Adjusted R-squared:  0.4222 
## F-statistic: 312.8 on 5 and 2128 DF,  p-value: < 2.2e-16
```

```r
AIC(urm_mod3, urm_mod3.a)
```

```
##            df      AIC
## urm_mod3    8 18627.66
## urm_mod3.a  7 18627.01
```

* The AIC *decreases* by *less than 2* for `urm_mod3.a`, so we should retain the simplest model, which is `urm_mod3.a`.
* Now let's try removing `experiment1` from the model.


```r
urm_mod3.b <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(urm_mod3.b)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + satv_c + aleksikc_c + hsgpa_c, 
##     data = master_true)
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

```r
AIC(urm_mod3.a, urm_mod3.b)
```

```
##            df      AIC
## urm_mod3.a  7 18627.01
## urm_mod3.b  6 18626.30
```

* The AIC *decreases* by *less than 2* for `urm_mod3.b`, so we should keep the simplest model, which is `urm_mod3.b`.
* Now let's try removing `satv_c`.


```r
urm_mod3.c <- lm(final_c ~ satm_c + aleksikc_c + hsgpa_c, 
                           data = master_true)
summary(urm_mod3.c)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + aleksikc_c + hsgpa_c, data = master_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -87.617 -12.297   2.092  13.674  52.278 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.100270   0.414518   0.242    0.809    
## satm_c       0.166767   0.006052  27.558   <2e-16 ***
## aleksikc_c   0.322489   0.027487  11.732   <2e-16 ***
## hsgpa_c     31.869250   2.339770  13.621   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 19.08 on 2130 degrees of freedom
## Multiple R-squared:  0.4174,	Adjusted R-squared:  0.4166 
## F-statistic: 508.7 on 3 and 2130 DF,  p-value: < 2.2e-16
```

```r
AIC(urm_mod3.b, urm_mod3.c)
```

```
##            df      AIC
## urm_mod3.b  6 18626.30
## urm_mod3.c  5 18645.73
```

* The AIC *increases* for `urm_mod3.c`, so we should keep `urm_mod3.b` as our final model, which is summarized above.

## Notes
* It actually appears that no interactions are retained for any of the models. This is because when I originally ran the third models for `sex_id` and `urm_id` last week, I accidentally omitted `hsgpa_c`. This drove up the AIC, which gave the illusion that getting rid of the demographic interactions created models that fit the data worse.
