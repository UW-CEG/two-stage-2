---
title: "Exploring Regression"
author: "Jackson Hughes"
date: "5/5/2021"
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
  select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id, satverbal, satmath, high_sch_gpa, mastered_topics_initial_kc, experiment1, satm_c, satv_c, aleksikc_c, hsgpa_c, final_c, satm_z, satv_z, aleksikc_z, hsgpa_z, final_z)
master_unique <- na.omit(master_unique_2)
```

# Regression

## Multiple Regression with the "centered" data

### 1. Regression of `final_c` on `experiment1`


```r
final_c_model <- lm(final_c ~ experiment1, data = master_unique)
summary(final_c_model)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -79.923 -15.633   2.197  18.657  45.097 
## 
## Coefficients:
##                         Estimate Std. Error t value Pr(>|t|)  
## (Intercept)             -0.07866    0.42939  -0.183   0.8547  
## experiment1EXPERIMENTAL -1.00252    0.57941  -1.730   0.0836 .
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 24.71 on 7347 degrees of freedom
## Multiple R-squared:  0.0004073,	Adjusted R-squared:  0.0002713 
## F-statistic: 2.994 on 1 and 7347 DF,  p-value: 0.08363
```

### 2. Regression of `final_c` on `experiment1`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, and `hsgpa_c`


```r
final_c_model_controlled <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_unique)
summary(final_c_model_controlled)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.702 -12.156   2.039  12.908  54.988 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.570918   0.327827   1.742   0.0816 .  
## experiment1EXPERIMENTAL -0.805132   0.440962  -1.826   0.0679 .  
## satm_c                   0.147985   0.004041  36.617  < 2e-16 ***
## satv_c                   0.032261   0.004135   7.801 6.96e-15 ***
## aleksikc_c               0.301307   0.014318  21.044  < 2e-16 ***
## hsgpa_c                 31.506693   1.257004  25.065  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.81 on 7343 degrees of freedom
## Multiple R-squared:  0.4215,	Adjusted R-squared:  0.4211 
## F-statistic:  1070 on 5 and 7343 DF,  p-value: < 2.2e-16
```

### 3. Regression of `final_c` on `experiment1`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, `hsgpa_c`, and `sex_id`


```r
final_c_model_controlled_sex <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + sex_id, data = master_unique)
summary(final_c_model_controlled_sex)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c + sex_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -90.817 -12.057   1.919  13.175  53.654 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              2.692047   0.409496   6.574 5.23e-11 ***
## experiment1EXPERIMENTAL -0.804725   0.438803  -1.834   0.0667 .  
## satm_c                   0.139130   0.004152  33.507  < 2e-16 ***
## satv_c                   0.037558   0.004161   9.026  < 2e-16 ***
## aleksikc_c               0.295396   0.014264  20.709  < 2e-16 ***
## hsgpa_c                 33.141614   1.265316  26.192  < 2e-16 ***
## sex_idFemale            -3.938246   0.459566  -8.569  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.71 on 7342 degrees of freedom
## Multiple R-squared:  0.4272,	Adjusted R-squared:  0.4268 
## F-statistic: 912.8 on 6 and 7342 DF,  p-value: < 2.2e-16
```

### 4. Regression of `final_c` on `experiment1`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, `hsgpa_c`, and `eop_id`


```r
final_c_model_controlled_eop <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + eop_id, data = master_unique)
summary(final_c_model_controlled_eop)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c + eop_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.535 -11.962   2.005  12.936  55.687 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.303135   0.346528   0.875   0.3817    
## experiment1EXPERIMENTAL -0.874795   0.441795  -1.980   0.0477 *  
## satm_c                   0.149965   0.004125  36.354  < 2e-16 ***
## satv_c                   0.033754   0.004181   8.072 7.99e-16 ***
## aleksikc_c               0.300190   0.014321  20.962  < 2e-16 ***
## hsgpa_c                 31.797963   1.262561  25.185  < 2e-16 ***
## eop_idEOP                1.365704   0.574280   2.378   0.0174 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.8 on 7342 degrees of freedom
## Multiple R-squared:  0.422,	Adjusted R-squared:  0.4215 
## F-statistic: 893.3 on 6 and 7342 DF,  p-value: < 2.2e-16
```

### 5. Regression of `final_c` on `experiment1`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, `hsgpa_c`, and `fgn_id`


```r
final_c_model_controlled_fgn <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + fgn_id, data = master_unique)
summary(final_c_model_controlled_fgn)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c + fgn_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.824 -12.260   2.027  12.862  55.666 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              1.050560   0.364423   2.883  0.00395 ** 
## experiment1EXPERIMENTAL -0.835034   0.440833  -1.894  0.05824 .  
## satm_c                   0.146218   0.004082  35.822  < 2e-16 ***
## satv_c                   0.028846   0.004286   6.730 1.83e-11 ***
## aleksikc_c               0.302519   0.014316  21.132  < 2e-16 ***
## hsgpa_c                 31.375200   1.257078  24.959  < 2e-16 ***
## fgn_idFGN               -1.641736   0.546062  -3.007  0.00265 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.8 on 7342 degrees of freedom
## Multiple R-squared:  0.4222,	Adjusted R-squared:  0.4218 
## F-statistic: 894.2 on 6 and 7342 DF,  p-value: < 2.2e-16
```

### 6. Regression of `final_c` on `experiment1`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, `hsgpa_c`, and `urm_id`


```r
final_c_model_controlled_urm <- lm(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + urm_id, data = master_unique)
summary(final_c_model_controlled_urm)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + 
##     hsgpa_c + urm_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.526 -11.981   1.958  12.935  55.537 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.361285   0.337745   1.070   0.2848    
## experiment1EXPERIMENTAL -0.832959   0.440928  -1.889   0.0589 .  
## satm_c                   0.150059   0.004120  36.421  < 2e-16 ***
## satv_c                   0.032871   0.004141   7.939 2.34e-15 ***
## aleksikc_c               0.299256   0.014335  20.876  < 2e-16 ***
## hsgpa_c                 31.890221   1.265397  25.202  < 2e-16 ***
## urm_idURM                1.765470   0.688477   2.564   0.0104 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.8 on 7342 degrees of freedom
## Multiple R-squared:  0.422,	Adjusted R-squared:  0.4216 
## F-statistic: 893.5 on 6 and 7342 DF,  p-value: < 2.2e-16
```

## Multiple Regression with the "z-scored" ("scaled") data

### 1. Regression of `final_z` on `experiment1`


```r
final_z_model <- lm(final_z ~ experiment1, data = master_unique)
summary(final_z_model)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.1645 -0.6190  0.0880  0.7387  1.8073 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)  
## (Intercept)             -0.003153   0.017093  -0.184   0.8537  
## experiment1EXPERIMENTAL -0.039656   0.023065  -1.719   0.0856 .
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9839 on 7347 degrees of freedom
## Multiple R-squared:  0.0004022,	Adjusted R-squared:  0.0002661 
## F-statistic: 2.956 on 1 and 7347 DF,  p-value: 0.08561
```

### 2. Regression of `final_z` on `experiment1`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, and `hsgpa_z`


```r
final_z_model_controlled <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z, data = master_unique)
summary(final_z_model_controlled)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + 
##     hsgpa_z, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.4983 -0.4816  0.0827  0.5106  2.1793 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.023143   0.013040   1.775   0.0760 .  
## experiment1EXPERIMENTAL -0.032785   0.017539  -1.869   0.0616 .  
## satm_z                   0.434448   0.011900  36.509  < 2e-16 ***
## satv_z                   0.088218   0.011319   7.794 7.41e-15 ***
## aleksikc_z               0.200142   0.009367  21.366  < 2e-16 ***
## hsgpa_z                  0.226314   0.008988  25.180  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.748 on 7343 degrees of freedom
## Multiple R-squared:  0.4225,	Adjusted R-squared:  0.4221 
## F-statistic:  1074 on 5 and 7343 DF,  p-value: < 2.2e-16
```

### 3. Regression of `final_z` on `experiment1`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, `hsgpa_z`, and `sex_id`


```r
final_z_model_controlled_sex <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + sex_id, data = master_unique)
summary(final_z_model_controlled_sex)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + 
##     hsgpa_z + sex_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.5835 -0.4765  0.0779  0.5230  2.1255 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.107982   0.016286   6.630 3.58e-11 ***
## experiment1EXPERIMENTAL -0.032726   0.017452  -1.875   0.0608 .  
## satm_z                   0.408276   0.012224  33.400  < 2e-16 ***
## satv_z                   0.102745   0.011389   9.022  < 2e-16 ***
## aleksikc_z               0.196309   0.009331  21.038  < 2e-16 ***
## hsgpa_z                  0.238117   0.009048  26.319  < 2e-16 ***
## sex_idFemale            -0.157548   0.018277  -8.620  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7443 on 7342 degrees of freedom
## Multiple R-squared:  0.4283,	Adjusted R-squared:  0.4278 
## F-statistic: 916.7 on 6 and 7342 DF,  p-value: < 2.2e-16
```

### 4. Regression of `final_z` on `experiment1`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, `hsgpa_z`, and `eop_id`


```r
final_z_model_controlled_eop <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + eop_id, data = master_unique)
summary(final_z_model_controlled_eop)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + 
##     hsgpa_z + eop_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.4918 -0.4771  0.0802  0.5140  2.2064 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.012720   0.013783   0.923   0.3561    
## experiment1EXPERIMENTAL -0.035498   0.017572  -2.020   0.0434 *  
## satm_z                   0.440165   0.012147  36.237  < 2e-16 ***
## satv_z                   0.092214   0.011445   8.057 9.07e-16 ***
## aleksikc_z               0.199401   0.009370  21.281  < 2e-16 ***
## hsgpa_z                  0.228337   0.009027  25.295  < 2e-16 ***
## eop_idEOP                0.053162   0.022841   2.327   0.0200 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7478 on 7342 degrees of freedom
## Multiple R-squared:  0.4229,	Adjusted R-squared:  0.4224 
## F-statistic: 896.8 on 6 and 7342 DF,  p-value: < 2.2e-16
```

### 5. Regression of `final_z` on `experiment1`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, `hsgpa_z`, and `fgn_id`


```r
final_z_model_controlled_fgn <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + fgn_id, data = master_unique)
summary(final_z_model_controlled_fgn)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + 
##     hsgpa_z + fgn_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.5031 -0.4880  0.0814  0.5133  2.2067 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.042405   0.014495   2.926  0.00345 ** 
## experiment1EXPERIMENTAL -0.033984   0.017533  -1.938  0.05263 .  
## satm_z                   0.429191   0.012018  35.711  < 2e-16 ***
## satv_z                   0.078772   0.011733   6.714 2.04e-11 ***
## aleksikc_z               0.200963   0.009366  21.457  < 2e-16 ***
## hsgpa_z                  0.225395   0.008988  25.078  < 2e-16 ***
## fgn_idFGN               -0.065935   0.021718  -3.036  0.00241 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7476 on 7342 degrees of freedom
## Multiple R-squared:  0.4232,	Adjusted R-squared:  0.4227 
## F-statistic: 897.9 on 6 and 7342 DF,  p-value: < 2.2e-16
```

### 6. Regression of `final_z` on `experiment1`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, `hsgpa_z`, and `urm_id`


```r
final_z_model_controlled_urm <- lm(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + urm_id, data = master_unique)
summary(final_z_model_controlled_urm)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + 
##     hsgpa_z + urm_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.4915 -0.4753  0.0814  0.5140  2.2006 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)              0.014961   0.013434   1.114   0.2655    
## experiment1EXPERIMENTAL -0.033870   0.017538  -1.931   0.0535 .  
## satm_z                   0.440448   0.012132  36.305  < 2e-16 ***
## satv_z                   0.089858   0.011334   7.928 2.55e-15 ***
## aleksikc_z               0.198816   0.009379  21.199  < 2e-16 ***
## hsgpa_z                  0.228979   0.009047  25.311  < 2e-16 ***
## urm_idURM                0.068902   0.027381   2.516   0.0119 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7478 on 7342 degrees of freedom
## Multiple R-squared:  0.423,	Adjusted R-squared:  0.4225 
## F-statistic:   897 on 6 and 7342 DF,  p-value: < 2.2e-16
```

## Multiple Regression + interactions with the centered data:

### 1. Regression of `final_c` on `experiment1` with interactions from `sex_id`


```r
final_c_model_sex_int <- lm(final_c ~ experiment1*sex_id, data = master_unique)
summary(final_c_model_sex_int)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * sex_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -76.487 -16.359   2.957  17.941  47.941 
## 
## Coefficients:
##                                      Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                            3.4612     0.6367   5.436 5.61e-08 ***
## experiment1EXPERIMENTAL               -0.2267     0.8601  -0.264    0.792    
## sex_idFemale                          -6.3842     0.8550  -7.467 9.17e-14 ***
## experiment1EXPERIMENTAL:sex_idFemale  -1.3677     1.1540  -1.185    0.236    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 24.46 on 7345 degrees of freedom
## Multiple R-squared:  0.02117,	Adjusted R-squared:  0.02077 
## F-statistic: 52.95 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 2. Regression of `final_c` on `experiment1` with interactions from `sex_id`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, and `hsgpa_c`


