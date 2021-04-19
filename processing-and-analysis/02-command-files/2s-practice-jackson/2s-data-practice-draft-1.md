---
# title: "Two-Stage Data Practice"
# author: "Jackson Hughes"
date: "4/11/2021"
output: 
  html_document:
    keep_md: yes
---



### Set working directories


```r
proj_dir = here()
importable_data_dir = here("processing-and-analysis", "importable-data")
analysis_data_dir = here("processing-and-analysis", "analysis-data")
metadata_dir = here("original-data", "metadata")
setwd("G:/Shared drives/CEG Two-Stage Exams Analysis (4 REAL)/two-stage-2/processing-and-analysis/02-command-files/2s-practice-jackson")
```

### Import dataset

**Note:** There seems to be trouble with uploading the data through google file stream and the "here" package, so the data will be uploaded through my local device.


```r
master = read.csv("C:/Users/jacks/Downloads/master_2s_small_deidentified (DO NOT TOUCH).csv")
```

## Relationship 1: Scores for Exam 1, 2, and Final by:
* `course_fullid`
* `course_fullid` and `ver`
* `course_fullid` and demographic `eop_id`, `sex_id`, `fgn_id`, `urm_id` (all separate)

### Remove duplicate rows of student data:


```r
master_extras_removed = subset(master, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))

master_unique = unique(master_extras_removed, incomparables = FALSE)
```

### Faceted histograms of scores for exam 1, 2, and final by `course_fullid`:


```r
ggplot(master_unique, aes(x = exam1)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 1 Score", y = "Number of Students", title = "Exam 1 Score Distributions by Course ID") +
  facet_wrap(~ course_fullid, nrow = 2)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam2)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 2 Score", y = "Number of Students", title = "Exam 2 Score Distributions by Course ID") +
  facet_wrap(~ course_fullid, nrow = 2)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-4-2.png)<!-- -->

```r
ggplot(master_unique, aes(x = finalexam)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Final Exam Score", y = "Number of Students", title = "Final Exam Score Distributions by Course ID") +
  facet_wrap(~ course_fullid, nrow = 2)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-4-3.png)<!-- -->

### Boxplots of Scores for Exam 1, 2, and final by `course_fullid`:


```r
ggplot(master_unique, aes(x = course_fullid, y = exam1)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 1 Score", title = "Exam 1 Score Distributions by Course ID")
```

```
## Warning: Removed 66 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam2)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 2 Score", title = "Exam 2 Score Distributions by Course ID")
```

```
## Warning: Removed 74 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-5-2.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = finalexam)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Final Exam Score", title = "Final Exam Score Distributions by Course ID")
```

