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

#### First, I'm going to temporarily change `ta_sect` into character values


```r
master_unique$ta_sect <- as.character(master_unique$ta_sect)
```

#### Now, I'm going to split the dataset in half; one only contains 2016 data, and the other only contains 2017 data.


```r
master_unique_2016 <- subset(master_unique, course_fullid != "CHEM_142_A_2017_4" & course_fullid != "CHEM_142_B_2017_4")
master_unique_2017 <- subset(master_unique, course_fullid != "CHEM_142_A_2016_4" & course_fullid != "CHEM_142_B_2016_4")
```

#### Now I can individually change the values of the `ta_sect` column without it affecting the `ta_sect` values from the other year


```r
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AA", "AA_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AB", "AB_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AC", "AC_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AD", "AD_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AE", "AE_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AF", "AF_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AG", "AG_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AH", "AH_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AI", "AI_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AJ", "AJ_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AK", "AK_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AL", "AL_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AM", "AM_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AN", "AN_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AO", "AO_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AP", "AP_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AQ", "AQ_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AR", "AR_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AS", "AS_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AT", "AT_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AU", "AU_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AV", "AV_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AW", "AW_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AX", "AX_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AY", "AY_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="AZ", "AZ_16")

master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BA", "BA_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BB", "BB_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BC", "BC_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BD", "BD_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BE", "BE_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BF", "BF_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BG", "BG_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BH", "BH_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BI", "BI_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BJ", "BJ_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BK", "BK_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BL", "BL_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BM", "BM_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BN", "BN_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BO", "BO_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BP", "BP_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BQ", "BQ_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BR", "BR_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BS", "BS_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BT", "BT_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BU", "BU_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BV", "BV_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BW", "BW_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BX", "BX_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BY", "BY_16")
master_unique_2016$ta_sect <- replace(master_unique_2016$ta_sect, master_unique_2016$ta_sect=="BZ", "BZ_16")

master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AA", "AA_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AB", "AB_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AC", "AC_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AD", "AD_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AE", "AE_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AF", "AF_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AG", "AG_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AH", "AH_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AI", "AI_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AJ", "AJ_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AK", "AK_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AL", "AL_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AM", "AM_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AN", "AN_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AO", "AO_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AP", "AP_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AQ", "AQ_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AR", "AR_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AS", "AS_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AT", "AT_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AU", "AU_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AV", "AV_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AW", "AW_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AX", "AX_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AY", "AY_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="AZ", "AZ_17")

master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BA", "BA_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BB", "BB_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BC", "BC_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BD", "BD_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BE", "BE_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BF", "BF_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BG", "BG_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BH", "BH_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BI", "BI_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BJ", "BJ_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BK", "BK_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BL", "BL_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BM", "BM_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BN", "BN_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BO", "BO_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BP", "BP_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BQ", "BQ_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BR", "BR_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BS", "BS_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BT", "BT_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BU", "BU_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BV", "BV_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BW", "BW_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BX", "BX_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BY", "BY_17")
master_unique_2017$ta_sect <- replace(master_unique_2017$ta_sect, master_unique_2017$ta_sect=="BZ", "BZ_17")
```

#### That was a lot... anyways now I'm going to join the data frames back together and make `ta_sect` into a factor again!


```r
master_true <- rbind(master_unique_2016, master_unique_2017)

master_true$ta_sect <- as.factor(master_true$ta_sect)
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
## -4.4294 -0.6320  0.1018  0.7018  2.9273 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  ta_sect  (Intercept)  18.03    4.246  
##  Residual             336.37   18.340  
## Number of obs: 7321, groups:  ta_sect, 108
## 
## Fixed effects:
##                          Estimate Std. Error t value
## (Intercept)              0.420468   0.684220   0.615
## experiment1EXPERIMENTAL -0.750637   0.946457  -0.793
## satm_c                   0.145235   0.004050  35.858
## satv_c                   0.031886   0.004121   7.737
## aleksikc_c               0.303828   0.014390  21.113
## hsgpa_c                 31.523395   1.256575  25.087
## 
## Correlation of Fixed Effects:
##             (Intr) e1EXPE satm_c satv_c alksk_
## e1EXPERIMEN -0.721                            
## satm_c       0.039 -0.002                     
## satv_c      -0.033 -0.003 -0.604              
## aleksikc_c   0.000  0.008 -0.187 -0.015       
## hsgpa_c     -0.005  0.002 -0.062 -0.108 -0.012
```

## Model Selection Using AIC

#### I am going to be comparing the standard regression model with the random intercepts model


```r
AIC(mod_1, mod_2)
```

```
##       df      AIC
## mod_1  7 63734.08
## mod_2  8 63561.51
```

#### It appears that for the random intercepts model, the AIC decreases by a significant amount (decreases by MUCH more than 2), so the random intercepts model is a much better fit for the data. Cool!

## Notes
* The way that I made the `ta_sect` values unique across both years was very long and convoluted. There has to be a better way... right?
* It's too damn hot out.
