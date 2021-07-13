---
title: 'MLM With Non-Unique TA Sections'
author: "Jackson Hughes"
date: "7/12/2021"
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
master_extras_removed <- subset(master, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))
master_unique_1 <- unique(master_extras_removed, incomparables = FALSE)
master_unique_2 <- master_unique_1 %>% 
  select(exam1, exam2, finalexam, course_fullid, ta_sect, ver, sex_id, urm_id, eop_id, fgn_id, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc, experiment1, satm_c, satv_c, aleksikc_c, hsgpa_c, final_c, satm_z, satv_z, aleksikc_z, hsgpa_z, final_z)
master_unique <- na.omit(master_unique_2)
```

# Regression

## Regression of `final_c` on `experiment1` with control variables (no MLM)


```r
mod_1 <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_unique)
summary(mod_1)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.732 -12.006   2.003  12.879  54.983 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.610106   0.328189   1.859   0.0631 .  
## experiment1EXPERIMENTAL -0.808950   0.441427  -1.833   0.0669 .  
## satm_c                   0.147775   0.004053  36.463  < 2e-16 ***
## satv_c                   0.032531   0.004142   7.853 4.64e-15 ***
## aleksikc_c               0.300737   0.014343  20.967  < 2e-16 ***
## hsgpa_c                 31.511301   1.257999  25.049  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.79 on 7315 degrees of freedom
## Multiple R-squared:  0.4218,	Adjusted R-squared:  0.4214 
## F-statistic:  1067 on 5 and 7315 DF,  p-value: < 2.2e-16
```

## MLM random intercepts model for TA section (centered data)


```r
mod_2 <- lmer(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_unique)
summary(mod_2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c +  
##     (1 | ta_sect)
##    Data: master_unique
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
##                          Estimate Std. Error t value
## (Intercept)              0.637424   0.569195   1.120
## experiment1EXPERIMENTAL -0.944387   0.441795  -2.138
## satm_c                   0.146296   0.004033  36.274
## satv_c                   0.033824   0.004107   8.235
## aleksikc_c               0.301956   0.014336  21.062
## hsgpa_c                 31.070227   1.252658  24.803
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satm_c satv_c alksk_
## e1EXPERIMEN -0.436                            
## satm_c       0.039  0.009                     
## satv_c      -0.038 -0.011 -0.603              
## aleksikc_c   0.006  0.011 -0.190 -0.013       
## hsgpa_c     -0.004  0.004 -0.059 -0.105 -0.008
```

## Random effects models including demographic interactions

### Interactions from `sex_id`


```r
sex_id_mod <- lmer(final_c ~ experiment1*sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_unique)
summary(sex_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * sex_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
##    Data: master_unique
## 
## REML criterion at convergence: 63521.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.3988 -0.6321  0.0912  0.7044  2.8207 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.13    3.336  
##  Residual             338.60   18.401  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                                       Estimate Std. Error t value
## (Intercept)                           3.179679   0.673267   4.723
## experiment1EXPERIMENTAL              -1.764289   0.658997  -2.677
## sex_idFemale                         -4.746987   0.666485  -7.122
## satm_c                                0.137618   0.004141  33.230
## satv_c                                0.039165   0.004133   9.476
## aleksikc_c                            0.296132   0.014281  20.736
## hsgpa_c                              32.712556   1.260619  25.950
## experiment1EXPERIMENTAL:sex_idFemale  1.517018   0.882932   1.718
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL sx_dFm satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.541                                                   
## sex_idFemal     -0.539  0.537                                            
## satm_c          -0.055  0.000           0.165                            
## satv_c           0.028 -0.019          -0.114 -0.614                     
## aleksikc_c      -0.013  0.007           0.033 -0.172 -0.020              
## hsgpa_c          0.061 -0.019          -0.123 -0.092 -0.081 -0.015       
## e1EXPERIMENTAL:  0.399 -0.745          -0.724  0.007  0.016  0.000  0.030
```

### Interactions from `urm_id`


```r
urm_id_mod <- lmer(final_c ~ experiment1*urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_unique)
summary(urm_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * urm_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
##    Data: master_unique
## 
## REML criterion at convergence: 63586.4
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.2715 -0.6212  0.1002  0.7012  2.8765 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.4     3.377  
##  Residual             341.7    18.484  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                                    Estimate Std. Error t value
## (Intercept)                        0.583253   0.585318   0.996
## experiment1EXPERIMENTAL           -1.253940   0.474291  -2.644
## urm_idURM                          0.518473   1.018652   0.509
## satm_c                             0.148215   0.004112  36.042
## satv_c                             0.034486   0.004112   8.386
## aleksikc_c                         0.299753   0.014356  20.880
## hsgpa_c                           31.564443   1.263230  24.987
## experiment1EXPERIMENTAL:urm_idURM  2.127075   1.324307   1.606
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ur_URM satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.448                                                   
## urm_idURM       -0.208  0.250                                            
## satm_c           0.007  0.008           0.145                            
## satv_c          -0.043 -0.016           0.030 -0.579                     
## aleksikc_c       0.013  0.013          -0.039 -0.198 -0.017              
## hsgpa_c         -0.011 -0.022           0.035 -0.035 -0.097 -0.016       
## e1EXPERIMENTAL:  0.157 -0.364          -0.736 -0.015  0.012 -0.003  0.061
```

### Interactions from `eop_id`


```r
eop_id_mod <- lmer(final_c ~ experiment1*eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_unique)
summary(eop_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * eop_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
##    Data: master_unique
## 
## REML criterion at convergence: 63590.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.2810 -0.6184  0.1021  0.7038  2.8837 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  11.23    3.351  
##  Residual             341.84   18.489  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                                    Estimate Std. Error t value
## (Intercept)                        0.439002   0.593759   0.739
## experiment1EXPERIMENTAL           -1.119868   0.502885  -2.227
## eop_idEOP                          1.059802   0.853872   1.241
## satm_c                             0.148154   0.004115  35.999
## satv_c                             0.035227   0.004154   8.481
## aleksikc_c                         0.300746   0.014345  20.965
## hsgpa_c                           31.371710   1.259161  24.915
## experiment1EXPERIMENTAL:eop_idEOP  0.455174   1.071324   0.425
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ep_EOP satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.457                                                   
## eop_idEOP       -0.282  0.307                                            
## satm_c          -0.004  0.004           0.148                            
## satv_c          -0.067 -0.011           0.114 -0.554                     
## aleksikc_c       0.015  0.006          -0.035 -0.194 -0.019              
## hsgpa_c         -0.014 -0.020           0.039 -0.038 -0.090 -0.012       
## e1EXPERIMENTAL:  0.213 -0.474          -0.738 -0.019 -0.017  0.012  0.036
```

### Interactions from `fgn_id`


```r
fgn_id_mod <- lmer(final_c ~ experiment1*fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_unique)
summary(fgn_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * fgn_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
##    Data: master_unique
## 
## REML criterion at convergence: 63575.3
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.3202 -0.6321  0.1009  0.7031  2.9454 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  10.91    3.303  
##  Residual             341.17   18.471  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                                    Estimate Std. Error t value
## (Intercept)                        0.617454   0.605646   1.019
## experiment1EXPERIMENTAL           -0.037508   0.520650  -0.072
## fgn_idFGN                          0.080102   0.753286   0.106
## satm_c                             0.144414   0.004071  35.475
## satv_c                             0.030141   0.004255   7.084
## aleksikc_c                         0.303141   0.014323  21.164
## hsgpa_c                           30.757043   1.253012  24.546
## experiment1EXPERIMENTAL:fgn_idFGN -3.308594   0.977133  -3.386
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL fg_FGN satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.485                                                   
## fgn_idFGN       -0.364  0.380                                            
## satm_c          -0.001  0.008           0.102                            
## satv_c          -0.101 -0.011           0.183 -0.537                     
## aleksikc_c       0.013  0.008          -0.021 -0.192 -0.020              
## hsgpa_c          0.000 -0.022          -0.013 -0.054 -0.093 -0.009       
## e1EXPERIMENTAL:  0.251 -0.531          -0.692  0.004  0.012  0.001  0.048
```

## Model Selection Using AIC

#### Standard regression model vs. random intercepts model (no demographic interactions)


```r
AIC(mod_1, mod_2)
```

```
##       df      AIC
## mod_1  7 63734.08
## mod_2  8 63614.71
```

#### AIC comparison of all of the demographic interactions random effects models


```r
AIC(sex_id_mod, urm_id_mod, eop_id_mod, fgn_id_mod)
```

```
##            df      AIC
## sex_id_mod 10 63541.54
## urm_id_mod 10 63606.41
## eop_id_mod 10 63610.51
## fgn_id_mod 10 63595.26
```