```r
final_c_model_sex_int_controlled <- lm(final_c ~ experiment1*sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_unique)
summary(final_c_model_sex_int_controlled)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * sex_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -90.451 -11.897   1.847  12.975  54.076 
## 
## Coefficients:
##                                       Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                           3.157123   0.490103   6.442 1.26e-10 ***
## experiment1EXPERIMENTAL              -1.652496   0.658482  -2.510   0.0121 *  
## sex_idFemale                         -4.777298   0.668814  -7.143 1.00e-12 ***
## satm_c                                0.139204   0.004152  33.527  < 2e-16 ***
## satv_c                                0.037675   0.004161   9.054  < 2e-16 ***
## aleksikc_c                            0.295417   0.014262  20.713  < 2e-16 ***
## hsgpa_c                              33.214185   1.265843  26.239  < 2e-16 ***
## experiment1EXPERIMENTAL:sex_idFemale  1.526251   0.883992   1.727   0.0843 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.71 on 7341 degrees of freedom
## Multiple R-squared:  0.4275,	Adjusted R-squared:  0.4269 
## F-statistic:   783 on 7 and 7341 DF,  p-value: < 2.2e-16
```

### 3. Regression of `final_c` on `experiment1` with interactions from `eop_id`


```r
final_c_model_eop_int <- lm(final_c ~ experiment1*eop_id, data = master_unique)
summary(final_c_model_eop_int)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * eop_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -83.052 -16.892   2.668  19.465  51.708 
## 
## Coefficients:
##                                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                         2.6357     0.4682   5.630 1.87e-08 ***
## experiment1EXPERIMENTAL            -0.5876     0.6413  -0.916    0.360    
## eop_idEOP                         -13.6253     1.0489 -12.990  < 2e-16 ***
## experiment1EXPERIMENTAL:eop_idEOP   1.1083     1.3670   0.811    0.418    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 24.11 on 7345 degrees of freedom
## Multiple R-squared:  0.04867,	Adjusted R-squared:  0.04828 
## F-statistic: 125.3 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 4. Regression of `final_c` on `experiment1` with interactions from `eop_id`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, and `hsgpa_c`


```r
final_c_model_eop_int_controlled <- lm(final_c ~ experiment1*eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_unique)
summary(final_c_model_eop_int_controlled)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * eop_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.546 -12.013   1.994  12.945  55.676 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.290684   0.368846   0.788   0.4307    
## experiment1EXPERIMENTAL           -0.851687   0.500144  -1.703   0.0886 .  
## eop_idEOP                          1.428374   0.856675   1.667   0.0955 .  
## satm_c                             0.149975   0.004127  36.343  < 2e-16 ***
## satv_c                             0.033761   0.004182   8.072 7.99e-16 ***
## aleksikc_c                         0.300168   0.014324  20.956  < 2e-16 ***
## hsgpa_c                           31.794007   1.263283  25.168  < 2e-16 ***
## experiment1EXPERIMENTAL:eop_idEOP -0.105228   1.067308  -0.099   0.9215    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.8 on 7341 degrees of freedom
## Multiple R-squared:  0.422,	Adjusted R-squared:  0.4214 
## F-statistic: 765.5 on 7 and 7341 DF,  p-value: < 2.2e-16
```

### 5. Regression of `final_c` on `experiment1` with interactions from `fgn_id`


```r
final_c_model_fgn_int <- lm(final_c ~ experiment1*fgn_id, data = master_unique)
summary(final_c_model_fgn_int)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * fgn_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -76.061 -15.854   3.309  18.219  52.842 
## 
## Coefficients:
##                                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                         3.1094     0.4916   6.326 2.67e-10 ***
## experiment1EXPERIMENTAL             0.5272     0.6592   0.800    0.424    
## fgn_idFGN                         -10.9337     0.9103 -12.011  < 2e-16 ***
## experiment1EXPERIMENTAL:fgn_idFGN  -6.4234     1.2404  -5.179 2.30e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 23.81 on 7345 degrees of freedom
## Multiple R-squared:  0.07223,	Adjusted R-squared:  0.07186 
## F-statistic: 190.6 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 6. Regression of `final_c` on `experiment1` with interactions from `fgn_id`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, and `hsgpa_c`


