---
# title: "data_2s_summary"
# author: "Ganling"
# date: "4/19/2021"
output: 
  html_document:
    keep_md: yes
---

Load the packages

```r
library(here)
```

```
## here() starts at G:/Shared drives/CEG Two-Stage Exams Analysis/Colleen/two-stage-2
```

```r
library(tidyverse)
```

```
## -- Attaching packages ------------------------------------------------------- tidyverse 1.3.0 --
```

```
## v ggplot2 3.3.2     v purrr   0.3.4
## v tibble  3.0.3     v dplyr   1.0.1
## v tidyr   1.1.1     v stringr 1.4.0
## v readr   1.3.1     v forcats 0.5.0
```

```
## Warning: package 'ggplot2' was built under R version 4.0.3
```

```
## -- Conflicts ---------------------------------------------------------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(skimr)
```

```
## Warning: package 'skimr' was built under R version 4.0.3
```

```r
library(moderndive)
```

```
## Warning: package 'moderndive' was built under R version 4.0.3
```

```r
library(ggplot2)
library(dplyr)
library(readr)
```

# Import data 

## Read the files using `here` package

Create directory strings.

```r
proj_dir <- here()

# CFC: add original data directory string 
original_data_dir <- here("original-data", "/")

# CFC: add numbers to importable and analysis data directory strings
importable_data_dir <- here("processing-and-analysis", "01-importable-data", "/")
analysis_data_dir <- here("processing-and-analysis", "03-analysis-data", "/")
metadata_dir <- here("original-data", "metadata", "/")
```

Copy data from `original-data` to `01-importable-data`

```r
# CFC: Copy rds file from the `original_data_dir` to the `importable_data_dir`.
copy_from <- paste0(original_data_dir, "master_2s_small_deidentified.rds")
copy_to <- paste0(importable_data_dir, "master_2s_small_deidentified.rds")

# CFC: If the file already exists in the target directory, the file will not be 
# overwritten and this command will return `FALSE`.
file.copy(copy_from, copy_to)
```

```
## [1] FALSE
```

Read in data.

```r
# `master_2s_small_deidentified (DO NOT TOUCH)` <- readRDS("G:/Shared drives/CEG R Coding Practice/Ganling/two-stage-2/processing-and-analysis/01-importable-data/dataset/master_2s_small_deidentified (DO NOT TOUCH).rds")

# data_2s <- `master_2s_small_deidentified (DO NOT TOUCH)`

# CFC: IMport data set from `importable_data_dir`
data_2s <- readRDS(paste0(importable_data_dir, "master_2s_small_deidenitified.rds"))
```

### Before starting to make historgrams and boxplots, I would like to remove all the duplicating rows and make a new data frame

```r
data_2s_extras_removed = subset(data_2s, select = -c(item_num, stud_ans, exam_key, corr, bloom_rating, complexity_rating_mean, complexity_rating_median, item_code, stem))

data_2s_unique = unique(data_2s_extras_removed, incomparables = FALSE)
View(data_2s_unique)

data_2s_exam_score <- data_2s_unique %>% select(exam1, exam2, finalexam, course_fullid, ver, sex_id, urm_id, eop_id, fgn_id)

data_2s_exam_score_true <- na.omit(data_2s_exam_score)
```

# Now I would like to graph the histogram and boxplots for exam scores and course_id

```r
#histogram for coruse_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam1)) + geom_histogram(color = "white") + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID")) + facet_wrap(~ course_fullid, nrow = 1 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam2)) + geom_histogram(color = "white") + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID")) + facet_wrap(~ course_fullid, nrow = 1 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-6-2.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= finalexam)) + geom_histogram(color = "white") + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID")) + facet_wrap(~ course_fullid, nrow = 1 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-6-3.png)<!-- -->

```r
#boxplot for course_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= course_fullid, y= exam1)) + geom_boxplot() + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID")) + facet_wrap(~ course_fullid, nrow = 1 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-6-4.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= course_fullid, y= exam2)) + geom_boxplot() + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID")) + facet_wrap(~ course_fullid, nrow = 1 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-6-5.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= course_fullid, y= finalexam)) + geom_boxplot() + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID")) + facet_wrap(~ course_fullid, nrow = 1 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-6-6.png)<!-- -->

# Next, we want to look at exam score, course id and version 

```r
#histogram for coruse_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam1)) + geom_histogram(color = "white") + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and version")) + facet_wrap(~ course_fullid + ver, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam2)) + geom_histogram(color = "white") + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and version")) + facet_wrap(~ course_fullid + ver, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-7-2.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= finalexam)) + geom_histogram(color = "white") + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and version")) + facet_wrap(~ course_fullid + ver, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-7-3.png)<!-- -->

```r
#boxplot for course_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= course_fullid, y= exam1)) + geom_boxplot() + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and version")) + facet_wrap(~ course_fullid + ver, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-7-4.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= course_fullid, y= exam2)) + geom_boxplot() + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and version")) + facet_wrap(~ course_fullid + ver, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-7-5.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= course_fullid, y= finalexam)) + geom_boxplot() + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and version")) + facet_wrap(~ course_fullid + ver, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-7-6.png)<!-- -->

