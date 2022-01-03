---
title: 'Two Stage Data Visualization'
author: "Jackson Hughes"
date: "1/3/2022"
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
  select(two_stage_id, course_fullid, exp, exam1, exam2, ta_sect, sex_id, urm_id, eop_id, fgn_id, satm, satv, aleksikc_score, hs_gpa, final)
```

## Data Visualization

### Box Blots

#### Final exam score vs. `sex_id`


```r
ggplot(data = master, mapping = aes(x = sex_id, y = final)) +
  geom_boxplot() +
  labs(title = "Boxplot of final exam score vs. sex ID")
```

```
## Warning: Removed 35 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

#### Final exam score vs. `urm_id`


```r
ggplot(data = master, mapping = aes(x = urm_id, y = final)) +
  geom_boxplot() +
  labs(title = "Boxplot of final exam score vs. URM ID")
```

```
## Warning: Removed 35 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

#### Final exam score vs. `eop_id`


```r
ggplot(data = master, mapping = aes(x = eop_id, y = final)) +
  geom_boxplot() +
  labs(title = "Boxplot of final exam score vs. EOP ID")
```

```
## Warning: Removed 35 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

#### Final exam score vs. `fgn_id`


```r
ggplot(data = master, mapping = aes(x = fgn_id, y = final)) +
  geom_boxplot() +
  labs(title = "Boxplot of final exam score vs. FGN ID")
```

```
## Warning: Removed 35 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

#### `satm` vs. `sex_id`


```r
ggplot(data = master, mapping = aes(x = sex_id, y = satm)) +
  geom_boxplot() +
  labs(title = "Boxplot of SAT math score vs. sex ID")
```

```
## Warning: Removed 62 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

#### `satm` vs. `urm_id`


```r
ggplot(data = master, mapping = aes(x = urm_id, y = satm)) +
  geom_boxplot() +
  labs(title = "Boxplot of SAT math score vs. URM ID")
```

```
## Warning: Removed 62 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

#### `satm` vs. `eop_id`


```r
ggplot(data = master, mapping = aes(x = eop_id, y = satm)) +
  geom_boxplot() +
  labs(title = "Boxplot of SAT math score vs. EOP ID")
```

```
## Warning: Removed 62 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

#### `satm` vs. `fgn_id`


```r
ggplot(data = master, mapping = aes(x = fgn_id, y = satm)) +
  geom_boxplot() +
  labs(title = "Boxplot of SAT math score vs. FGN ID")
```

```
## Warning: Removed 62 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

#### `aleksikc_score` vs. `sex_id`


```r
ggplot(data = master, mapping = aes(x = sex_id, y = aleksikc_score)) +
  geom_boxplot() +
  labs(title = "Boxplot of ALEKS Initial Knowledge Check score vs. sex ID")
```

```
## Warning: Removed 26 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

#### `aleksikc_score` vs. `urm_id`


```r
ggplot(data = master, mapping = aes(x = urm_id, y = aleksikc_score)) +
  geom_boxplot() +
  labs(title = "Boxplot of ALEKS Initial Knowledge Check score vs. URM ID")
```

```
## Warning: Removed 26 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

#### `aleksikc_score` vs. `eop_id`


```r
ggplot(data = master, mapping = aes(x = eop_id, y = aleksikc_score)) +
  geom_boxplot() +
  labs(title = "Boxplot of ALEKS Initial Knowledge Check score vs. EOP ID")
```

```
## Warning: Removed 26 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

#### `aleksikc_score` vs. `fgn_id`


```r
ggplot(data = master, mapping = aes(x = fgn_id, y = aleksikc_score)) +
  geom_boxplot() +
  labs(title = "Boxplot of ALEKS Initial Knowledge Check score vs. FGN ID")
```

```
## Warning: Removed 26 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

#### `hs_gpa` vs. `sex_id`


```r
ggplot(data = master, mapping = aes(x = sex_id, y = hs_gpa)) +
  geom_boxplot() +
  labs(title = "Boxplot of high school GPA vs. sex ID")
```

```
## Warning: Removed 56 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

#### `hs_gpa` vs. `urm_id`


```r
ggplot(data = master, mapping = aes(x = urm_id, y = hs_gpa)) +
  geom_boxplot() +
  labs(title = "Boxplot of high school GPA vs. URM ID")
```

