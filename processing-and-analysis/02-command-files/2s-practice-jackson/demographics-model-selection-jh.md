---
title: 'Model Selection on Demographic Interactions'
author: "Jackson Hughes"
date: "7/19/2021"
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


```r
master_true <- master_unique %>% 
  mutate(ta_sect = ifelse(str_detect(course_fullid, "2016"), paste(ta_sect, "16", sep = "_"), paste(ta_sect, "17", sep = "_")))
```

# Model Selection!

## Model selection on `sex_id` interactions

### Fixed-effects only (`sex_id`)


```r
sex_mod1 <- lm(final_c ~ experiment1*sex_id + experiment1 + sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
```

### Random-intercepts model with `REML = TRUE` (`sex_id`)


```r
sex_mod2 <- lmer(final_c ~ experiment1*sex_id + experiment1 + sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
```

### AIC comparison of `sex_mod1` and `sex_mod2`


```r
AIC(sex_mod1, sex_mod2)
```

```
##          df      AIC
## sex_mod1  9 63662.44
## sex_mod2 10 63494.22
```

* The AIC *decreases* by 168.22 for `sex_mod2`.

### Model with no `sex_id` interactions and `REML = FALSE`


```r
sex_mod3 <- lmer(final_c ~ experiment1 + sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = FALSE)
```

### AIC comparision of `sex_mod2` and `sex_mod3`


```r
AIC(sex_mod2, sex_mod3)
```

```
##          df      AIC
## sex_mod2 10 63494.22
## sex_mod3  9 63475.91
```

* The AIC *decreases* by 18.31 for `sex_mod3`. Therefore, the model without interactions from `sex_id` fits the best.

### Final result: random effects model with no `sex_id` interactions and `REML = TRUE`


```r
sex_mod4 <- lmer(final_c ~ experiment1 + sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
summary(sex_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 + sex_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                          Estimate Std. Error t value
## (Intercept)              2.433031   0.717178   3.393
## experiment1EXPERIMENTAL -0.750240   0.931789  -0.805
## sex_idFemale            -3.762328   0.461102  -8.159
## satm_c                   0.136930   0.004159  32.920
## satv_c                   0.036858   0.004147   8.887
## aleksikc_c               0.298561   0.014340  20.820
## hsgpa_c                 33.008236   1.264096  26.112
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE sx_dFm satm_c satv_c alksk_
## e1EXPERIMEN -0.677                                   
## sex_idFemal -0.343 -0.001                            
## satm_c      -0.048 -0.002  0.245                     
## satv_c       0.019 -0.003 -0.146 -0.615              
## aleksikc_c  -0.015  0.008  0.045 -0.170 -0.021       
## hsgpa_c      0.045  0.003 -0.144 -0.094 -0.085 -0.019
```

## Model selection on `urm_id` interactions

### Fixed-effects only (`urm_id`)


```r
urm_mod1 <- lm(final_c ~ experiment1*urm_id + experiment1 + urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
```

### Random-intercepts model with `REML = TRUE` (`urm_id`)


```r
urm_mod2 <- lmer(final_c ~ experiment1*urm_id + experiment1 + urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
```

### AIC comparison of `urm_mod1` and `urm_mod2`


```r
AIC(urm_mod1, urm_mod2)
```

```
##          df      AIC
## urm_mod1  9 63731.76
## urm_mod2 10 63552.88
```

* The AIC *decreases* by 178.88 for `urm_mod2`.

### Model with no `urm_id` interactions and `REML = FALSE`


```r
urm_mod3 <- lmer(final_c ~ experiment1 + urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = FALSE)
```

### AIC comparison of `urm_mod2` and `urm_mod3`


```r
AIC(urm_mod2, urm_mod3)
```

```
##          df      AIC
## urm_mod2 10 63552.88
## urm_mod3  9 63536.59
```

* The AIC *decreases* by 16.29 for `urm_mod3`. Therefore, the model without interactions from `urm_id` fits the best.

### Final result: random effects model with no `urm_id` interactions and `REML = TRUE`


```r
urm_mod4 <- lmer(final_c ~ experiment1 + urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
summary(urm_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 + urm_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                          Estimate Std. Error t value
## (Intercept)              0.222016   0.689502   0.322
## experiment1EXPERIMENTAL -0.783242   0.947451  -0.827
## urm_idURM                1.661903   0.697732   2.382
## satm_c                   0.147127   0.004125  35.663
## satv_c                   0.032513   0.004128   7.877
## aleksikc_c               0.301982   0.014407  20.960
## hsgpa_c                 31.883610   1.264975  25.205
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE ur_URM satm_c satv_c alksk_
## e1EXPERIMEN -0.714                                   
## urm_idURM   -0.119 -0.013                            
## satm_c       0.016 -0.004  0.192                     
## satv_c      -0.041 -0.004  0.062 -0.579              
## aleksikc_c   0.007  0.009 -0.055 -0.194 -0.018       
## hsgpa_c     -0.019  0.001  0.118 -0.037 -0.100 -0.019
```

## Model selection on `eop_id` interactions

### Fixed-effects only (`eop_id`)


