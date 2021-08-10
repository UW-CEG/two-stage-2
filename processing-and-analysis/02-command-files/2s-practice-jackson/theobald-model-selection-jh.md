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
master_unique_1 <- master_extras_removed %>% 
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
## sex_mod1  9 18607.93
## sex_mod2 10 18616.32
```

* The AIC *increases* for `sex_mod2` ...
* Therefore the random effects will not be included

### Model with no `sex_id` interactions or random effects


```r
sex_mod3 <- lm(final_c ~ experiment1 + sex_id + satm_c + satv_c + aleksikc_c, data = master_true)
```

### AIC comparision of `sex_mod1` and `sex_mod3` to see if we should include `sex_id` interactions


```r
AIC(sex_mod1, sex_mod3)
```

```
##          df      AIC
## sex_mod1  9 18607.93
## sex_mod3  7 18783.23
```

* The AIC *increases* for `sex_mod3`, so we should proceed with `sex_mod1`, which includes `sex_id` interactions.

### Removal of parameters one by one


```r
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

* `satv_c` has the lowest t-value, so let's try removing it:


```r
sex_mod1.a <- lm(final_c ~ experiment1*sex_id + experiment1 + sex_id + satm_c + aleksikc_c + hsgpa_c, data = master_true)
AIC(sex_mod1, sex_mod1.a)
```

```
##            df      AIC
## sex_mod1    9 18607.93
## sex_mod1.a  8 18634.87
```

* The AIC *increases* for `sex_mod1.a`, so we should keep the original model `sex_mod1` (the summary for it is a few lines above ^)

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
## urm_mod1  9 18629.57
## urm_mod2 10 18635.45
```

* The AIC *increases* for `urm_mod2` (ughhhh). So we will proceed with the model without random effects.

### Model with no `urm_id` interactions or random effects


```r
urm_mod3 <- lm(final_c ~ experiment1 + urm_id + satm_c + satv_c + aleksikc_c, data = master_true)
```

### AIC comparison of `urm_mod1` and `urm_mod3`


```r
AIC(urm_mod1, urm_mod3)
```

```
##          df      AIC
## urm_mod1  9 18629.57
## urm_mod3  7 18789.79
```

* The AIC *increases* for `urm_mod3`, which indicates that we should retain the `urm_id` interactions in our model.

### Removal of parameters one by one


```r
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

* `satv_c` has the lowest t-value, so we should try removing that parameter from the model first.


```r
urm_mod1.a <- lm(final_c ~ experiment1*urm_id + experiment1 + urm_id + satm_c + aleksikc_c + hsgpa_c, data = master_true)
AIC(urm_mod1, urm_mod1.a)
```

```
##            df      AIC
## urm_mod1    9 18629.57
## urm_mod1.a  8 18649.80
```

* The AIC increases for `urm_mod1.a`, so we should retain the `urm_mod1` model (summary is a few lines above ^)

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
## eop_mod1  9 18629.78
## eop_mod2 10 18636.63
```

* The AIC *increases* for `eop_mod2`, so we should not include random effects in our model.

### Model with no `eop_id` interactions or random effects


```r
eop_mod3 <- lm(final_c ~ experiment1 + eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
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
* In this case it appears that the model could go either way, but I will go ahead and keep the `eop_id` interactions

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

* Once again, `satv_c` has the lowest t-value, so we will try removing it from the model.


```r
eop_mod3.a <- lm(final_c ~ experiment1 + eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
AIC(eop_mod3, eop_mod3.a)
```

```
##            df      AIC
## eop_mod3    8 18627.79
## eop_mod3.a  8 18627.79
```

* There is no difference WTF. Let's just keep `eop_mod3`, which includes `satv_c` as a parameter (summary is a few lines above ^)

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
##          df     AIC
## fgn_mod1  9 18624.2
## fgn_mod2 10 18632.2
```

* The AIC *increases* for `fgn_mod2`. Therefore we will not include random effects in our model.

### Model with no `fgn_id` interactions or random effects


```r
fgn_mod3 <- lm(final_c ~ experiment1 + fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c, data = master_true)
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

* The AIC *increases* for `fgn_mod3`, so we will retain the `fgn_id` interactions in our model.

### Removing parameters one by one


```r
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

* `satv_c` again has the lowest t-value, so we will try removing it from the model.


```r
fgn_mod1.a <- lm(final_c ~ experiment1*fgn_id + experiment1 + fgn_id + satm_c + aleksikc_c + hsgpa_c, data = master_true)
AIC(fgn_mod1, fgn_mod1.a)
```

```
##            df      AIC
## fgn_mod1    9 18624.20
## fgn_mod1.a  8 18638.27
```

* The AIC *increases* for `fgn_mod1.a`, so we will retain `fgn_mod1` (summary is a few lines above ^)

## Notes
* I changed the "condensing the dataset" code to create a data frame that I believe is now truly unique (`master_true` now only has 2134 rows as opposed to ~7000 like before)
* ^ I did this by also including `two_stage_id` in the group of columns to run the `unique` function on. Every student has a unique 2 stage ID, so it must be important to run the `unique` function on this portion of the data as well.
* Now with this "new" unique data, it appears that the model does not fit as well when random effects are included...
