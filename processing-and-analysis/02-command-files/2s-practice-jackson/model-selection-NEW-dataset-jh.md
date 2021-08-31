---
title: 'Model Selection with the New Dataset'
author: "Jackson Hughes"
date: "8/30/2021"
output: 
  html_document:
    keep_md: yes
---



## Set up

### Designate working directories and copy file


```r
proj_dir <- here()
original_data_dir   <- here("original-data", "/")
importable_data_dir <- here("processing-and-analysis", "01-importable-data", "/")
analysis_data_dir   <- here("processing-and-analysis", "03-analysis-data", "/")
metadata_dir <- here("original-data", "metadata", "/")

copy_from <- paste0(original_data_dir, "two_stage_master_wide_deid.rds")
copy_to <- paste0(importable_data_dir, "two_stage_master_wide_deid.rds")
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```

### Import dataset


```r
master_original_1 <- readRDS(paste0(importable_data_dir, "two_stage_master_wide_deid.rds"))
```

### Change value of NAs in `exp` column to `EXPERIMENT`


```r
master_original_2 <- master_original_1 %>% 
  mutate(exp = replace_na(exp, "EXPERIMENT"))
```


### Select relevant columns


```r
master <- master_original_2 %>% 
  select(two_stage_id, course_fullid, exp, exam1, exam2, ta_sect, sex_id, urm_id, eop_id, fgn_id, satm_c, satv_c, aleksikc_c, hs_gpa_c, final_c, satm_z, satv_z, aleksikc_z, hs_gpa_z, final_z)
```

# Model Selection

## Model selection for `sex_id`

### 1. Fixed-effects only model (sex)


```r
sex_mod1 <- lm(final_c ~ exp * sex_id +
                         exp + sex_id +
                         satm_c + satv_c + aleksikc_c + hs_gpa_c,
                         data = master)
summary(sex_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * sex_id + exp + sex_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = master)
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

### 2. Random intercepts model with `ta_sect`; REML = TRUE (sex)


```r
sex_mod2 <- lmer(final_c ~ exp * sex_id +
                           exp + sex_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c +
                         (1 | ta_sect),
                         data = master, REML = TRUE)
