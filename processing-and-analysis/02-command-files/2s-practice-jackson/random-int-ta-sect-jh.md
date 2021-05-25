---
title: 'MLM: Random Intercepts Model'
author: "Jackson Hughes"
date: "5/24/2021"
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

## Random intercepts model for TA section (centered data)


```r
rand_int_mod_c <- lmer(final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + (1|ta_sect), data = master_unique)
summary(rand_int_mod_c)
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

## Random intercepts model for TA section (z-scored data)


```r
rand_int_mod_z <- lmer(final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + (1|ta_sect), data = master_unique)
summary(rand_int_mod_z)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z +  
##     (1 | ta_sect)
##    Data: master_unique
## 
## REML criterion at convergence: 16406.3
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -4.2468 -0.6282  0.1003  0.6974  2.8289 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept) 0.01755  0.1325  
##  Residual             0.54122  0.7357  
## Number of obs: 7321, groups:  ta_sect, 57
## 
## Fixed effects:
##                          Estimate Std. Error t value
## (Intercept)              0.025729   0.022570   1.140
## experiment1EXPERIMENTAL -0.038226   0.017574  -2.175
## satm_z                   0.429672   0.011876  36.179
## satv_z                   0.092508   0.011244   8.227
## aleksikc_z               0.200202   0.009379  21.345
## hsgpa_z                  0.223195   0.008958  24.916
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satm_z satv_z alksk_
## e1EXPERIMEN -0.438                            
## satm_z       0.039  0.009                     
## satv_z      -0.038 -0.011 -0.603              
## aleksikc_z   0.006  0.009 -0.192 -0.013       
## hsgpa_z     -0.004  0.004 -0.058 -0.107 -0.007
```

## Notes
* Not sure how to interpret the summary data yet
* What is "Correlation of Fixed Effects"?