```r
final_c_model_fgn_int_controlled <- lm(final_c ~ experiment1*fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_unique)
summary(final_c_model_fgn_int_controlled)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * fgn_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_unique)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -89.27 -12.12   1.95  13.09  56.84 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.477232   0.394219   1.211  0.22610    
## experiment1EXPERIMENTAL            0.213681   0.520019   0.411  0.68115    
## fgn_idFGN                          0.326965   0.753005   0.434  0.66415    
## satm_c                             0.146099   0.004078  35.825  < 2e-16 ***
## satv_c                             0.028605   0.004283   6.679 2.58e-11 ***
## aleksikc_c                         0.302513   0.014303  21.151  < 2e-16 ***
## hsgpa_c                           31.163788   1.257169  24.789  < 2e-16 ***
## experiment1EXPERIMENTAL:fgn_idFGN -3.715873   0.979628  -3.793  0.00015 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.78 on 7341 degrees of freedom
## Multiple R-squared:  0.4234,	Adjusted R-squared:  0.4228 
## F-statistic: 769.9 on 7 and 7341 DF,  p-value: < 2.2e-16
```

### 7. Regression of `final_c` on `experiment1` with interactions from `urm_id`


```r
final_c_model_urm_int <- lm(final_c ~ experiment1*urm_id, data = master_unique)
summary(final_c_model_urm_int)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * urm_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -81.684 -16.389   4.036  17.911  51.463 
## 
## Coefficients:
##                                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                         1.4069     0.4517   3.115  0.00185 ** 
## experiment1EXPERIMENTAL            -0.7274     0.6120  -1.188  0.23469    
## urm_idURM                         -12.1521     1.2918  -9.407  < 2e-16 ***
## experiment1EXPERIMENTAL:urm_idURM  -0.5830     1.7037  -0.342  0.73219    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 24.36 on 7345 degrees of freedom
## Multiple R-squared:  0.02947,	Adjusted R-squared:  0.02907 
## F-statistic: 74.35 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 8. Regression of `final_c` on `experiment1` with interactions from `urm_id`, controlling for `satm_c`, `satv_c`, `aleksikc_c`, and `hsgpa_c`


```r
final_c_model_urm_int_controlled <- lm(final_c ~ experiment1*urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_unique)
summary(final_c_model_urm_int_controlled)
```

```
## 
## Call:
## lm(formula = final_c ~ experiment1 * urm_id + satm_c + satv_c + 
##     aleksikc_c + hsgpa_c, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -88.483 -11.991   1.959  12.972  55.594 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.412746   0.349939   1.179   0.2382    
## experiment1EXPERIMENTAL           -0.928555   0.472583  -1.965   0.0495 *  
## urm_idURM                          1.341509   1.020999   1.314   0.1889    
## satm_c                             0.150020   0.004121  36.404  < 2e-16 ***
## satv_c                             0.032892   0.004141   7.943 2.27e-15 ***
## aleksikc_c                         0.299245   0.014335  20.875  < 2e-16 ***
## hsgpa_c                           31.929483   1.267381  25.193  < 2e-16 ***
## experiment1EXPERIMENTAL:urm_idURM  0.740733   1.317225   0.562   0.5739    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.8 on 7341 degrees of freedom
## Multiple R-squared:  0.4221,	Adjusted R-squared:  0.4215 
## F-statistic: 765.8 on 7 and 7341 DF,  p-value: < 2.2e-16
```

## Multiple Regression + interactions with the z-scored (scaled) data:

### 1. Regression of `final_z` on `experiment1` with interactions from `sex_id`


```r
final_z_model_sex_int <- lm(final_z ~ experiment1*sex_id, data = master_unique)
summary(final_z_model_sex_int)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * sex_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.0284 -0.6528  0.1185  0.7190  1.9213 
## 
## Coefficients:
##                                      Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                           0.13872    0.02535   5.473 4.57e-08 ***
## experiment1EXPERIMENTAL              -0.01065    0.03424  -0.311    0.756    
## sex_idFemale                         -0.25586    0.03404  -7.517 6.27e-14 ***
## experiment1EXPERIMENTAL:sex_idFemale -0.05107    0.04594  -1.112    0.266    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9737 on 7345 degrees of freedom
## Multiple R-squared:  0.02112,	Adjusted R-squared:  0.02072 
## F-statistic: 52.83 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 2. Regression of `final_z` on `experiment1` with interactions from `sex_id`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, and `hsgpa_z`