```
## Warning: Removed 56 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-18-1.png)<!-- -->

#### `hs_gpa` vs. `eop_id`


```r
ggplot(data = master, mapping = aes(x = eop_id, y = hs_gpa)) +
  geom_boxplot() +
  labs(title = "Boxplot of high school GPA vs. EOP ID")
```

```
## Warning: Removed 56 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-19-1.png)<!-- -->

#### `hs_gpa` vs. `fgn_id`


```r
ggplot(data = master, mapping = aes(x = fgn_id, y = hs_gpa)) +
  geom_boxplot() +
  labs(title = "Boxplot of high school GPA vs. FGN ID")
```

```
## Warning: Removed 56 rows containing non-finite values (stat_boxplot).
```

![](data-visualization_files/figure-html/unnamed-chunk-20-1.png)<!-- -->

### Scatterplots

#### Create subsets of original data for each demographic ID


```r
master_male <- master %>% 
  filter(sex_id == "Male")
master_female <- master %>% 
  filter(sex_id == "Female")
master_urm <- master %>% 
  filter(urm_id == "URM")
master_non_urm <- master %>% 
  filter(urm_id == "non-URM")
master_eop <- master %>% 
  filter(eop_id == "EOP")
master_non_eop <- master %>% 
  filter(eop_id == "non-EOP")
master_fgn <- master %>% 
  filter(fgn_id == "FGN")
master_non_fgn <- master %>% 
  filter(fgn_id == "non-FGN")
```

#### Final exam score vs. `satm` (grouped by `sex_id`)


```r
ggplot() +
  geom_point(data = master_male, aes(x = satm, y = final), color = "blue") +
  geom_smooth(data = master_male, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "blue") +
  geom_point(data = master_female, aes(x = satm, y = final), color = "red") +
  geom_smooth(data = master_female, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "red")
```

```
## Warning: Removed 45 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 51 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 45 rows containing missing values (geom_point).
```

```
## Warning: Removed 51 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

#### Final exam score vs. `aleksikc_score` (grouped by `sex_id`)


```r
ggplot() +
  geom_point(data = master_male, aes(x = aleksikc_score, y = final), color = "blue") +
  geom_smooth(data = master_male, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "blue") +
  geom_point(data = master_female, aes(x = aleksikc_score, y = final), color = "red") +
  geom_smooth(data = master_female, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "red")
```

```
## Warning: Removed 31 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 30 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 31 rows containing missing values (geom_point).
```

```
## Warning: Removed 30 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-23-1.png)<!-- -->

#### Final exam score vs. `hs_gpa` (grouped by `sex_id`)


```r
ggplot() +
  geom_point(data = master_male, aes(x = hs_gpa, y = final), color = "blue") +
  geom_smooth(data = master_male, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "blue") +
  geom_point(data = master_female, aes(x = hs_gpa, y = final), color = "red") +
  geom_smooth(data = master_female, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "red")
```

```
## Warning: Removed 44 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 47 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 44 rows containing missing values (geom_point).
```