# course id and eop status

```r
#histogram for coruse_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam1)) + geom_histogram(color = "white") + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and eop")) + facet_wrap(~ course_fullid + eop_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam2)) + geom_histogram(color = "white") + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and eop")) + facet_wrap(~ course_fullid + eop_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-8-2.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= finalexam)) + geom_histogram(color = "white") + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and eop")) + facet_wrap(~ course_fullid + eop_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-8-3.png)<!-- -->

```r
#boxplot for course_id
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam1)) + geom_boxplot() + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and eop")) + facet_wrap(~ course_fullid + eop_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-8-4.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam2)) + geom_boxplot() + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and eop")) + facet_wrap(~ course_fullid + eop_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-8-5.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= finalexam)) + geom_boxplot() + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and eop")) + facet_wrap(~ course_fullid + eop_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-8-6.png)<!-- -->

# course id and sex id

```r
#histogram for coruse_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam1)) + geom_histogram(color = "white") + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and sex")) + facet_wrap(~ course_fullid + sex_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam2)) + geom_histogram(color = "white") + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and sex")) + facet_wrap(~ course_fullid + sex_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-9-2.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= finalexam)) + geom_histogram(color = "white") + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and sex")) + facet_wrap(~ course_fullid + sex_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-9-3.png)<!-- -->

```r
#boxplot for course_id
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam1)) + geom_boxplot() + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and sex")) + facet_wrap(~ course_fullid + sex_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-9-4.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam2)) + geom_boxplot() + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and sex")) + facet_wrap(~ course_fullid + sex_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-9-5.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= finalexam)) + geom_boxplot() + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and sex")) + facet_wrap(~ course_fullid + sex_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-9-6.png)<!-- -->

# course id and fgn status

```r
#histogram for coruse_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam1)) + geom_histogram(color = "white") + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and fgn")) + facet_wrap(~ course_fullid + fgn_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam2)) + geom_histogram(color = "white") + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and fgn")) + facet_wrap(~ course_fullid + fgn_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-10-2.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= finalexam)) + geom_histogram(color = "white") + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and fgn")) + facet_wrap(~ course_fullid + fgn_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-10-3.png)<!-- -->

```r
#boxplot for course_id
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam1)) + geom_boxplot() + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and fgn")) + facet_wrap(~ course_fullid + fgn_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-10-4.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam2)) + geom_boxplot() + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and fgn")) + facet_wrap(~ course_fullid + fgn_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-10-5.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= finalexam)) + geom_boxplot() + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and fgn")) + facet_wrap(~ course_fullid + fgn_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-10-6.png)<!-- -->

# urm status and course id

```r
#histogram for coruse_id
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam1)) + geom_histogram(color = "white") + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and urm")) + facet_wrap(~ course_fullid + urm_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= exam2)) + geom_histogram(color = "white") + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and urm")) + facet_wrap(~ course_fullid + urm_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-11-2.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(x= finalexam)) + geom_histogram(color = "white") + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and urm")) + facet_wrap(~ course_fullid + urm_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-11-3.png)<!-- -->

```r
#boxplot for course_id
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam1)) + geom_boxplot() + labs(x = "Exam 1 score", y= "number of students", title = ("Exam 1 score Distributions by Course ID and urm")) + facet_wrap(~ course_fullid + urm_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-11-4.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= exam2)) + geom_boxplot() + labs(x = "Exam 2 score", y= "number of students", title = ("Exam 2 score Distributions by Course ID and urm")) + facet_wrap(~ course_fullid + urm_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-11-5.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true, mapping = aes(y= finalexam)) + geom_boxplot() + labs(x = "Final exam score", y= "number of students", title = ("Final exam score Distributions by Course ID and urm")) + facet_wrap(~ course_fullid + urm_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-11-6.png)<!-- -->

# Now I want to look at A17 individual quiz score by ver, eop_id, sex_id, fgn_id and urm_id

```r
#filter out the right data set
data_2s_exam_score_true_A17 <- data_2s_unique %>% filter(course_fullid %in% c("CHEM_142_A_2017_4")) %>% select(q1_ind, q1_grp, q1_total, course_fullid, ver, eop_id, sex_id, fgn_id, urm_id)

ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_ind)) + geom_histogram(color = "white") + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by version during A17")) + facet_wrap(~ ver, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 41 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

```r
#by eop
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_ind)) + geom_histogram(color = "white") + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by eop during A17")) + facet_wrap(~ eop_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 41 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-2.png)<!-- -->

```r
#by sex
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_ind)) + geom_histogram(color = "white") + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by sex during A17")) + facet_wrap(~ sex_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 41 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-3.png)<!-- -->

```r
#by fgn
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_ind)) + geom_histogram(color = "white") + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by first generation status during A17")) + facet_wrap(~ fgn_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 41 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-4.png)<!-- -->

```r
#by urm
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_ind)) + geom_histogram(color = "white") + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by representation during A17")) + facet_wrap(~ urm_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 41 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-5.png)<!-- -->

```r
##boxplots
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_ind)) + geom_boxplot() + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by version during A17")) + facet_wrap(~ ver, nrow = 3 )
```

```
## Warning: Removed 41 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-6.png)<!-- -->

