---
title: "Two-Stage Report"
author: "Jackson Hughes"
date: "5/11/2022"
output: 
  html_document:
    keep_md: yes
---

Do two-stage exams help alleviate historic demographic-based achievement gaps in introductory chemistry courses at UW? A statistical analysis of classroom performance using R programming.



## Introduction

In undergraduate science courses, the implementation of collaborative testing has been an increasingly popular way to facilitate student learning. Two-stage exams are a form of collaborative learning where students first take an exam on their own, and then spread out into groups to take the same, or a very similar exam. Students typically receive credit for a mixture of both the individual and group portions of the exam, with the individual portion weighted more heavily. Student performance has been shown to generally increase on the group portion of the exam as compared to the individual portion [Gilley and Clarkston 2014], indicating evidence for immediate learning between the two stages of the exam. On the other hand, two-stage exams have recently been shown not to affect long-term retention of course information for students in a medical radiation technology class [Bentley et al. 2021]. Similarly, students who participated in two-stage exams in an introductory computer science class presented short-term gains by outperforming individually assessed students on quizzes shortly after exams, but this effect was lost on a long-term scale due to no difference in performance during a final exam [Cao and Porter 2017]. Evidently, two-stage exams can facilitate immediate, short-term learning for students, but evidence for this type of collaborative testing to augment student learning on a long-term basis is inconclusive.

At the University of Washington, two-stage exams were implemented as quizzes in an introductory chemistry course (CHEM 142) in Autumn 2017 to investigate if historical demographic-based achievement gaps in introductory chemistry can be alleviated by collaborative testing. These two-stage quizzes were taken by students during weekly TA-led quiz sections.

The demographic identifiers collected from teh UW Registrar students were binary sex ID, under-represented minority status, first generation status, and Education Opportunity Program status. Student performance on the final exam for this course was compared with a control CHEM 142 class in which two-stage exams were not implemented. Within R, linear models were generated to analyze how final exam scores change between the experimental and control years for four different demographic groups, further controlling for student performance indicators like high school GPA, SAT scores, and homework scores.

Here, the experimental design and data presentation for the investigation of the effect of two-stage quizzes on demographic-based achievement gaps in introductory chemistry courses at UW is presented.

## Methods

All statistical analyses were conducted within the R programming environment. We collected student performance and demographic data from a control group consisting of Autumn 2016 CHEM 142 students who did not take two-stage quizzes, and from an experimental group of Autumn 2017 CHEM 142 students who participated in two-stage exams. Data from both cohorts of students were merged to a master dataset where a categorical variable titled `exp` was assigned to each student as `CONTROL` or `EXPERIMENT`. Demographic data for binary sex, under-represented minority status, first-generation status, and Education Opportunity Program status were contained within categorical variables titled `sex_id`, `urm_id`, `fgn_id`, and `eop_id`, respectively. A variety of numerical student performance indicators were included in the dataset to be used as control variables: SAT math and SAT verbal scores as `satm` and `satv`, ALEKS initial knowledge check score as `aleks_ikc`, and unweighted high school GPA as `hs_gpa`. 

We used linear regression to generate four models to compare exam scores across the control and experimental student groups with interactions from each demographic identifier. Furthermore, we used multilevel regression to investigate if random effects on student exam performance from TA section should be accounted for within the models. Since two-stage quizzes were administered within TA-led quiz sections, it is possible that the degree to which this intervention influenced student exam performance could be impacted by variables such as the time of day of the quiz section, the different TAs leading the quiz sections, or the various settings in which quiz sections were held. The fit of the fixed-effects linear regression models and that of the multilevel regression models associated with each demographic ID were compared using Akaike’s Information Criterion (AIC) [CITATION], and we determined that the linear models that did not include random effects from TA sections fit the data more accurately. Next, we removed parameters from the linear models one by one, starting with the demographic ID interactions, and then moving on to the student performance indicator control variables, comparing the AIC value of each new model to the best-fitting one that came before it. If the fit of one model was better or the same than a more complex model that came before it based on its AIC value, then we retained the less complex model as more representative of the data. [More specifics about AIC should be included here?]

Here, an annotated example of model selection for the models associated with `sex_id` by comparison of AIC values is shown.

### Model selection for `sex_id`

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

## Results

The best-fitting regression models for student performance on the final exams between the control and experimental years for each demographic ID are provided along with their interpretations.


```r
# `sex_id`
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

(interpretation here)


```r
# `urm_id`
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

(interpretation here)


```r
# `eop_id`
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

(interpretation here)


```r
# `fgn_id`
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

(interpretation here)

There was no evidence for significant change in final exam scores for any of the four demographic groups between the control and experimental years, indicating that two-stage exams are not shown to help narrow historical demographic-based achievement gaps in our context and implementation. The context of this study was to investigate student learning on a long-term scale throughout the entire quarter rather than short-term. These findings reflect those of the studies discussed earlier in the introduction—two-stage exams did not facilitate student learning and retention on a long-term scale.
