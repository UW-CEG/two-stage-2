---
title: "Model_Analysis_New_Data"
author: "Ganling"
date: "8/30/2021"
output: 
  html_document:
    keep_md: yes
---

## Set working directories

```r
original_data_dir   <- here("original-data", "/")
importable_data_dir <- here("processing-and-analysis", "01-importable-data", "/")
analysis_data_dir   <- here("processing-and-analysis", "03-analysis-data", "/")
```

## Import data set
### Copy data from `original-data` to `01-importable-data`

```r
copy_from <- paste0(original_data_dir, 
 "two_stage_master_wide_deid.rds")
copy_to <- paste0(importable_data_dir, "two_stage_master_wide_deid.rds")
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```
### Import master data set

```r
df <- readRDS(paste0(importable_data_dir, "two_stage_master_wide_deid.rds"))
```
## Regression Analysis
### Select columns

```r
df_1 <- df %>%
  select(two_stage_id, class.x, class.y, qtr, course_fullid, ta_sect, exp, exam1_c, exam2_c, final_c, sex_id, urm_id, eop_id, fgn_id)
```