```
## Warning: Removed 47 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-24-1.png)<!-- -->

#### Final exam score vs. `satm` (grouped by `urm_id`)


```r
ggplot() +
  geom_point(data = master_urm, aes(x = satm, y = final), color = "red") +
  geom_smooth(data = master_urm, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_urm, aes(x = satm, y = final), color = "black") +
  geom_smooth(data = master_non_urm, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 12 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 48 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 12 rows containing missing values (geom_point).
```

```
## Warning: Removed 48 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-25-1.png)<!-- -->

#### Final exam score vs. `aleksikc_score` (grouped by `urm_id`)


```r
ggplot() +
  geom_point(data = master_urm, aes(x = aleksikc_score, y = final), color = "red") +
  geom_smooth(data = master_urm, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_urm, aes(x = aleksikc_score, y = final), color = "black") +
  geom_smooth(data = master_non_urm, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 13 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 42 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 13 rows containing missing values (geom_point).
```

```
## Warning: Removed 42 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-26-1.png)<!-- -->

#### Final exam score vs. `hs_gpa` (grouped by `urm_id`)


```r
ggplot() +
  geom_point(data = master_urm, aes(x = hs_gpa, y = final), color = "red") +
  geom_smooth(data = master_urm, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_urm, aes(x = hs_gpa, y = final), color = "black") +
  geom_smooth(data = master_non_urm, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 17 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 69 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 17 rows containing missing values (geom_point).
```

```
## Warning: Removed 69 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-27-1.png)<!-- -->

#### Final exam score vs. `satm` (grouped by `eop_id`)


```r
ggplot() +
  geom_point(data = master_eop, aes(x = satm, y = final), color = "red") +
  geom_smooth(data = master_eop, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_eop, aes(x = satm, y = final), color = "black") +
  geom_smooth(data = master_non_eop, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 19 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 77 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 19 rows containing missing values (geom_point).
```

```
## Warning: Removed 77 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-28-1.png)<!-- -->

#### Final exam score vs. `aleksikc_score` (grouped by `eop_id`)


```r
ggplot() +
  geom_point(data = master_eop, aes(x = aleksikc_score, y = final), color = "red") +
  geom_smooth(data = master_eop, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_eop, aes(x = aleksikc_score, y = final), color = "black") +
  geom_smooth(data = master_non_eop, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 24 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 37 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 24 rows containing missing values (geom_point).
```

```
## Warning: Removed 37 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-29-1.png)<!-- -->

#### Final exam score vs. `hs_gpa` (grouped by `eop_id`)


```r
ggplot() +
  geom_point(data = master_eop, aes(x = hs_gpa, y = final), color = "red") +
  geom_smooth(data = master_eop, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_eop, aes(x = hs_gpa, y = final), color = "black") +
  geom_smooth(data = master_non_eop, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 22 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 69 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 22 rows containing missing values (geom_point).
```

```
## Warning: Removed 69 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-30-1.png)<!-- -->

#### Final exam score vs. `satm` (grouped by `fgn_id`)


```r
ggplot() +
  geom_point(data = master_fgn, aes(x = satm, y = final), color = "red") +
  geom_smooth(data = master_fgn, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_fgn, aes(x = satm, y = final), color = "black") +
  geom_smooth(data = master_non_fgn, aes(x = satm, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 41 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 52 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 41 rows containing missing values (geom_point).
```

```
## Warning: Removed 52 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-31-1.png)<!-- -->

#### Final exam score vs. `aleksikc_score` (grouped by `fgn_id`)


```r
ggplot() +
  geom_point(data = master_fgn, aes(x = aleksikc_score, y = final), color = "red") +
  geom_smooth(data = master_fgn, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_fgn, aes(x = aleksikc_score, y = final), color = "black") +
  geom_smooth(data = master_non_fgn, aes(x = aleksikc_score, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 31 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 30 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 31 rows containing missing values (geom_point).
```

```
## Warning: Removed 30 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-32-1.png)<!-- -->

#### Final exam score vs. `hs_gpa` (grouped by `fgn_id`)


```r
ggplot() +
  geom_point(data = master_fgn, aes(x = hs_gpa, y = final), color = "red") +
  geom_smooth(data = master_fgn, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "red") +
  geom_point(data = master_non_fgn, aes(x = hs_gpa, y = final), color = "black") +
  geom_smooth(data = master_non_fgn, aes(x = hs_gpa, y = final), method = 'lm', formula = y ~ x, color = "black")
```

```
## Warning: Removed 40 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 44 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 40 rows containing missing values (geom_point).
```

```
## Warning: Removed 44 rows containing missing values (geom_point).
```

![](data-visualization_files/figure-html/unnamed-chunk-33-1.png)<!-- -->

### Boxplots

#### `sex_id` distribution between control and experimental years


```r
ggplot(data = master, aes(x = sex_id)) +
  geom_bar() +
  facet_wrap(~ exp)
```

![](data-visualization_files/figure-html/unnamed-chunk-34-1.png)<!-- -->

#### `urm_id` distribution between control and experimental years


```r
ggplot(data = master, aes(x = urm_id)) +
  geom_bar() +
  facet_wrap(~ exp)
```

![](data-visualization_files/figure-html/unnamed-chunk-35-1.png)<!-- -->

#### `eop_id` distribution between control and experimental years


```r
ggplot(data = master, aes(x = eop_id)) +
  geom_bar() +
  facet_wrap(~ exp)
```

![](data-visualization_files/figure-html/unnamed-chunk-36-1.png)<!-- -->

#### `fgn_id` distribution between control and experimental years


```r
ggplot(data = master, aes(x = fgn_id)) +
  geom_bar() +
  facet_wrap(~ exp)
```

![](data-visualization_files/figure-html/unnamed-chunk-37-1.png)<!-- -->