```r
#by eop
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_ind)) + geom_boxplot() + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by eop during A17")) + facet_wrap(~ eop_id, nrow = 3 )
```

```
## Warning: Removed 41 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-7.png)<!-- -->

```r
#by sex
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_ind)) + geom_boxplot() + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by sex during A17")) + facet_wrap(~ sex_id, nrow = 3 )
```

```
## Warning: Removed 41 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-8.png)<!-- -->

```r
#by fgn
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_ind)) + geom_boxplot() + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by first generation status during A17")) + facet_wrap(~ fgn_id, nrow = 3 )
```

```
## Warning: Removed 41 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-9.png)<!-- -->

```r
#by urm
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_ind)) + geom_boxplot() + labs(x = "Quiz 1 Individual score", y= "number of students", title = ("Quiz 1 individual score Distributions by representation during A17")) + facet_wrap(~ urm_id, nrow = 3 )
```

```
## Warning: Removed 41 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-12-10.png)<!-- -->


# Now I want to look at A17 individual quiz score by ver, eop_id, sex_id, fgn_id and urm_id

```r
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_grp)) + geom_histogram(color = "white") + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by version during A17")) + facet_wrap(~ ver, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 28 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
#by eop
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_grp)) + geom_histogram(color = "white") + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by eop during A17")) + facet_wrap(~ eop_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 28 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-2.png)<!-- -->

```r
#by sex
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_grp)) + geom_histogram(color = "white") + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by sex during A17")) + facet_wrap(~ sex_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 28 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-3.png)<!-- -->

```r
#by fgn
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_grp)) + geom_histogram(color = "white") + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by first generation status during A17")) + facet_wrap(~ fgn_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 28 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-4.png)<!-- -->

```r
#by urm
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_grp)) + geom_histogram(color = "white") + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by representation during A17")) + facet_wrap(~ urm_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 28 rows containing non-finite values (stat_bin).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-5.png)<!-- -->

```r
##boxplots
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_grp)) + geom_boxplot() + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by version during A17")) + facet_wrap(~ ver, nrow = 3 )
```

```
## Warning: Removed 28 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-6.png)<!-- -->

```r
#by eop
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_grp)) + geom_boxplot() + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by eop during A17")) + facet_wrap(~ eop_id, nrow = 3 )
```

```
## Warning: Removed 28 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-7.png)<!-- -->

```r
#by sex
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_grp)) + geom_boxplot() + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by sex during A17")) + facet_wrap(~ sex_id, nrow = 3 )
```

```
## Warning: Removed 28 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-8.png)<!-- -->

```r
#by fgn
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_grp)) + geom_boxplot() + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by first generation status during A17")) + facet_wrap(~ fgn_id, nrow = 3 )
```

```
## Warning: Removed 28 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-9.png)<!-- -->

```r
#by urm
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_grp)) + geom_boxplot() + labs(x = "Quiz 1 group score", y= "number of students", title = ("Quiz 1 group score Distributions by representation during A17")) + facet_wrap(~ urm_id, nrow = 3 )
```

```
## Warning: Removed 28 rows containing non-finite values (stat_boxplot).
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-13-10.png)<!-- -->

```r
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_total)) + geom_histogram(color = "white") + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by version during A17")) + facet_wrap(~ ver, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

```r
#by eop
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_total)) + geom_histogram(color = "white") + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by eop during A17")) + facet_wrap(~ eop_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-2.png)<!-- -->

```r
#by sex
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_total)) + geom_histogram(color = "white") + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by sex during A17")) + facet_wrap(~ sex_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-3.png)<!-- -->

```r
#by fgn
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_total)) + geom_histogram(color = "white") + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by first generation status during A17")) + facet_wrap(~ fgn_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-4.png)<!-- -->

```r
#by urm
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(x= q1_total)) + geom_histogram(color = "white") + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by representation during A17")) + facet_wrap(~ urm_id, nrow = 3 )
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-5.png)<!-- -->

```r
##boxplots
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_total)) + geom_boxplot() + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by version during A17")) + facet_wrap(~ ver, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-6.png)<!-- -->

```r
#by eop
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_total)) + geom_boxplot() + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by eop during A17")) + facet_wrap(~ eop_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-7.png)<!-- -->

```r
#by sex
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_total)) + geom_boxplot() + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by sex during A17")) + facet_wrap(~ sex_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-8.png)<!-- -->

```r
#by fgn
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_total)) + geom_boxplot() + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by first generation status during A17")) + facet_wrap(~ fgn_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-9.png)<!-- -->

```r
#by urm
ggplot(data = data_2s_exam_score_true_A17, mapping = aes(y= q1_total)) + geom_boxplot() + labs(x = "Quiz 1 total score", y= "number of students", title = ("Quiz 1 total score Distributions by representation during A17")) + facet_wrap(~ urm_id, nrow = 3 )
```

![](graphs-for-exam-and-quiz-scores-gz_files/figure-html/unnamed-chunk-14-10.png)<!-- -->

# Now I am done with the graphs, I will come back and make the box plots look prettier