```r
final_z_model_sex_int_controlled <- lm(final_z ~ experiment1*sex_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = master_unique)
summary(final_z_model_sex_int_controlled)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * sex_id + satm_z + satv_z + 
##     aleksikc_z + hsgpa_z, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.5687 -0.4734  0.0695  0.5148  2.1425 
## 
## Coefficients:
##                                       Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                           0.126584   0.019496   6.493 8.98e-11 ***
## experiment1EXPERIMENTAL              -0.066613   0.026191  -2.543   0.0110 *  
## sex_idFemale                         -0.191118   0.026615  -7.181 7.61e-13 ***
## satm_z                                0.408478   0.012223  33.420  < 2e-16 ***
## satv_z                                0.103083   0.011389   9.051  < 2e-16 ***
## aleksikc_z                            0.196240   0.009330  21.033  < 2e-16 ***
## hsgpa_z                               0.238684   0.009052  26.368  < 2e-16 ***
## experiment1EXPERIMENTAL:sex_idFemale  0.061005   0.035161   1.735   0.0828 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7442 on 7341 degrees of freedom
## Multiple R-squared:  0.4285,	Adjusted R-squared:  0.428 
## F-statistic: 786.3 on 7 and 7341 DF,  p-value: < 2.2e-16
```

### 3. Regression of `final_z` on `experiment1` with interactions from `eop_id`