```r
eop_mod1 <- lm(final_c ~ experiment1*eop_id + experiment1 + eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
```

### Random-intercepts model with `REML = TRUE` (`eop_id`)


```r
eop_mod2 <- lmer(final_c ~ experiment1*eop_id + experiment1 + eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
```

### AIC comparison of `eop_mod1` and `eop_mod2`


```r
AIC(eop_mod1, eop_mod2)
```

```
##          df      AIC
## eop_mod1  9 63732.78
## eop_mod2 10 63557.41
```

* The AIC *decreases* by 175.37 for `eop_mod2`.

### Model with no `eop_id` interactions and `REML = FALSE`


```r
eop_mod3 <- lmer(final_c ~ experiment1 + eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = FALSE)
```

### AIC comparison of `eop_mod2` and `eop_mod3`


```r
AIC(eop_mod2, eop_mod3)
```

```
##          df      AIC
## eop_mod2 10 63557.41
## eop_mod3  9 63537.26
```

* The AIC *decreases* by 20.15 for `eop_mod3`. Therefore, the model without interactions from `eop_id` fits the best.

### Final result: random effects model with no `eop_id` interactions and `REML = TRUE`


```r
eop_mod4 <- lmer(final_c ~ experiment1 + eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
summary(eop_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 + eop_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                          Estimate Std. Error t value
## (Intercept)              0.168937   0.693283   0.244
## experiment1EXPERIMENTAL -0.825532   0.947455  -0.871
## eop_idEOP                1.298958   0.581053   2.236
## satm_c                   0.147088   0.004132  35.595
## satv_c                   0.033261   0.004164   7.987
## aleksikc_c               0.302698   0.014396  21.027
## hsgpa_c                 31.808909   1.262470  25.196
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE ep_EOP satm_c satv_c alksk_
## e1EXPERIMEN -0.706                                   
## eop_idEOP   -0.161 -0.034                            
## satm_c       0.006 -0.008  0.200                     
## satv_c      -0.056 -0.008  0.146 -0.556              
## aleksikc_c   0.006  0.009 -0.037 -0.191 -0.020       
## hsgpa_c     -0.021 -0.001  0.099 -0.040 -0.092 -0.016
```

## Model selection on `fgn_id` interactions

### Fixed-effects only (`fgn_id`)


```r
fgn_mod1 <- lm(final_c ~ experiment1*fgn_id + experiment1 + fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
```

### Random-intercepts model with `REML = TRUE` (`fgn_id`)


```r
fgn_mod2 <- lmer(final_c ~ experiment1*fgn_id + experiment1 + fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
```

### AIC comparison of `fgn_mod1` and `fgn_mod2`


```r
AIC(fgn_mod1, fgn_mod2)
```

```
##          df      AIC
## fgn_mod1  9 63713.53
## fgn_mod2 10 63544.11
```

* The AIC *decreases* by 169.42 for `fgn_mod2`.

### Model with no `fgn_id` interactions and `REML = FALSE`


```r
fgn_mod3 <- lmer(final_c ~ experiment1 + fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = FALSE)
```

### AIC comparison of `fgn_mod2` and `fgn_mod3`


```r
AIC(fgn_mod2, fgn_mod3)
```

```
##          df      AIC
## fgn_mod2 10 63544.11
## fgn_mod3  9 63535.00
```

* The AIC *decreases* by 9.11 for `eop_mod3`. Therefore, the model without interactions from `eop_id` fits the best. 

### Final result: random effects model with no `fgn_id` interactions and `REML = TRUE`


```r
fgn_mod4 <- lmer(final_c ~ experiment1 + fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_true, REML = TRUE)
summary(fgn_mod4)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_c ~ experiment1 + fgn_id + satm_c + satv_c + aleksikc_c +  
##     hsgpa_c + (1 | ta_sect)
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
##                          Estimate Std. Error t value
## (Intercept)              0.851659   0.699966   1.217
## experiment1EXPERIMENTAL -0.782266   0.942858  -0.830
## fgn_idFGN               -1.468229   0.545320  -2.692
## satm_c                   0.143637   0.004093  35.097
## satv_c                   0.028890   0.004269   6.767
## aleksikc_c               0.304944   0.014389  21.193
## hsgpa_c                 31.423043   1.256675  25.005
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE fg_FGN satm_c satv_c alksk_
## e1EXPERIMEN -0.705                                   
## fgn_idFGN   -0.229  0.011                            
## satm_c       0.005  0.000  0.146                     
## satv_c      -0.092  0.000  0.263 -0.538              
## aleksikc_c   0.007  0.008 -0.028 -0.189 -0.021       
## hsgpa_c     -0.012  0.003  0.031 -0.056 -0.096 -0.013
```

## Notes/Questions
* For every demographic identifier, the model selection process has shown that the best-fitting model is the one that includes random effects, but *doesn't* include any interactions from the demographic IDs.
* How are we able to significantly compare `dem_mod2` and `dem_mod3` when one uses REML and the other uses ML?
* ^^ According to Zuur et al. 2009, "we must use REML estimators to compare these (nested) models," i.e., the models that include the `ta_sect` random effects.
