---
title: 'MLM With Unique TA Sections'
author: "Jackson Hughes"
date: "6/28/2021"
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

### Make `ta_sect` values unique across `course_fullid`

#### I'm going to be trying the code that CFC showed us


```r
master_true <- master_unique %>% 
  mutate(ta_sect = ifelse(str_detect(course_fullid, "2016"), paste(ta_sect, "16", sep = "_"), paste(ta_sect, "17", sep = "_")))
```

# Regression

## Regression of `final_c` on `experiment1` with control variables (no MLM)


```r
mod_1 <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
summary(mod_1)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_true)
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
mod_2 <- lmer(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true)
summary(mod_2)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c +  
##     (1 | ta_sect)
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
##                          Estimate Std. Error t value
## (Intercept)              0.418187   0.684614   0.611
## experiment1EXPERIMENTAL -0.753496   0.947419  -0.795
## satm_c                   0.145244   0.004050  35.860
## satv_c                   0.031904   0.004121   7.742
## aleksikc_c               0.303877   0.014390  21.117
## hsgpa_c                 31.529202   1.256596  25.091
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satm_c satv_c alksk_
## e1EXPERIMEN -0.721                            
## satm_c       0.040 -0.002                     
## satv_c      -0.033 -0.003 -0.604              
## aleksikc_c   0.001  0.008 -0.187 -0.015       
## hsgpa_c     -0.005  0.002 -0.061 -0.108 -0.012
```

## Random effects models including demographic interactions

### Interactions from `sex_id`


```r
sex_id_mod <- lmer(final_c ~ experiment1*sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true)
summary(sex_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * sex_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                                       Estimate Std. Error t value
## (Intercept)                           2.901708   0.764893   3.794
## experiment1EXPERIMENTAL              -1.610332   1.052227  -1.530
## sex_idFemale                         -4.613909   0.669087  -6.896
## satm_c                                0.136967   0.004159  32.933
## satv_c                                0.037000   0.004148   8.921
## aleksikc_c                            0.298657   0.014338  20.829
## hsgpa_c                              33.067489   1.264367  26.153
## experiment1EXPERIMENTAL:sex_idFemale  1.559497   0.888033   1.756
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL sx_dFm satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.724                                                   
## sex_idFemal     -0.475  0.337                                            
## satm_c          -0.043 -0.004           0.165                            
## satv_c           0.024 -0.012          -0.115 -0.614                     
## aleksikc_c      -0.013  0.005           0.028 -0.170 -0.021              
## hsgpa_c          0.051 -0.010          -0.118 -0.094 -0.084 -0.019       
## e1EXPERIMENTAL:  0.349 -0.465          -0.725  0.005  0.020  0.004  0.027
```

### Interactions from `urm_id`


```r
urm_id_mod <- lmer(final_c ~ experiment1*urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true)
summary(urm_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * urm_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                                    Estimate Std. Error t value
## (Intercept)                        0.386873   0.699205   0.553
## experiment1EXPERIMENTAL           -1.098088   0.968317  -1.134
## urm_idURM                          0.282789   1.025037   0.276
## satm_c                             0.146998   0.004125  35.634
## satv_c                             0.032587   0.004127   7.896
## aleksikc_c                         0.302114   0.014405  20.972
## hsgpa_c                           32.033628   1.267378  25.276
## experiment1EXPERIMENTAL:urm_idURM  2.448620   1.333264   1.837
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ur_URM satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.720                                                   
## urm_idURM       -0.176  0.122                                            
## satm_c           0.013 -0.001           0.142                            
## satv_c          -0.039 -0.005           0.035 -0.579                     
## aleksikc_c       0.008  0.008          -0.041 -0.194 -0.018              
## hsgpa_c         -0.010 -0.011           0.033 -0.038 -0.099 -0.018       
## e1EXPERIMENTAL:  0.130 -0.178          -0.733 -0.016  0.010  0.005  0.064
```

### Interactions from `eop_id`


```r
eop_id_mod <- lmer(final_c ~ experiment1*eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true)
summary(eop_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * eop_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                                    Estimate Std. Error t value
## (Intercept)                        0.235751   0.705001   0.334
## experiment1EXPERIMENTAL           -0.953294   0.977372  -0.975
## eop_idEOP                          0.954933   0.858484   1.112
## satm_c                             0.147040   0.004133  35.574
## satv_c                             0.033223   0.004165   7.976
## aleksikc_c                         0.302882   0.014400  21.033
## hsgpa_c                           31.834800   1.263417  25.197
## experiment1EXPERIMENTAL:eop_idEOP  0.587915   1.079954   0.544
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL ep_EOP satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.717                                                   
## eop_idEOP       -0.236  0.155                                            
## satm_c           0.002 -0.003           0.150                            
## satv_c          -0.058 -0.004           0.111 -0.555                     
## aleksikc_c       0.010  0.003          -0.042 -0.191 -0.020              
## hsgpa_c         -0.014 -0.010           0.039 -0.041 -0.092 -0.015       
## e1EXPERIMENTAL:  0.175 -0.241          -0.736 -0.021 -0.017  0.023  0.037
```

### Interactions from `fgn_id`


```r
fgn_id_mod <- lmer(final_c ~ experiment1*fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true)
summary(fgn_id_mod)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 * fgn_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                                    Estimate Std. Error t value
## (Intercept)                        0.324271   0.715197   0.453
## experiment1EXPERIMENTAL            0.170009   0.980926   0.173
## fgn_idFGN                          0.329805   0.757416   0.435
## satm_c                             0.143628   0.004090  35.120
## satv_c                             0.028667   0.004266   6.719
## aleksikc_c                         0.304793   0.014379  21.198
## hsgpa_c                           31.212568   1.257265  24.826
## experiment1EXPERIMENTAL:fgn_idFGN -3.351159   0.980296  -3.419
## 
## Correlation of Fixed Effects:
##                 (Intr) ex1EXPERIMENTAL fg_FGN satm_c satv_c alksk_ hsgp_c
## ex1EXPERIMENTAL -0.721                                                   
## fgn_idFGN       -0.311  0.205                                            
## satm_c           0.005  0.000           0.105                            
## satv_c          -0.086 -0.004           0.178 -0.538                     
## aleksikc_c       0.007  0.007          -0.022 -0.189 -0.021              
## hsgpa_c         -0.001 -0.011          -0.012 -0.056 -0.095 -0.013       
## e1EXPERIMENTAL:  0.216 -0.284          -0.695  0.001  0.015  0.003  0.049
```

## Model Selection Using AIC

#### Standard regression model vs. random intercepts model (no demographic interactions)


```r
AIC(mod_1, mod_2)
```

```
##       df      AIC
## mod_1  7 63734.08
## mod_2  8 63561.45
```

#### AIC comparison of all of the demographic interactions random effects models


```r
AIC(sex_id_mod, urm_id_mod, eop_id_mod, fgn_id_mod)
```

```
##            df      AIC
## sex_id_mod 10 63494.22
## urm_id_mod 10 63552.88
## eop_id_mod 10 63557.41
## fgn_id_mod 10 63544.11
```

## Notes
* I'd like to discuss regex some more... I had some trouble implementing the CFC code from the google doc into this data set.