```r
final_z_model_eop_int <- lm(final_z ~ experiment1*eop_id, data = master_unique)
summary(final_z_model_eop_int)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * eop_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.2884 -0.6770  0.1056  0.7707  2.0723 
## 
## Coefficients:
##                                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.10563    0.01864   5.668  1.5e-08 ***
## experiment1EXPERIMENTAL           -0.02454    0.02553  -0.961    0.336    
## eop_idEOP                         -0.54606    0.04176 -13.078  < 2e-16 ***
## experiment1EXPERIMENTAL:eop_idEOP  0.05046    0.05442   0.927    0.354    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9599 on 7345 degrees of freedom
## Multiple R-squared:  0.04868,	Adjusted R-squared:  0.04829 
## F-statistic: 125.3 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 4. Regression of `final_z` on `experiment1` with interactions from `eop_id`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, and `hsgpa_z`


```r
final_z_model_eop_int_controlled <- lm(final_z ~ experiment1*eop_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = master_unique)
summary(final_z_model_eop_int_controlled)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * eop_id + satm_z + satv_z + 
##     aleksikc_z + hsgpa_z, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.4927 -0.4761  0.0794  0.5151  2.2056 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.011723   0.014671   0.799   0.4243    
## experiment1EXPERIMENTAL           -0.033648   0.019893  -1.691   0.0908 .  
## eop_idEOP                          0.058182   0.034084   1.707   0.0879 .  
## satm_z                             0.440230   0.012152  36.226  < 2e-16 ***
## satv_z                             0.092248   0.011447   8.058 8.96e-16 ***
## aleksikc_z                         0.199379   0.009371  21.276  < 2e-16 ***
## hsgpa_z                            0.228285   0.009031  25.277  < 2e-16 ***
## experiment1EXPERIMENTAL:eop_idEOP -0.008424   0.042449  -0.198   0.8427    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7478 on 7341 degrees of freedom
## Multiple R-squared:  0.4229,	Adjusted R-squared:  0.4224 
## F-statistic: 768.6 on 7 and 7341 DF,  p-value: < 2.2e-16
```

### 5. Regression of `final_z` on `experiment1` with interactions from `fgn_id`


```r
final_z_model_fgn_int <- lm(final_z ~ experiment1*fgn_id, data = master_unique)
summary(final_z_model_fgn_int)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * fgn_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.0116 -0.6315  0.1326  0.7214  2.1178 
## 
## Coefficients:
##                                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.12461    0.01957   6.367 2.04e-10 ***
## experiment1EXPERIMENTAL            0.01937    0.02625   0.738    0.461    
## fgn_idFGN                         -0.43819    0.03624 -12.090  < 2e-16 ***
## experiment1EXPERIMENTAL:fgn_idFGN -0.24904    0.04939  -5.043 4.70e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9481 on 7345 degrees of freedom
## Multiple R-squared:  0.0719,	Adjusted R-squared:  0.07152 
## F-statistic: 189.7 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 6. Regression of `final_z` on `experiment1` with interactions from `fgn_id`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, and `hsgpa_z`