```
## Warning: Removed 107 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-5-3.png)<!-- -->


### Faceted Histograms of Scores for Exam 1, 2, and final by `course_fullid` and `ver`:


```r
ggplot(master_unique, aes(x = exam1)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 1 Score", y = "Number of Students", title = "Exam 1 Score Distributions by Course ID and Test Version") +
  facet_wrap(~ course_fullid + ver, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam2)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 2 Score", y = "Number of Students", title = "Exam 2 Score Distributions by Course ID and Test Version") +
  facet_wrap(~ course_fullid + ver, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-6-2.png)<!-- -->

```r
ggplot(master_unique, aes(x = finalexam)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Final Exam Score", y = "Number of Students", title = "Final Exam Score Distributions by Course ID and Test Version") +
  facet_wrap(~ course_fullid + ver, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-6-3.png)<!-- -->

### Faceted Boxplots of Scores for Exam 1, 2, and final by `course_fullid` and `ver`:


```r
ggplot(master_unique, aes(x = course_fullid, y = exam1)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 1 Score", title = "Exam 1 Score Distributions by Course ID") +
  facet_wrap(~ ver, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam2)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 2 Score", title = "Exam 2 Score Distributions by Course ID") +
  facet_wrap(~ ver, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-7-2.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = finalexam)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Final Exam Score", title = "Final Exam Score Distributions by Course ID") +
  facet_wrap(~ ver, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-7-3.png)<!-- -->


### Faceted Histograms of Scores for Exam 1, 2, and Final by `course_fullid` and demographic `eop_id`, `sex_id`, `fgn_id`, `urm_id`:


```r
ggplot(master_unique, aes(x = exam1)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 1 Score", y = "Number of Students", title = "Exam 1 Score Distributions by Course ID and EOP ID") +
  facet_wrap(~ course_fullid + eop_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam2)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 2 Score", y = "Number of Students", title = "Exam 2 Score Distributions by Course ID and EOP ID") +
  facet_wrap(~ course_fullid + eop_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-2.png)<!-- -->

```r
ggplot(master_unique, aes(x = finalexam)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Final Exam Score", y = "Number of Students", title = "Final Exam Score Distributions by Course ID and EOP ID") +
  facet_wrap(~ course_fullid + eop_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-3.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam1)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 1 Score", y = "Number of Students", title = "Exam 1 Score Distributions by Course ID and Sex") +
  facet_wrap(~ course_fullid + sex_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-4.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam2)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 2 Score", y = "Number of Students", title = "Exam 2 Score Distributions by Course ID and Sex") +
  facet_wrap(~ course_fullid + sex_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-5.png)<!-- -->

```r
ggplot(master_unique, aes(x = finalexam)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Final Exam Score", y = "Number of Students", title = "Final Exam Score Distributions by Course ID and Sex") +
  facet_wrap(~ course_fullid + sex_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-6.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam1)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 1 Score", y = "Number of Students", title = "Exam 1 Score Distributions by Course ID and FGN ID") +
  facet_wrap(~ course_fullid + fgn_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-7.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam2)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 2 Score", y = "Number of Students", title = "Exam 2 Score Distributions by Course ID and FGN ID") +
  facet_wrap(~ course_fullid + fgn_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-8.png)<!-- -->

```r
ggplot(master_unique, aes(x = finalexam)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Final Exam Score", y = "Number of Students", title = "Final Exam Score Distributions by Course ID and FGN ID") +
  facet_wrap(~ course_fullid + fgn_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-9.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam1)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 1 Score", y = "Number of Students", title = "Exam 1 Score Distributions by Course ID and URM ID") +
  facet_wrap(~ course_fullid + urm_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-10.png)<!-- -->

```r
ggplot(master_unique, aes(x = exam2)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Exam 2 Score", y = "Number of Students", title = "Exam 2 Score Distributions by Course ID and URM ID") +
  facet_wrap(~ course_fullid + urm_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-11.png)<!-- -->

```r
ggplot(master_unique, aes(x = finalexam)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Final Exam Score", y = "Number of Students", title = "Final Exam Score Distributions by Course ID and URM ID") +
  facet_wrap(~ course_fullid + urm_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_bin).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-8-12.png)<!-- -->

### Faceted Histograms of Scores for Exam 1, 2, and Final by `course_fullid` and demographic `eop_id`, `sex_id`, `fgn_id`, `urm_id`:


```r
ggplot(master_unique, aes(x = course_fullid, y = exam1)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 1 Score", title = "Exam 1 Score Distributions by Course ID and EOP ID") +
  facet_wrap(~ eop_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam2)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 2 Score", title = "Exam 2 Score Distributions by Course ID and EOP ID") +
  facet_wrap(~ eop_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-2.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = finalexam)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Final Exam Score", title = "Final Exam Score Distributions by Course ID and EOP ID") +
  facet_wrap(~ eop_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-3.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam1)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 1 Score", title = "Exam 1 Score Distributions by Course ID and Sex") +
  facet_wrap(~ sex_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-4.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam2)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 2 Score", title = "Exam 2 Score Distributions by Course ID and Sex") +
  facet_wrap(~ sex_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-5.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = finalexam)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Final Exam Score", title = "Final Exam Score Distributions by Course ID and Sex") +
  facet_wrap(~ sex_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-6.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam1)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 1 Score", title = "Exam 1 Score Distributions by Course ID and FGN ID") +
  facet_wrap(~ fgn_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-7.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam2)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 2 Score", title = "Exam 2 Score Distributions by Course ID and FGN ID") +
  facet_wrap(~ fgn_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-8.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = finalexam)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Final Exam Score", title = "Final Exam Score Distributions by Course ID and FGN ID") +
  facet_wrap(~ fgn_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-9.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam1)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 1 Score", title = "Exam 1 Score Distributions by Course ID and URM ID") +
  facet_wrap(~ urm_id, nrow = 3)
```

```
## Warning: Removed 66 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-10.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = exam2)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Exam 2 Score", title = "Exam 2 Score Distributions by Course ID and URM ID") +
  facet_wrap(~ urm_id, nrow = 3)
```

```
## Warning: Removed 74 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-11.png)<!-- -->

```r
ggplot(master_unique, aes(x = course_fullid, y = finalexam)) +
  geom_boxplot() +
  labs(x = "Course ID", y = "Final Exam Score", title = "Final Exam Score Distributions by Course ID and URM ID") +
  facet_wrap(~ urm_id, nrow = 3)
```

```
## Warning: Removed 107 rows containing non-finite values (stat_boxplot).
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-9-12.png)<!-- -->

## Relationship 2: Scores for each of the A17 individual quiz scores (`q1_ind`) by:
* `ver`
* `eop_id`, `sex_id`, `fgn_id`, `urm_id`

### Condense dataset to only include Autumn 2017 data:


```r
fall_2017 = master %>% 
  filter(course_fullid == "CHEM_142_A_2017_4" | course_fullid == "CHEM_142_B_2017_4")
fall_2017 = na.omit(fall_2017)
```

### Faceted histograms of `q1_ind` by `ver`:


```r
ggplot(fall_2017, aes(x = q1_ind)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Individual Quiz Score", y = "Number of Students", title = "Individual Quiz Score Distributions by Version") +
  facet_wrap(~ ver, nrow = 2)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

### Boxplots of `q1_ind` by `ver`:


```r
ggplot(fall_2017, aes(x = ver, y = q1_ind)) +
  geom_boxplot() +
  labs(x = "Version", y = "Individual Quiz Scores", title = "Boxplots of Individual Quiz Scores by Version")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

### Faceted Histograms of `q1_ind` by `eop_id`, `sex_id`, `fgn_id`, and `urm_id`: 


```r
ggplot(fall_2017, aes(x = q1_ind)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Individual Quiz Score", y = "number of students", title = "Distributions of Individual Quiz Scores by EOP ID") +
  facet_wrap(~ eop_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_ind)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Individual Quiz Score", y = "number of students", title = "Distributions of Individual Quiz Scores by Sex") +
  facet_wrap(~ sex_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-13-2.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_ind)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Individual Quiz Score", y = "number of students", title = "Distributions of Individual Quiz Scores by FGN ID") +
  facet_wrap(~ fgn_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-13-3.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_ind)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Individual Quiz Score", y = "number of students", title = "Distributions of Individual Quiz Scores by URM ID") +
  facet_wrap(~ urm_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-13-4.png)<!-- -->

### Boxplots of `q1_ind` by `eop_id`, `sex_id`, `fgn_id`, and `urm_id`: 


```r
ggplot(fall_2017, aes(x = eop_id, y = q1_ind)) +
  geom_boxplot() +
  labs(x = "EOP ID", y = "Individual Quiz Scores", title = "Distributions of Individual Quiz Scores by EOP ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

```r
ggplot(fall_2017, aes(x = sex_id, y = q1_ind)) +
  geom_boxplot() +
  labs(x = "Sex", y = "Individual Quiz Scores", title = "Distributions of Individual Quiz Scores by Sex")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-14-2.png)<!-- -->

```r
ggplot(fall_2017, aes(x = fgn_id, y = q1_ind)) +
  geom_boxplot() +
  labs(x = "FGN ID", y = "Individual Quiz Scores", title = "Distributions of Individual Quiz Scores by FGN ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-14-3.png)<!-- -->

```r
ggplot(fall_2017, aes(x = urm_id, y = q1_ind)) +
  geom_boxplot() +
  labs(x = "URM ID", y = "Individual Quiz Scores", title = "Distributions of Individual Quiz Scores by URM ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-14-4.png)<!-- -->

## Relationship 3: Scores for each of the A17 group quiz scores (`q1_grp`) by:
* `ver`
* `eop_id`, `sex_id`, `fgn_id`, `urm_id`

### Faceted histograms of `q1_grp` by `ver`:


```r
ggplot(fall_2017, aes(x = q1_grp)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Group Quiz Score", y = "Number of Students", title = "Group Quiz Score Distributions by Version") +
  facet_wrap(~ ver, nrow = 2)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

### Boxplots of `q1_grp` by `ver`:


```r
ggplot(fall_2017, aes(x = ver, y = q1_grp)) +
  geom_boxplot() +
  labs(x = "Version", y = "Group Quiz Scores", title = "Boxplots of Group Quiz Scores by Version")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

### Faceted Histograms of `q1_grp` by `eop_id`, `sex_id`, `fgn_id`, and `urm_id`: 


```r
ggplot(fall_2017, aes(x = q1_grp)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Group Quiz Score", y = "number of students", title = "Distributions of Group Quiz Scores by EOP ID") +
  facet_wrap(~ eop_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_grp)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Group Quiz Score", y = "number of students", title = "Distributions of Group Quiz Scores by Sex") +
  facet_wrap(~ sex_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-17-2.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_grp)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Group Quiz Score", y = "number of students", title = "Distributions of Group Quiz Scores by FGN ID") +
  facet_wrap(~ fgn_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-17-3.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_grp)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Group Quiz Score", y = "number of students", title = "Distributions of Group Quiz Scores by URM ID") +
  facet_wrap(~ urm_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-17-4.png)<!-- -->

### Boxplots of `q1_grp` by `eop_id`, `sex_id`, `fgn_id`, and `urm_id`: 


```r
ggplot(fall_2017, aes(x = eop_id, y = q1_grp)) +
  geom_boxplot() +
  labs(x = "EOP ID", y = "Group Quiz Scores", title = "Distributions of Group Quiz Scores by EOP ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-18-1.png)<!-- -->

```r
ggplot(fall_2017, aes(x = sex_id, y = q1_grp)) +
  geom_boxplot() +
  labs(x = "Sex", y = "Group Quiz Scores", title = "Distributions of Group Quiz Scores by Sex")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-18-2.png)<!-- -->

```r
ggplot(fall_2017, aes(x = fgn_id, y = q1_grp)) +
  geom_boxplot() +
  labs(x = "FGN ID", y = "Group Quiz Scores", title = "Distributions of Group Quiz Scores by FGN ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-18-3.png)<!-- -->

```r
ggplot(fall_2017, aes(x = urm_id, y = q1_grp)) +
  geom_boxplot() +
  labs(x = "URM ID", y = "Group Quiz Scores", title = "Distributions of Group Quiz Scores by URM ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-18-4.png)<!-- -->

## Relationship 4: Scores for each of the A17 total quiz scores (`q1_total`) by:
* `ver`
* `eop_id`, `sex_id`, `fgn_id`, `urm_id`

### Faceted histograms of `q1_total` by `ver`:


```r
ggplot(fall_2017, aes(x = q1_total)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Total Quiz Score", y = "Number of Students", title = "Total Quiz Score Distributions by Version") +
  facet_wrap(~ ver, nrow = 2)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-19-1.png)<!-- -->

### Boxplots of `q1_total` by `ver`:


```r
ggplot(fall_2017, aes(x = ver, y = q1_total)) +
  geom_boxplot() +
  labs(x = "Version", y = "Total Quiz Scores", title = "Boxplots of Total Quiz Scores by Version")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-20-1.png)<!-- -->

### Faceted Histograms of `q1_total` by `eop_id`, `sex_id`, `fgn_id`, and `urm_id`: 


```r
ggplot(fall_2017, aes(x = q1_total)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Total Quiz Score", y = "number of students", title = "Distributions of Total Quiz Scores by EOP ID") +
  facet_wrap(~ eop_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-21-1.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_total)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Total Quiz Score", y = "number of students", title = "Distributions of Total Quiz Scores by Sex") +
  facet_wrap(~ sex_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-21-2.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_total)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Total Quiz Score", y = "number of students", title = "Distributions of Total Quiz Scores by FGN ID") +
  facet_wrap(~ fgn_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-21-3.png)<!-- -->

```r
ggplot(fall_2017, aes(x = q1_total)) +
  geom_histogram(binwidth = 5, color = "white") +
  labs(x = "Total Quiz Score", y = "number of students", title = "Distributions of Total Quiz Scores by URM ID") +
  facet_wrap(~ urm_id, nrow = 1)
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-21-4.png)<!-- -->

### Boxplots of `q1_total` by `eop_id`, `sex_id`, `fgn_id`, and `urm_id`: 


```r
ggplot(fall_2017, aes(x = eop_id, y = q1_total)) +
  geom_boxplot() +
  labs(x = "EOP ID", y = "Total Quiz Scores", title = "Distributions of Total Quiz Scores by EOP ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

```r
ggplot(fall_2017, aes(x = sex_id, y = q1_total)) +
  geom_boxplot() +
  labs(x = "Sex", y = "Total Quiz Scores", title = "Distributions of Total Quiz Scores by Sex")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-22-2.png)<!-- -->

```r
ggplot(fall_2017, aes(x = fgn_id, y = q1_total)) +
  geom_boxplot() +
  labs(x = "FGN ID", y = "Total Quiz Scores", title = "Distributions of Total Quiz Scores by FGN ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-22-3.png)<!-- -->

```r
ggplot(fall_2017, aes(x = urm_id, y = q1_total)) +
  geom_boxplot() +
  labs(x = "URM ID", y = "Total Quiz Scores", title = "Distributions of Total Quiz Scores by URM ID")
```

![](2s-data-practice-draft-1_files/figure-html/unnamed-chunk-22-4.png)<!-- -->

## Summary Statistics:

### Exam Scores by `course_fullid`:

```r
fall_2016_A = filter(master_unique, course_fullid == "CHEM_142_A_2016_4")
fall_2016_B = filter(master_unique, course_fullid == "CHEM_142_B_2016_4")
fall_2017_A = filter(master_unique, course_fullid == "CHEM_142_A_2017_4")
fall_2017_B = filter(master_unique, course_fullid == "CHEM_142_B_2017_4")

summary_exam1_fall_2016_A = fall_2016_A %>% 
  summarize(mean = mean(exam1), std_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_exam2_fall_2016_A = fall_2016_A %>% 
  summarize(mean = mean(exam2), std_dev = sd(exam2), min = min(exam2), max = max(exam2))
summary_exam2_fall_2016_A
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_final_fall_2016_A = fall_2016_A %>% 
  summarize(mean = mean(finalexam), std_dev = sd(finalexam), min = min(finalexam), max = max(finalexam))
summary_final_fall_2016_A
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_exam1_fall_2016_B = fall_2016_B %>% 
  summarize(mean = mean(exam1), std_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_B
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_exam2_fall_2016_B = fall_2016_B %>% 
  summarize(mean = mean(exam2), std_dev = sd(exam2), min = min(exam2), max = max(exam2))
summary_exam2_fall_2016_B
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_final_fall_2016_B = fall_2016_B %>% 
  summarize(mean = mean(finalexam), std_dev = sd(finalexam), min = min(finalexam), max = max(finalexam))
summary_final_fall_2016_B
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_exam1_fall_2017_A = fall_2017_A %>% 
  summarize(mean = mean(exam1), std_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2017_A
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_exam2_fall_2017_A = fall_2017_A %>% 
  summarize(mean = mean(exam2), std_dev = sd(exam2), min = min(exam2), max = max(exam2))
summary_exam2_fall_2017_A
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_final_fall_2017_A = fall_2017_A %>% 
  summarize(mean = mean(finalexam), std_dev = sd(finalexam), min = min(finalexam), max = max(finalexam))
summary_final_fall_2017_A
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_exam1_fall_2017_B = fall_2017_B %>% 
  summarize(mean = mean(exam1), std_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2017_B
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_exam2_fall_2017_B = fall_2017_B %>% 
  summarize(mean = mean(exam2), std_dev = sd(exam2), min = min(exam2), max = max(exam2))
summary_exam2_fall_2017_B
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

```r
summary_final_fall_2017_B = fall_2017_B %>% 
  summarize(mean = mean(finalexam), std_dev = sd(finalexam), min = min(finalexam), max = max(finalexam))
summary_final_fall_2017_B
```

```
##   mean std_dev min max
## 1   NA      NA  NA  NA
```

### Exam Scores by `course_fullid` and `eop_id`:

```r
summary_exam1_fall_2016_A_eop = fall_2016_A %>%
  filter(eop_id == "EOP") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A_eop
```

```
##      mean   st_dev min max
## 1 60.6485 19.27863  10  98
```

```r
summary_exam1_fall_2016_A_non_eop = fall_2016_A %>%
  filter(eop_id == "non-EOP") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A_non_eop
```

```
##       mean   st_dev min max
## 1 70.60532 16.60899  13 103
```

### Exam Scores by `course_fullid` and `sex_id`:

```r
summary_exam1_fall_2016_A_male = fall_2016_A %>%
  filter(sex_id == "Male") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A_male
```

```
##       mean  st_dev min max
## 1 71.32236 16.9691  10 103
```

```r
summary_exam1_fall_2016_A_female = fall_2016_A %>%
  filter(eop_id == "Female") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
```

```
## Warning in min(exam1): no non-missing arguments to min; returning Inf
```

```
## Warning in max(exam1): no non-missing arguments to max; returning -Inf
```

```r
summary_exam1_fall_2016_A_female
```

```
##   mean st_dev min  max
## 1  NaN     NA Inf -Inf
```

### Exam Scores by `course_fullid` and `fgn_id`:

```r
summary_exam1_fall_2016_A_fgn = fall_2016_A %>%
  filter(fgn_id == "FGN") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A_fgn
```

```
##       mean  st_dev min max
## 1 62.73496 18.8128  10  98
```

```r
summary_exam1_fall_2016_A_non_fgn = fall_2016_A %>%
  filter(fgn_id == "non-FGN") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A_non_fgn
```

```
##       mean   st_dev min max
## 1 71.59851 16.21766  28 103
```

### Exam scores by `course_fullid` and `urm_id`:

```r
summary_exam1_fall_2016_A_urm = fall_2016_A %>%
  filter(urm_id == "URM") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A_urm
```

```
##       mean   st_dev min max
## 1 61.25455 21.17138  10  98
```

```r
summary_exam1_fall_2016_A_non_urm = fall_2016_A %>%
  filter(urm_id == "non-URM") %>% 
  summarize(mean = mean(exam1), st_dev = sd(exam1), min = min(exam1), max = max(exam1))
summary_exam1_fall_2016_A_non_urm
```

```
##       mean   st_dev min max
## 1 69.69822 16.86576  13 103
```

## Notes:
* Need to figure out how to remove the NAs
* Still need to include summary stats for the quiz data
* The dataset is still being uploaded through my local device (I forgot how to upload properly through the file stream)
