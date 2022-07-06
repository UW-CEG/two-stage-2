---
title: "Model_Analysis_New_Data"
author: "Ganling"
date: "8/30/2021"
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
copy_from <- paste0(original_data_dir, 
 "two_stage_master_wide_deid.rds")
copy_to <- paste0(importable_data_dir, "two_stage_master_wide_deid.rds")
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```
### Import master data set

```r
df <- readRDS(paste0(importable_data_dir, "two_stage_master_wide_deid.rds"))
```
## Model Selection
### Change `NAs` to experiment and select columns

```r
df_1 <- df %>%
  mutate(exp = replace_na(exp, "EXPERIMENT"))
df_true <- df_1 %>%
  select(two_stage_id, class.x, class.y, qtr, course_fullid, ta_sect, exp, exam1_c, exam2_c, final_c, satm_c, satv_c, aleksikc_c, hs_gpa_c, final_c, sex_id, urm_id, eop_id, fgn_id)
```
### Model selection on `sex_id`
#### Sex Model 1: fixed effect

```r
sex_mod1 <- lm(final_c ~ exp * sex_id + 
                 exp + sex_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
summary(sex_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * sex_id + exp + sex_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -61.581  -7.775   1.386   8.823  36.733 
## 
## Coefficients:
##                             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                 1.786594   0.639191   2.795  0.00524 ** 
## expEXPERIMENT              -0.838761   0.848729  -0.988  0.32315    
## sex_idFemale               -2.889390   0.877542  -3.293  0.00101 ** 
## satm_c                      0.099012   0.004970  19.923  < 2e-16 ***
## satv_c                      0.021155   0.005106   4.143 3.56e-05 ***
## aleksikc_c                  0.194925   0.018152  10.739  < 2e-16 ***
## hs_gpa_c                   22.726225   1.661504  13.678  < 2e-16 ***
## expEXPERIMENT:sex_idFemale  0.869752   1.150669   0.756  0.44982    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.83 on 2029 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4277,	Adjusted R-squared:  0.4257 
## F-statistic: 216.6 on 7 and 2029 DF,  p-value: < 2.2e-16
```
#### Sex model 2: add in random effect

```r
sex_mod2 <- lmer(final_c ~ exp * sex_id + 
                   exp + sex_id +
                   satm_c + satv_c + aleksikc_c + hs_gpa_c + 
                   (1 | ta_sect),
                 data = df_true, REML = TRUE)
summary(sex_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * sex_id + exp + sex_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: df_true
## 
## REML criterion at convergence: 16185.2
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.7506 -0.6042  0.1037  0.6946  2.8594 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.564   1.251  
##  Residual             163.051  12.769  
## Number of obs: 2037, groups:  ta_sect, 104
## 
## Fixed effects:
##                             Estimate Std. Error t value
## (Intercept)                 1.777626   0.663558   2.679
## expEXPERIMENT              -0.847830   0.883901  -0.959
## sex_idFemale               -2.883101   0.876921  -3.288
## satm_c                      0.098792   0.004968  19.885
## satv_c                      0.021152   0.005101   4.147
## aleksikc_c                  0.195079   0.018162  10.741
## hs_gpa_c                   22.637907   1.660729  13.631
## expEXPERIMENT:sex_idFemale  0.892834   1.150918   0.776
## 
## Correlation of Fixed Effects:
##              (Intr) exEXPERIMENT sx_dFm satm_c satv_c alksk_ hs_gp_
## exEXPERIMENT -0.736                                                
## sex_idFemal  -0.716  0.516                                         
## satm_c       -0.114 -0.010        0.159                            
## satv_c        0.073 -0.013       -0.103 -0.543                     
## aleksikc_c   -0.026 -0.001        0.035 -0.203 -0.005              
## hs_gpa_c      0.094 -0.031       -0.136 -0.101 -0.092 -0.016       
## eEXPERIMENT:  0.524 -0.709       -0.732  0.004  0.019  0.005  0.037
```

```r
AIC(sex_mod1, sex_mod2)
```

```
##          df      AIC
## sex_mod1  9 16186.44
## sex_mod2 10 16205.22
```
* Since the AIC value went up after adding random effect, we can ignore the random effect. 

#### Sex model 3: no interaction or random effect

```r
sex_mod3 <- lm(final_c ~ exp + sex_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(sex_mod1, sex_mod3)
```

```
##          df      AIC
## sex_mod1  9 16186.44
## sex_mod3  8 16185.02
```

```r
summary(sex_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + sex_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -61.780  -7.861   1.322   8.856  36.499 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    1.523236   0.535825   2.843  0.00452 ** 
## expEXPERIMENT -0.365043   0.572268  -0.638  0.52362    
## sex_idFemale  -2.403368   0.597123  -4.025 5.91e-05 ***
## satm_c         0.098995   0.004969  19.922  < 2e-16 ***
## satv_c         0.021081   0.005105   4.130 3.78e-05 ***
## aleksikc_c     0.194861   0.018150  10.736  < 2e-16 ***
## hs_gpa_c      22.677976   1.660102  13.661  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.83 on 2030 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4275,	Adjusted R-squared:  0.4258 
## F-statistic: 252.7 on 6 and 2030 DF,  p-value: < 2.2e-16
```
* The AIC value decreased for `sex_mod3`
* The interaction between `exp` and `sex_id` should not be included

#### Sex model 3.a: No `exp` 

```r
sex_mod3.a <- lm(final_c ~ sex_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(sex_mod3, sex_mod3.a)
```

```
##            df      AIC
## sex_mod3    8 16185.02
## sex_mod3.a  7 16183.43
```

```r
summary(sex_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -61.943  -7.841   1.273   8.871  36.332 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.321101   0.432018   3.058  0.00226 ** 
## sex_idFemale -2.405558   0.597025  -4.029 5.80e-05 ***
## satm_c        0.098964   0.004968  19.919  < 2e-16 ***
## satv_c        0.021085   0.005104   4.131 3.75e-05 ***
## aleksikc_c    0.194892   0.018147  10.740  < 2e-16 ***
## hs_gpa_c     22.669520   1.659807  13.658  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.83 on 2031 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4274,	Adjusted R-squared:  0.426 
## F-statistic: 303.2 on 5 and 2031 DF,  p-value: < 2.2e-16
```
* The AIC value slightly decreased
* Since sex_mod3.a is slightly smaller, we keep this model

#### Sex model 3.b: Remove `sex_id`

```r
sex_mod3.b <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.b)
```

```
##            df      AIC
## sex_mod3.a  7 16183.43
## sex_mod3.b  6 16197.64
```

```r
summary(sex_mod3.b)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + satv_c + aleksikc_c + hs_gpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.680  -7.919   1.392   8.811  37.160 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.010370   0.285346   0.036 0.971015    
## satm_c       0.103736   0.004843  21.420  < 2e-16 ***
## satv_c       0.018388   0.005079   3.621 0.000301 ***
## aleksikc_c   0.199022   0.018186  10.944  < 2e-16 ***
## hs_gpa_c    21.592688   1.644279  13.132  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.87 on 2032 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4228,	Adjusted R-squared:  0.4217 
## F-statistic: 372.1 on 4 and 2032 DF,  p-value: < 2.2e-16
```
* Since the AIC value increased, `sex_id` should be kept in the model

#### Sex model 3.c: Remove `satv_c`

```r
sex_mod3.c <- lm(final_c ~ sex_id + 
                 satm_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.c)
```

```
##            df      AIC
## sex_mod3.a  7 16183.43
## sex_mod3.c  6 16198.47
```

```r
summary(sex_mod3.c)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + aleksikc_c + hs_gpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -61.536  -7.934   1.170   9.016  34.885 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.148402   0.431688   2.660 0.007869 ** 
## sex_idFemale -2.082084   0.594204  -3.504 0.000468 ***
## satm_c        0.110115   0.004188  26.296  < 2e-16 ***
## aleksikc_c    0.195310   0.018218  10.720  < 2e-16 ***
## hs_gpa_c     23.302233   1.659247  14.044  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.88 on 2032 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4226,	Adjusted R-squared:  0.4214 
## F-statistic: 371.8 on 4 and 2032 DF,  p-value: < 2.2e-16
```
* The AIC value went up, so `satv_c` should be kept in our model 

#### Sex model 3.d: Remove `aleksikc_c`

```r
sex_mod3.d <- lm(final_c ~ sex_id + 
                 satm_c + satv_c + hs_gpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.d)
```

```
## Warning in AIC.default(sex_mod3.a, sex_mod3.d): models are not all fitted to the
## same number of observations
```

```
##            df      AIC
## sex_mod3.a  7 16183.43
## sex_mod3.d  6 16506.82
```

```r
summary(sex_mod3.d)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + satv_c + hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -54.930  -8.289   1.459   9.230  39.789 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.386235   0.440222   3.149  0.00166 ** 
## sex_idFemale -2.669989   0.609182  -4.383 1.23e-05 ***
## satm_c        0.110660   0.004962  22.302  < 2e-16 ***
## satv_c        0.021345   0.005209   4.098 4.33e-05 ***
## hs_gpa_c     23.052189   1.699232  13.566  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 13.2 on 2058 degrees of freedom
##   (124 observations deleted due to missingness)
## Multiple R-squared:  0.3989,	Adjusted R-squared:  0.3978 
## F-statistic: 341.5 on 4 and 2058 DF,  p-value: < 2.2e-16
```
* The AIC value increased, `aleksikc_c` should be kept

#### Sex Model 3.e: remove `hs_gpa_c` 

```r
sex_mod3.e <- lm(final_c ~ sex_id + 
                 satm_c + satv_c + aleksikc_c , 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.e)
```

```
## Warning in AIC.default(sex_mod3.a, sex_mod3.e): models are not all fitted to the
## same number of observations
```

```
##            df      AIC
## sex_mod3.a  7 16183.43
## sex_mod3.e  6 16583.44
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
## -66.306  -8.588   1.217   9.694  35.751 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   0.742974   0.445053   1.669   0.0952 .  
## sex_idFemale -1.162846   0.610977  -1.903   0.0571 .  
## satm_c        0.104776   0.005123  20.451  < 2e-16 ***
## satv_c        0.028214   0.005274   5.349 9.81e-08 ***
## aleksikc_c    0.198525   0.018815  10.551  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 13.39 on 2060 degrees of freedom
##   (122 observations deleted due to missingness)
## Multiple R-squared:  0.3769,	Adjusted R-squared:  0.3757 
## F-statistic: 311.5 on 4 and 2060 DF,  p-value: < 2.2e-16
```
* The AIC value went up, `hs_gpa_c` is kept

#### Sex Model 3.f: Remove `satm_c`

```r
sex_mod3.f <- lm(final_c ~ sex_id + 
                 satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(sex_mod3.a, sex_mod3.f)
```

```
##            df      AIC
## sex_mod3.a  7 16183.43
## sex_mod3.f  6 16544.93
```

```r
summary(sex_mod3.f)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satv_c + aleksikc_c + hs_gpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -67.990  -8.757   1.979   9.951  39.533 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   2.931205   0.463882   6.319 3.23e-10 ***
## sex_idFemale -5.240792   0.633767  -8.269 2.40e-16 ***
## satv_c        0.076318   0.004684  16.295  < 2e-16 ***
## aleksikc_c    0.268570   0.019419  13.830  < 2e-16 ***
## hs_gpa_c     26.009053   1.804987  14.410  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 14.02 on 2032 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.3155,	Adjusted R-squared:  0.3142 
## F-statistic: 234.2 on 4 and 2032 DF,  p-value: < 2.2e-16
```
* Again, the AIC value went up. `satm_c` should be kept
* `sex_mod3.a` is the best fitting model  

### Model selection on `eop_id`
#### Eop model 1: fixed effect

```r
eop_mod1 <- lm(final_c ~ exp * eop_id + 
                 exp + eop_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
summary(eop_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * eop_id + exp + eop_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.463  -7.837   1.445   8.732  37.601 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.080828   0.478885   0.169 0.865983    
## expEXPERIMENT           -0.362641   0.643119  -0.564 0.572900    
## eop_idEOP                0.761577   1.160628   0.656 0.511785    
## satm_c                   0.104809   0.005009  20.924  < 2e-16 ***
## satv_c                   0.018906   0.005120   3.693 0.000228 ***
## aleksikc_c               0.198655   0.018201  10.915  < 2e-16 ***
## hs_gpa_c                21.716733   1.652829  13.139  < 2e-16 ***
## expEXPERIMENT:eop_idEOP -0.218503   1.442285  -0.151 0.879598    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.88 on 2029 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4231,	Adjusted R-squared:  0.4211 
## F-statistic: 212.6 on 7 and 2029 DF,  p-value: < 2.2e-16
```
#### EOP model 2: random effect

```r
eop_mod2 <- lmer(final_c ~ exp * eop_id + 
                   exp + eop_id +
                   satm_c + satv_c + aleksikc_c + hs_gpa_c + 
                   (1 | ta_sect),
                 data = df_true, REML = TRUE)
summary(eop_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * eop_id + exp + eop_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: df_true
## 
## REML criterion at convergence: 16200.1
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6431 -0.6182  0.1052  0.6792  2.9155 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.682   1.297  
##  Residual             164.241  12.816  
## Number of obs: 2037, groups:  ta_sect, 104
## 
## Fixed effects:
##                          Estimate Std. Error t value
## (Intercept)              0.083679   0.513863   0.163
## expEXPERIMENT           -0.371323   0.692551  -0.536
## eop_idEOP                0.719151   1.160566   0.620
## satm_c                   0.104504   0.005007  20.872
## satv_c                   0.018917   0.005114   3.699
## aleksikc_c               0.198804   0.018212  10.916
## hs_gpa_c                21.644070   1.652540  13.097
## expEXPERIMENT:eop_idEOP -0.158893   1.444865  -0.110
## 
## Correlation of Fixed Effects:
##              (Intr) exEXPERIMENT ep_EOP satm_c satv_c alksk_ hs_gp_
## exEXPERIMENT -0.728                                                
## eop_idEOP    -0.415  0.276                                         
## satm_c       -0.076 -0.013        0.181                            
## satv_c       -0.043  0.005        0.100 -0.480                     
## aleksikc_c    0.011 -0.005       -0.029 -0.221 -0.001              
## hs_gpa_c     -0.020 -0.025        0.036 -0.041 -0.104 -0.008       
## eEXPERIMENT:  0.307 -0.415       -0.744 -0.017 -0.027  0.022  0.033
```

```r
AIC(eop_mod1, eop_mod2)
```

```
##          df      AIC
## eop_mod1  9 16202.52
## eop_mod2 10 16220.08
```
* AIC value went up, random effect should not be removed
* Interaction has the lowest t value, it should be removed next

#### Eop model 3: remove `exp` and `eop_id` interaction and no radnom effect 

```r
eop_mod3 <- lm(final_c ~ exp + eop_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(eop_mod1, eop_mod3)
```

```
##          df      AIC
## eop_mod1  9 16202.52
## eop_mod3  8 16200.54
```

```r
summary(eop_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + eop_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.444  -7.853   1.464   8.724  37.619 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    0.104784   0.451916   0.232 0.816665    
## expEXPERIMENT -0.406100   0.575458  -0.706 0.480456    
## eop_idEOP      0.630627   0.774358   0.814 0.415519    
## satm_c         0.104795   0.005007  20.930  < 2e-16 ***
## satv_c         0.018885   0.005117   3.691 0.000229 ***
## aleksikc_c     0.198709   0.018193  10.922  < 2e-16 ***
## hs_gpa_c      21.724850   1.651563  13.154  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.88 on 2030 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4231,	Adjusted R-squared:  0.4214 
## F-statistic: 248.2 on 6 and 2030 DF,  p-value: < 2.2e-16
```
* The AIC value decreased with the new model, meaning the interaction should be removed


#### eop model 3.a: remove `exp`

```r
eop_mod3.a <- lm(final_c ~ eop_id + satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(eop_mod3, eop_mod3.a)
```

```
##            df      AIC
## eop_mod3    8 16200.54
## eop_mod3.a  7 16199.04
```

```r
summary(eop_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ eop_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.627  -7.885   1.383   8.859  37.420 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -0.114637   0.327911  -0.350 0.726676    
## eop_idEOP    0.598199   0.772898   0.774 0.439040    
## satm_c       0.104713   0.005005  20.922  < 2e-16 ***
## satv_c       0.018861   0.005116   3.687 0.000233 ***
## aleksikc_c   0.198762   0.018191  10.927  < 2e-16 ***
## hs_gpa_c    21.708095   1.651188  13.147  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.88 on 2031 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.423,	Adjusted R-squared:  0.4216 
## F-statistic: 297.8 on 5 and 2031 DF,  p-value: < 2.2e-16
```
* The AIC value decreased, `exp` should be removed from the model 

#### eop model 3.b: remove `eop_id`

```r
eop_mod3.b <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(eop_mod3.a, eop_mod3.b)
```

```
##            df      AIC
## eop_mod3.a  7 16199.04
## eop_mod3.b  6 16197.64
```

```r
summary(eop_mod3.b)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + satv_c + aleksikc_c + hs_gpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.680  -7.919   1.392   8.811  37.160 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.010370   0.285346   0.036 0.971015    
## satm_c       0.103736   0.004843  21.420  < 2e-16 ***
## satv_c       0.018388   0.005079   3.621 0.000301 ***
## aleksikc_c   0.199022   0.018186  10.944  < 2e-16 ***
## hs_gpa_c    21.592688   1.644279  13.132  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.87 on 2032 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4228,	Adjusted R-squared:  0.4217 
## F-statistic: 372.1 on 4 and 2032 DF,  p-value: < 2.2e-16
```
* The AIC value decreased slightly, and the `eop_mod3.b` has the lower value. `eop_id` should be removed

#### eop model 3.c: remove `satv_c`

```r
eop_mod3.c <- lm(final_c ~ satm_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(eop_mod3.b, eop_mod3.c)
```

```
##            df      AIC
## eop_mod3.b  6 16197.64
## eop_mod3.c  5 16208.74
```

```r
summary(eop_mod3.c)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + aleksikc_c + hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -61.212  -8.018   1.338   9.014  35.777 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.013524   0.286194   0.047    0.962    
## satm_c       0.113062   0.004114  27.485   <2e-16 ***
## aleksikc_c   0.198900   0.018240  10.905   <2e-16 ***
## hs_gpa_c    22.282617   1.638057  13.603   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.91 on 2033 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4191,	Adjusted R-squared:  0.4182 
## F-statistic: 488.9 on 3 and 2033 DF,  p-value: < 2.2e-16
```
* The AIC value went up, `satv_c` should be kept. 
* `eop_mod3.b` is the best fitting model

### Model selection on `fgn_id`
#### Fgn model 1: fixed effect

```r
fgn_mod1 <- lm(final_c ~ exp * fgn_id + 
                 exp + fgn_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
summary(fgn_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * fgn_id + exp + fgn_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.863  -8.093   1.326   8.898  38.750 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.292012   0.513843   0.568   0.5699    
## expEXPERIMENT            0.206041   0.673644   0.306   0.7597    
## fgn_idFGN               -0.122226   0.983342  -0.124   0.9011    
## satm_c                   0.102301   0.004889  20.926   <2e-16 ***
## satv_c                   0.015690   0.005243   2.993   0.0028 ** 
## aleksikc_c               0.198938   0.018150  10.961   <2e-16 ***
## hs_gpa_c                21.428654   1.644060  13.034   <2e-16 ***
## expEXPERIMENT:fgn_idFGN -2.365079   1.288751  -1.835   0.0666 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.85 on 2027 degrees of freedom
##   (152 observations deleted due to missingness)
## Multiple R-squared:  0.4262,	Adjusted R-squared:  0.4242 
## F-statistic:   215 on 7 and 2027 DF,  p-value: < 2.2e-16
```
#### Fgn model 2: Random effect

```r
fgn_mod2 <- lmer(final_c ~ exp * fgn_id + 
                   exp + fgn_id +
                   satm_c + satv_c + aleksikc_c + hs_gpa_c + 
                   (1 | ta_sect),
                 data = df_true, REML = TRUE)
summary(fgn_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * fgn_id + exp + fgn_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: df_true
## 
## REML criterion at convergence: 16174.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6916 -0.6277  0.0935  0.6860  3.0113 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.386   1.177  
##  Residual             163.662  12.793  
## Number of obs: 2035, groups:  ta_sect, 104
## 
## Fixed effects:
##                          Estimate Std. Error t value
## (Intercept)              0.285975   0.540893   0.529
## expEXPERIMENT            0.194851   0.712622   0.273
## fgn_idFGN               -0.120061   0.983402  -0.122
## satm_c                   0.102098   0.004888  20.887
## satv_c                   0.015776   0.005238   3.012
## aleksikc_c               0.199105   0.018161  10.963
## hs_gpa_c                21.365190   1.643972  12.996
## expEXPERIMENT:fgn_idFGN -2.308732   1.288299  -1.792
## 
## Correlation of Fixed Effects:
##              (Intr) exEXPERIMENT fg_FGN satm_c satv_c alksk_ hs_gp_
## exEXPERIMENT -0.741                                                
## fgn_idFGN    -0.525  0.365                                         
## satm_c       -0.053 -0.009        0.100                            
## satv_c       -0.095  0.002        0.177 -0.470                     
## aleksikc_c    0.006 -0.002       -0.013 -0.222 -0.001              
## hs_gpa_c      0.003 -0.032       -0.012 -0.059 -0.103 -0.007       
## eEXPERIMENT:  0.365 -0.495       -0.697  0.011  0.010  0.009  0.051
```

```r
AIC(fgn_mod1,fgn_mod2)
```

```
##          df      AIC
## fgn_mod1  9 16175.94
## fgn_mod2 10 16194.49
```
* AIC value went up, random effect should not be included

#### Fgn model 3: remove `exp` and `fgn_id` interaction and no radnom effect 

```r
fgn_mod3 <- lm(final_c ~ exp + fgn_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(fgn_mod1, fgn_mod3)
```

```
##          df      AIC
## fgn_mod1  9 16175.94
## fgn_mod3  8 16177.32
```

```r
summary(fgn_mod3)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + fgn_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.605  -8.063   1.393   8.877  37.985 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    0.654221   0.474702   1.378  0.16830    
## expEXPERIMENT -0.441877   0.574048  -0.770  0.44153    
## fgn_idFGN     -1.378524   0.706336  -1.952  0.05112 .  
## satm_c         0.102405   0.004891  20.936  < 2e-16 ***
## satv_c         0.015781   0.005246   3.008  0.00266 ** 
## aleksikc_c     0.199218   0.018160  10.970  < 2e-16 ***
## hs_gpa_c      21.579154   1.642972  13.134  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.85 on 2028 degrees of freedom
##   (152 observations deleted due to missingness)
## Multiple R-squared:  0.4252,	Adjusted R-squared:  0.4235 
## F-statistic:   250 on 6 and 2028 DF,  p-value: < 2.2e-16
```
* The AIC value went up, the demographic `interaction` should be retained 

#### Fgn model 3.a: remove `fgn_id`

```r
fgn_mod3.a <- lm(final_c ~ exp + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(fgn_mod1, fgn_mod3.a)
```

```
## Warning in AIC.default(fgn_mod1, fgn_mod3.a): models are not all fitted to the
## same number of observations
```

```
##            df      AIC
## fgn_mod1    9 16175.94
## fgn_mod3.a  7 16199.21
```

```r
summary(fgn_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + satm_c + satv_c + aleksikc_c + hs_gpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.512  -7.833   1.384   8.749  37.332 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    0.221077   0.428726   0.516 0.606148    
## expEXPERIMENT -0.378290   0.574396  -0.659 0.510236    
## satm_c         0.103764   0.004844  21.422  < 2e-16 ***
## satv_c         0.018387   0.005079   3.620 0.000302 ***
## aleksikc_c     0.198985   0.018189  10.940  < 2e-16 ***
## hs_gpa_c      21.602467   1.644575  13.136  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.88 on 2031 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4229,	Adjusted R-squared:  0.4215 
## F-statistic: 297.7 on 5 and 2031 DF,  p-value: < 2.2e-16
```
*The AIC value went up, suggesting fgn_mod1 is the best fitting model 
### Model selection on `urm_id`
#### Urm model 1: fixed effect 

```r
urm_mod1 <- lm(final_c ~ exp * urm_id + 
                 exp + urm_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
summary(urm_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * urm_id + exp + urm_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.714  -8.082   1.299   8.872  37.943 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.300288   0.477996   0.628 0.529937    
## expEXPERIMENT           -0.771516   0.645744  -1.195 0.232331    
## urm_idURM                0.426985   1.392476   0.307 0.759154    
## satm_c                   0.101886   0.005637  18.073  < 2e-16 ***
## satv_c                   0.021988   0.005657   3.887 0.000105 ***
## aleksikc_c               0.205390   0.019644  10.456  < 2e-16 ***
## hs_gpa_c                21.855091   1.738056  12.574  < 2e-16 ***
## expEXPERIMENT:urm_idURM  0.878648   1.795410   0.489 0.624627    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.87 on 1837 degrees of freedom
##   (342 observations deleted due to missingness)
## Multiple R-squared:  0.4214,	Adjusted R-squared:  0.4191 
## F-statistic: 191.1 on 7 and 1837 DF,  p-value: < 2.2e-16
```
#### Urm model 2: random effect

```r
urm_mod2 <- lmer(final_c ~ exp * urm_id + 
                   exp + urm_id +
                   satm_c + satv_c + aleksikc_c + hs_gpa_c + 
                   (1 | ta_sect),
                 data = df_true, REML = TRUE)
summary(urm_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * urm_id + exp + urm_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: df_true
## 
## REML criterion at convergence: 14668.6
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6522 -0.6256  0.1076  0.6798  2.9474 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.582   1.258  
##  Residual             163.970  12.805  
## Number of obs: 1845, groups:  ta_sect, 104
## 
## Fixed effects:
##                          Estimate Std. Error t value
## (Intercept)              0.319452   0.511463   0.625
## expEXPERIMENT           -0.812343   0.692536  -1.173
## urm_idURM                0.319496   1.392516   0.229
## satm_c                   0.101478   0.005636  18.007
## satv_c                   0.021990   0.005650   3.892
## aleksikc_c               0.205832   0.019650  10.475
## hs_gpa_c                21.838451   1.737306  12.570
## expEXPERIMENT:urm_idURM  1.055523   1.798584   0.587
## 
## Correlation of Fixed Effects:
##              (Intr) exEXPERIMENT ur_URM satm_c satv_c alksk_ hs_gp_
## exEXPERIMENT -0.731                                                
## urm_idURM    -0.328  0.232                                         
## satm_c        0.018  0.004        0.144                            
## satv_c       -0.067 -0.017        0.032 -0.580                     
## aleksikc_c    0.018  0.014       -0.038 -0.200 -0.021              
## hs_gpa_c     -0.027 -0.019        0.040 -0.035 -0.100 -0.022       
## eEXPERIMENT:  0.244 -0.337       -0.737 -0.016  0.010 -0.002  0.055
```

```r
AIC(urm_mod1, urm_mod2)
```

```
##          df      AIC
## urm_mod1  9 14672.14
## urm_mod2 10 14688.57
```
* The AIC value went up, random effect should not be kept
* Next, `interaction` should be removed

#### Urm model 3: Remove random effects and interactions

```r
urm_mod3 <- lm(final_c ~ exp + urm_id + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(urm_mod1, urm_mod3)
```

```
##          df      AIC
## urm_mod1  9 14672.14
## urm_mod3  8 14670.38
```

```r
summary(urm_mod3) 
```

```
## 
## Call:
## lm(formula = final_c ~ exp + urm_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.765  -8.066   1.300   8.829  37.874 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    0.239061   0.461238   0.518 0.604309    
## expEXPERIMENT -0.657544   0.602161  -1.092 0.274988    
## urm_idURM      0.930382   0.938374   0.991 0.321580    
## satm_c         0.101933   0.005635  18.088  < 2e-16 ***
## satv_c         0.021961   0.005655   3.883 0.000107 ***
## aleksikc_c     0.205418   0.019640  10.459  < 2e-16 ***
## hs_gpa_c      21.809305   1.735177  12.569  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.86 on 1838 degrees of freedom
##   (342 observations deleted due to missingness)
## Multiple R-squared:  0.4213,	Adjusted R-squared:  0.4194 
## F-statistic:   223 on 6 and 1838 DF,  p-value: < 2.2e-16
```
* The AIC value went down, thus the interaction should be removed

#### urm model 3.a: Remove `urm_id`

```r
urm_mod3.a <- lm(final_c ~ exp + 
                 satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(urm_mod3, urm_mod3.a)
```

```
## Warning in AIC.default(urm_mod3, urm_mod3.a): models are not all fitted to the
## same number of observations
```

```
##            df      AIC
## urm_mod3    8 14670.38
## urm_mod3.a  7 16199.21
```

```r
summary(urm_mod3.a)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + satm_c + satv_c + aleksikc_c + hs_gpa_c, 
##     data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.512  -7.833   1.384   8.749  37.332 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    0.221077   0.428726   0.516 0.606148    
## expEXPERIMENT -0.378290   0.574396  -0.659 0.510236    
## satm_c         0.103764   0.004844  21.422  < 2e-16 ***
## satv_c         0.018387   0.005079   3.620 0.000302 ***
## aleksikc_c     0.198985   0.018189  10.940  < 2e-16 ***
## hs_gpa_c      21.602467   1.644575  13.136  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.88 on 2031 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4229,	Adjusted R-squared:  0.4215 
## F-statistic: 297.7 on 5 and 2031 DF,  p-value: < 2.2e-16
```
*The AIC value went up, `urm_id` should be retained

#### urm model 3.b: remove `exp`

```r
urm_mod3.b <- lm(final_c ~ urm_id +
                   satm_c + satv_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(urm_mod3, urm_mod3.b)
```

```
##            df      AIC
## urm_mod3    8 14670.38
## urm_mod3.b  7 14669.58
```

```r
summary(urm_mod3.b)
```

```
## 
## Call:
## lm(formula = final_c ~ urm_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = df_true)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -61.069  -8.144   1.290   8.940  37.558 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -0.117729   0.325565  -0.362 0.717681    
## urm_idURM    0.902257   0.938069   0.962 0.336264    
## satm_c       0.101933   0.005636  18.087  < 2e-16 ***
## satv_c       0.021861   0.005655   3.866 0.000115 ***
## aleksikc_c   0.205725   0.019639  10.475  < 2e-16 ***
## hs_gpa_c    21.808346   1.735268  12.568  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.86 on 1839 degrees of freedom
##   (342 observations deleted due to missingness)
## Multiple R-squared:  0.4209,	Adjusted R-squared:  0.4193 
## F-statistic: 267.3 on 5 and 1839 DF,  p-value: < 2.2e-16
```
* The AIC value decreased, so `exp` should be removed 
** The T value is larger for `exp` compared to `urm_id` but the AIC value decreased only when we removed `exp`

#### Urm model 3.c: Remove `satv_c`

```r
urm_mod3.c <- lm(final_c ~ urm_id +
                   satm_c + aleksikc_c + hs_gpa_c, 
               data = df_true)
AIC(urm_mod3.b, urm_mod3.c)
```

```
##            df      AIC
## urm_mod3.b  7 14669.58
## urm_mod3.c  6 14682.51
```
* The AIC value went up, `urm_mod3.b` is the best fitting model
