```r
final_z_model_fgn_int_controlled <- lm(final_z ~ experiment1*fgn_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = master_unique)
summary(final_z_model_fgn_int_controlled)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * fgn_id + satm_z + satv_z + 
##     aleksikc_z + hsgpa_z, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.5213 -0.4818  0.0793  0.5211  2.2541 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.019178   0.015678   1.223 0.221286    
## experiment1EXPERIMENTAL            0.008527   0.020682   0.412 0.680150    
## fgn_idFGN                          0.013872   0.029950   0.463 0.643243    
## satm_z                             0.428941   0.012007  35.724  < 2e-16 ***
## satv_z                             0.077981   0.011724   6.652 3.11e-11 ***
## aleksikc_z                         0.201099   0.009357  21.492  < 2e-16 ***
## hsgpa_z                            0.223924   0.008987  24.915  < 2e-16 ***
## experiment1EXPERIMENTAL:fgn_idFGN -0.150607   0.038959  -3.866 0.000112 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7469 on 7341 degrees of freedom
## Multiple R-squared:  0.4244,	Adjusted R-squared:  0.4238 
## F-statistic: 773.2 on 7 and 7341 DF,  p-value: < 2.2e-16
```

### 7. Regression of `final_z` on `experiment1` with interactions from `urm_id`