summary(sex_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * sex_id + exp + sex_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: master
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

#### AIC comparison of `sex_mod1` and `sex_mod2`


```r
AIC(sex_mod1, sex_mod2)
```

```
##          df      AIC
## sex_mod1  9 16186.44
## sex_mod2 10 16205.22
```

* The AIC *increases* by more than 2 for `sex_mod2`, so random effects are not retained in the model.

### 3. Fixed-effects only model with *no* demographic interactions (sex)


```r
sex_mod1.a <- lm(final_c ~ exp + sex_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(sex_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * sex_id + exp + sex_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = master)
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

#### AIC comparison of `sex_mod1` and `sex_mod1.a`


```r
AIC(sex_mod1, sex_mod1.a)
```

```
##            df      AIC
## sex_mod1    9 16186.44
## sex_mod1.a  8 16185.02
```

* The AIC *decreases* by *less than 2* for `sex_mod1.a`, so we will move forward with the simplest model, which is `sex_mod1.a`. Therefore, demographic interactions will not be retained in this model.

### 4. Removal of parameters one by one

* We will start with removing `exp`, which has the lowest t-value


```r
sex_mod1.b <- lm(final_c ~ sex_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(sex_mod1.b)
```

```
## 
## Call:
## lm(formula = final_c ~ sex_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = master)
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

#### AIC comparison of `sex_mod1.a` and `sex_mod1.b`


```r
AIC(sex_mod1.a, sex_mod1.b)
```

```
##            df      AIC
## sex_mod1.a  8 16185.02
## sex_mod1.b  7 16183.43
```

* The AIC *decreases* by *less than 2* for `sex_mod1.b`, so we will retain this model since it is the simplest. 

* Now we will try removing `sex_id`


```r
sex_mod1.c <- lm(final_c ~ satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(sex_mod1.c)
```

```
## 
## Call:
## lm(formula = final_c ~ satm_c + satv_c + aleksikc_c + hs_gpa_c, 
##     data = master)
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

#### AIC comparison of `sex_mod1.b` and `sex_mod1.c`


```r
AIC(sex_mod1.b, sex_mod1.c)
```

```
##            df      AIC
## sex_mod1.b  7 16183.43
## sex_mod1.c  6 16197.64
```

* The AIC *increases* by *more than 2* for `sex_mod1.c`, so we will retain the `sex_id` variable in our model.
* Our final model is `sex_mod1.b`, which is summarized above.

## Model selection for `urm_id`

### 1. Fixed-effects only model (URM)


```r
urm_mod1 <- lm(final_c ~ exp * urm_id +
                         exp + urm_id +
                         satm_c + satv_c + aleksikc_c + hs_gpa_c,
                         data = master)
summary(urm_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * urm_id + exp + urm_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = master)
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

### 2. Random intercepts model with `ta_sect`; REML = TRUE (URM)


```r
urm_mod2 <- lmer(final_c ~ exp * urm_id +
                         exp + urm_id +
                         satm_c + satv_c + aleksikc_c + hs_gpa_c +
                         (1 | ta_sect),
                         data = master, REML = TRUE)
summary(urm_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * urm_id + exp + urm_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: master
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

#### AIC comparison of `urm_mod1` and `urm_mod2`


```r
AIC(urm_mod1, urm_mod2)
```

```
##          df      AIC
## urm_mod1  9 14672.14
## urm_mod2 10 14688.57
```

* The AIC *increases* by *more than 2* for `urm_mod2`, so we will not include random effects from `ta_sect` in our model.

### 3. Fixed-effects only model with *no* demographic interactions (URM)


```r
urm_mod1.a <- lm(final_c ~ exp + urm_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(urm_mod1.a)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + urm_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = master)
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

#### AIC comparison of `urm_mod1` and `urm_mod1.a`


```r
AIC(urm_mod1, urm_mod1.a)
```

```
##            df      AIC
## urm_mod1    9 14672.14
## urm_mod1.a  8 14670.38
```

* The AIC *decreases* by *less than 2* for `urm_mod1.a`, so we will retain this model because it is the simplest.

### 4. Removal of parameters one by one

* Start by removing `urm_id` because it has the lowest t-value


```r
urm_mod1.b <- lm(final_c ~ exp +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(urm_mod1.b)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + satm_c + satv_c + aleksikc_c + hs_gpa_c, 
##     data = master)
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

#### AIC comparison of `urm_mod1.a` and `urm_mod1.b`


```r
AIC(urm_mod1.a, urm_mod1.b)
```

```
## Warning in AIC.default(urm_mod1.a, urm_mod1.b): models are not all fitted to the
## same number of observations
```

```
##            df      AIC
## urm_mod1.a  8 14670.38
## urm_mod1.b  7 16199.21
```

* The AIC *increases* by *more than 2* for `urm_mod1.b`, so we will retain `urm_id` in our model.

* Now let's try removing `exp` from our model:


```r
urm_mod1.c <- lm(final_c ~ urm_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(urm_mod1.c)
```

```
## 
## Call:
## lm(formula = final_c ~ urm_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = master)
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

#### AIC comparison of `urm_mod1.a` and `urm_mod1.c`


```r
AIC(urm_mod1.a, urm_mod1.c)
```

```
##            df      AIC
## urm_mod1.a  8 14670.38
## urm_mod1.c  7 14669.58
```

* The AIC *decreases* by *less than 2* for `urm_mod1.c`, which is the simplest model, so we will remove the `exp` parameter from our model.

* Now let's try removing `satv_c`


```r
urm_mod1.d <- lm(final_c ~ urm_id +
                           satm_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(urm_mod1.d)
```

```
## 
## Call:
## lm(formula = final_c ~ urm_id + satm_c + aleksikc_c + hs_gpa_c, 
##     data = master)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -59.778  -8.239   1.217   9.205  35.759 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.038618   0.324265   0.119    0.905    
## urm_idURM    0.695947   0.940092   0.740    0.459    
## satm_c       0.114576   0.004607  24.869   <2e-16 ***
## aleksikc_c   0.207246   0.019709  10.515   <2e-16 ***
## hs_gpa_c    22.480863   1.733056  12.972   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.91 on 1840 degrees of freedom
##   (342 observations deleted due to missingness)
## Multiple R-squared:  0.4162,	Adjusted R-squared:  0.4149 
## F-statistic: 327.9 on 4 and 1840 DF,  p-value: < 2.2e-16
```

#### AIC comparison of `urm_mod1.c` and `urm_mod1.d`


```r
AIC(urm_mod1.c, urm_mod1.d)
```

```
##            df      AIC
## urm_mod1.c  7 14669.58
## urm_mod1.d  6 14682.51
```

* The AIC *increases* by *more than 2* for `urm_mod1.d`, so we will retain the `satv_c` parameter in our model.
* Our final model is `urm_mod1.c`, which is summarized above.

## Model selection for `eop_id`

### 1. Fixed-effects only model (EOP)


```r
eop_mod1 <- lm(final_c ~ exp * eop_id +
                         exp + eop_id +
                         satm_c + satv_c + aleksikc_c + hs_gpa_c,
                         data = master)
summary(eop_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * eop_id + exp + eop_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = master)
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

### 2. Random intercepts model with `ta_sect`; REML = TRUE (EOP)


```r
eop_mod2 <- lmer(final_c ~ exp * eop_id +
                           exp + eop_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c +
                           (1 | ta_sect),
                           data = master, REML = TRUE)
summary(eop_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * eop_id + exp + eop_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: master
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

#### AIC comparison of `urm_mod1` and `urm_mod2`


```r
AIC(eop_mod1, eop_mod2)
```

```
##          df      AIC
## eop_mod1  9 16202.52
## eop_mod2 10 16220.08
```

* The AIC *increases* by *more than 2* for `eop_mod2`, so we will not retain random effects from `ta_sect` in our model.


### 3. Fixed-effects only model with *no* demographic interactions (EOP)


```r
eop_mod1.a <- lm(final_c ~ exp + eop_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(eop_mod1.a)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + eop_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = master)
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

#### AIC comparison of `eop_mod1` and `eop_mod1.a`


```r
AIC(eop_mod1, eop_mod1.a)
```

```
##            df      AIC
## eop_mod1    9 16202.52
## eop_mod1.a  8 16200.54
```

* The AIC *decreases* by *less than 2* for `eop_mod1.a`, which is the simplest model, so we will not include interactions from `eop_id` in our model.

### 4. Removal of parameters one by one

* Start by removing `exp` because it has the lowest t-value


```r
eop_mod1.b <- lmer(final_c ~ eop_id +
                             satm_c + satv_c + aleksikc_c + hs_gpa_c +
                             (1 | ta_sect),
                             data = master, REML = TRUE)
summary(eop_mod1.b)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ eop_id + satm_c + satv_c + aleksikc_c + hs_gpa_c +  
##     (1 | ta_sect)
##    Data: master
## 
## REML criterion at convergence: 16204
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6565 -0.6152  0.1066  0.6789  2.9048 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.633   1.278  
##  Residual             164.152  12.812  
## Number of obs: 2037, groups:  ta_sect, 104
## 
## Fixed effects:
##              Estimate Std. Error t value
## (Intercept) -0.115816   0.351387  -0.330
## eop_idEOP    0.597339   0.774504   0.771
## satm_c       0.104430   0.005003  20.873
## satv_c       0.018880   0.005110   3.695
## aleksikc_c   0.198899   0.018200  10.928
## hs_gpa_c    21.638409   1.650849  13.107
## 
## Correlation of Fixed Effects:
##            (Intr) ep_EOP satm_c satv_c alksk_
## eop_idEOP  -0.460                            
## satm_c     -0.125  0.251                     
## satv_c     -0.057  0.119 -0.481              
## aleksikc_c  0.011 -0.019 -0.221  0.000       
## hs_gpa_c   -0.056  0.091 -0.041 -0.104 -0.009
```

#### AIC comparison of `eop_mod1.a` and `eop_mod1.b`


```r
AIC(eop_mod1.a, eop_mod1.b)
```

```
##            df      AIC
## eop_mod1.a  8 16200.54
## eop_mod1.b  8 16219.98
```

* The AIC *increases* by *more than 2* for `eop_mod1.b`, so we will keep `exp` as a parameter in our model.

* Now let's try removing `eop_id` from our model


```r
eop_mod1.c <- lmer(final_c ~ exp +
                             satm_c + satv_c + aleksikc_c + hs_gpa_c +
                             (1 | ta_sect),
                             data = master, REML = TRUE)
summary(eop_mod1.c)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ exp + satm_c + satv_c + aleksikc_c + hs_gpa_c + (1 |  
##     ta_sect)
##    Data: master
## 
## REML criterion at convergence: 16204.6
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6483 -0.6229  0.1037  0.6766  2.8956 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.688   1.299  
##  Residual             164.126  12.811  
## Number of obs: 2037, groups:  ta_sect, 104
## 
## Fixed effects:
##                Estimate Std. Error t value
## (Intercept)    0.216029   0.467638   0.462
## expEXPERIMENT -0.375422   0.629213  -0.597
## satm_c         0.103478   0.004843  21.368
## satv_c         0.018412   0.005074   3.629
## aleksikc_c     0.199127   0.018198  10.942
## hs_gpa_c      21.528396   1.644152  13.094
## 
## Correlation of Fixed Effects:
##             (Intr) eEXPER satm_c satv_c alksk_
## eEXPERIMENT -0.743                            
## satm_c      -0.001 -0.009                     
## satv_c      -0.002  0.000 -0.532              
## aleksikc_c  -0.001  0.004 -0.223  0.002       
## hs_gpa_c    -0.005 -0.008 -0.066 -0.116 -0.007
```

#### AIC comparison of `eop_mod1.a` and `eop_mod1.c`


```r
AIC(eop_mod1.a, eop_mod1.c)
```

```
##            df      AIC
## eop_mod1.a  8 16200.54
## eop_mod1.c  8 16220.64
```

* The AIC *increases* by *more than 2* for `eop_mod1.c`, so we will retain `eop_id` as a parameter in our model.

* Now let's try removing `satv_c` from our model


```r
eop_mod1.d <- lm(final_c ~ exp + eop_id +
                           satm_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(eop_mod1.d)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + eop_id + satm_c + aleksikc_c + hs_gpa_c, 
##     data = master)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -61.019  -7.944   1.340   8.987  36.064 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    0.171549   0.452955   0.379    0.705    
## expEXPERIMENT -0.391922   0.577231  -0.679    0.497    
## eop_idEOP      0.288623   0.771180   0.374    0.708    
## satm_c         0.113676   0.004405  25.809   <2e-16 ***
## aleksikc_c     0.198735   0.018250  10.890   <2e-16 ***
## hs_gpa_c      22.356939   1.647757  13.568   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.92 on 2031 degrees of freedom
##   (150 observations deleted due to missingness)
## Multiple R-squared:  0.4193,	Adjusted R-squared:  0.4178 
## F-statistic: 293.3 on 5 and 2031 DF,  p-value: < 2.2e-16
```

#### AIC comparison of `eop_mod1.a` and `eop_mod1.d`


```r
AIC(eop_mod1.a, eop_mod1.d)
```

```
##            df      AIC
## eop_mod1.a  8 16200.54
## eop_mod1.d  7 16212.17
```

* The AIC *increases* by *more than 2* for `eop_mod1.d`, so we will retain `satv_c` as a parameter in our model.
* Our final model is `eop_mod1.a`, which is summarized above.

## Model selection for `fgn_id`

### 1. Fixed-effects only model (fgn)


```r
fgn_mod1 <- lm(final_c ~ exp * fgn_id +
                         exp + fgn_id +
                         satm_c + satv_c + aleksikc_c + hs_gpa_c,
                         data = master)
summary(fgn_mod1)
```

```
## 
## Call:
## lm(formula = final_c ~ exp * fgn_id + exp + fgn_id + satm_c + 
##     satv_c + aleksikc_c + hs_gpa_c, data = master)
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

### 2. Random intercepts model with `ta_sect`; REML = TRUE (FGN)


```r
fgn_mod2 <- lmer(final_c ~ exp * fgn_id +
                           exp + fgn_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c +
                           (1 | ta_sect),
                           data = master, REML = TRUE)
summary(fgn_mod2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: 
## final_c ~ exp * fgn_id + exp + fgn_id + satm_c + satv_c + aleksikc_c +  
##     hs_gpa_c + (1 | ta_sect)
##    Data: master
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

#### AIC comparison of `fgn_mod1` and `fgn_mod2`


```r
AIC(fgn_mod1, fgn_mod2)
```

```
##          df      AIC
## fgn_mod1  9 16175.94
## fgn_mod2 10 16194.49
```

* The AIC *increases* by more than 2 for `fgn_mod2`, so random effects are not retained in the model.

### 3. Fixed-effects only model with *no* demographic interactions (FGN)


```r
fgn_mod1.a <- lm(final_c ~ exp + fgn_id +
                           satm_c + satv_c + aleksikc_c + hs_gpa_c,
                           data = master)
summary(fgn_mod1.a)
```

```
## 
## Call:
## lm(formula = final_c ~ exp + fgn_id + satm_c + satv_c + aleksikc_c + 
##     hs_gpa_c, data = master)
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

#### AIC comparison of `sex_mod1` and `sex_mod1.a`


```r
AIC(fgn_mod1, fgn_mod1.a)
```

```
##            df      AIC
## fgn_mod1    9 16175.94
## fgn_mod1.a  8 16177.32
```

* The AIC *increases* by *less than 2* for `fgn_mod1.a`, so we will move forward with the simplest model, which is `fgn_mod1.a`. Therefore, demographic interactions will not be retained in this model.

### 4. Removal of parameters one by one

* Start by removing `exp` because it has the lowest t-value


```r
fgn_mod1.b <- lmer(final_c ~ fgn_id +
                             satm_c + satv_c + aleksikc_c + hs_gpa_c +
                             (1 | ta_sect),
                             data = master, REML = TRUE)
summary(fgn_mod1.b)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ fgn_id + satm_c + satv_c + aleksikc_c + hs_gpa_c +  
##     (1 | ta_sect)
##    Data: master
## 
## REML criterion at convergence: 16181.4
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6820 -0.6183  0.1029  0.6775  2.9377 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.436   1.198  
##  Residual             163.758  12.797  
## Number of obs: 2035, groups:  ta_sect, 104
## 
## Fixed effects:
##             Estimate Std. Error t value
## (Intercept)  0.39329    0.36423   1.080
## fgn_idFGN   -1.33153    0.70527  -1.888
## satm_c       0.10217    0.00489  20.895
## satv_c       0.01590    0.00524   3.034
## aleksikc_c   0.19943    0.01817  10.977
## hs_gpa_c    21.50410    1.64250  13.092
## 
## Correlation of Fixed Effects:
##            (Intr) fg_FGN satm_c satv_c alksk_
## fgn_idFGN  -0.531                            
## satm_c     -0.089  0.150                     
## satv_c     -0.138  0.256 -0.470              
## aleksikc_c  0.007 -0.009 -0.222 -0.001       
## hs_gpa_c   -0.031  0.033 -0.060 -0.104 -0.007
```
#### AIC comparison of `fgn_mod1.a` and `fgn_mod1.b`


```r
AIC(fgn_mod1.a, fgn_mod1.b)
```

```
##            df      AIC
## fgn_mod1.a  8 16177.32
## fgn_mod1.b  8 16197.42
```

* The AIC *increases* by *more than 2* for `fgn_mod1.b`, so we will retain `exp` as a parameter in our model.

* Now let's try removing `fgn_id` from our model


```r
fgn_mod1.c <- lmer(final_c ~ exp +
                             satm_c + satv_c + aleksikc_c + hs_gpa_c +
                             (1 | ta_sect),
                             data = master, REML = TRUE)
summary(fgn_mod1.c)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ exp + satm_c + satv_c + aleksikc_c + hs_gpa_c + (1 |  
##     ta_sect)
##    Data: master
## 
## REML criterion at convergence: 16204.6
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.6483 -0.6229  0.1037  0.6766  2.8956 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.688   1.299  
##  Residual             164.126  12.811  
## Number of obs: 2037, groups:  ta_sect, 104
## 
## Fixed effects:
##                Estimate Std. Error t value
## (Intercept)    0.216029   0.467638   0.462
## expEXPERIMENT -0.375422   0.629213  -0.597
## satm_c         0.103478   0.004843  21.368
## satv_c         0.018412   0.005074   3.629
## aleksikc_c     0.199127   0.018198  10.942
## hs_gpa_c      21.528396   1.644152  13.094
## 
## Correlation of Fixed Effects:
##             (Intr) eEXPER satm_c satv_c alksk_
## eEXPERIMENT -0.743                            
## satm_c      -0.001 -0.009                     
## satv_c      -0.002  0.000 -0.532              
## aleksikc_c  -0.001  0.004 -0.223  0.002       
## hs_gpa_c    -0.005 -0.008 -0.066 -0.116 -0.007
```

#### AIC comparison of `fgn_mod1.a` and `fgn_mod1.c`


```r
AIC(fgn_mod1.a, fgn_mod1.c)
```

```
## Warning in AIC.default(fgn_mod1.a, fgn_mod1.c): models are not all fitted to the
## same number of observations
```

```
##            df      AIC
## fgn_mod1.a  8 16177.32
## fgn_mod1.c  8 16220.64
```

* The AIC *increases* by *more than 2* for `fgn_mod1.c`, so we will retain `fgn_id` as a parameter in our model.

* Now let's try removing `satv_c` from our model


```r
fgn_mod1.d <- lmer(final_c ~ exp + fgn_id +
                             satm_c + aleksikc_c + hs_gpa_c +
                             (1 | ta_sect),
                             data = master, REML = TRUE)
summary(fgn_mod1.d)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ exp + fgn_id + satm_c + aleksikc_c + hs_gpa_c + (1 |  
##     ta_sect)
##    Data: master
## 
## REML criterion at convergence: 16180.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.7272 -0.6223  0.1018  0.6920  2.8800 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)   1.415   1.19   
##  Residual             164.481  12.83   
## Number of obs: 2035, groups:  ta_sect, 104
## 
## Fixed effects:
##                Estimate Std. Error t value
## (Intercept)    0.800418   0.502485   1.593
## expEXPERIMENT -0.452924   0.621409  -0.729
## fgn_idFGN     -1.894477   0.683591  -2.771
## satm_c         0.109149   0.004326  25.234
## aleksikc_c     0.199423   0.018207  10.953
## hs_gpa_c      22.029130   1.637192  13.455
## 
## Correlation of Fixed Effects:
##             (Intr) eEXPER fg_FGN satm_c alksk_
## eEXPERIMENT -0.695                            
## fgn_idFGN   -0.394  0.030                     
## satm_c      -0.126  0.000  0.317              
## aleksikc_c   0.003  0.003 -0.009 -0.252       
## hs_gpa_c    -0.028 -0.007  0.061 -0.124 -0.007
```

#### AIC comparison of `fgn_mod1.a` and `fgn_mod1.d`


```r
AIC(fgn_mod1.a, fgn_mod1.d)
```

```
##            df      AIC
## fgn_mod1.a  8 16177.32
## fgn_mod1.d  8 16196.53
```

* The AIC *increases* by *more than 2* for `fgn_mod1.d`, so we will retain `satv_c` as a parameter in our model.
* Our final model is `fgn_mod1.a`, which is summarized above.

## Notes
* I ended up stopping the one-by-one removal of parameters after `satv_c` based on the intuition that the other parameters had t-values much greater than that of `satv_c`.