```r
final_z_model_urm_int <- lm(final_z ~ experiment1*urm_id, data = master_unique)
summary(final_z_model_urm_int)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * urm_id, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.2342 -0.6568  0.1598  0.7178  2.0625 
## 
## Coefficients:
##                                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.05638    0.01798   3.136  0.00172 ** 
## experiment1EXPERIMENTAL           -0.02948    0.02436  -1.210  0.22634    
## urm_idURM                         -0.48702    0.05143  -9.470  < 2e-16 ***
## experiment1EXPERIMENTAL:urm_idURM -0.01721    0.06782  -0.254  0.79966    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9696 on 7345 degrees of freedom
## Multiple R-squared:  0.02944,	Adjusted R-squared:  0.02904 
## F-statistic: 74.26 on 3 and 7345 DF,  p-value: < 2.2e-16
```

### 8. Regression of `final_z` on `experiment1` with interactions from `urm_id`, controlling for `satm_z`, `satv_z`, `aleksikc_z`, and `hsgpa_z`


```r
final_z_model_urm_int_controlled <- lm(final_z ~ experiment1*urm_id + satm_z + satv_z + aleksikc_z + hsgpa_z, data = master_unique)
summary(final_z_model_urm_int_controlled)
```

```
## 
## Call:
## lm(formula = final_z ~ experiment1 * urm_id + satm_z + satv_z + 
##     aleksikc_z + hsgpa_z, data = master_unique)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.4900 -0.4754  0.0788  0.5136  2.2025 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                        0.016749   0.013919   1.203   0.2289    
## experiment1EXPERIMENTAL           -0.037191   0.018796  -1.979   0.0479 *  
## urm_idURM                          0.054163   0.040621   1.333   0.1824    
## satm_z                             0.440334   0.012135  36.287  < 2e-16 ***
## satv_z                             0.089918   0.011335   7.933 2.46e-15 ***
## aleksikc_z                         0.198804   0.009379  21.196  < 2e-16 ***
## hsgpa_z                            0.229211   0.009060  25.300  < 2e-16 ***
## experiment1EXPERIMENTAL:urm_idURM  0.025734   0.052387   0.491   0.6233    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7478 on 7341 degrees of freedom
## Multiple R-squared:  0.423,	Adjusted R-squared:  0.4225 
## F-statistic: 768.8 on 7 and 7341 DF,  p-value: < 2.2e-16
```
