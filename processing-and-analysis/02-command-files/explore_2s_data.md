---
# title: "Untitled"
# author: "Colleen Craig"
date: "4/10/2021"
output: 
  html_document:
    keep_md: yes
---



## Exploring the `master_2s_small_deidentified.rds` data set

### _BEFORE MAKING ANY CHANGES TO THIS DOCUMENT..._

Create a new git branch in Rstudio. Call it something like "explore-2s-dataset".

### Import data 

1. Import `master_2s_small_deidentified (NO NOT TOUCH).rds` using `here()` and `read_rds()`
into a dataframe called `two_stage_df`.



### Relationships to Consider for the Plots and Summary Statstics

1. Scores for Exam 1, 2, and Final by...
    a. `course_fullid`
    a. `course_fullid` and `ver`
    a. `course_fullid` and demographic `eop_id`, `sex_id`, `fgn_id`, `urm_id` (all separate)
1. Scores for each of the A17 individual quiz scores (e.g. `q1_ind`) by...
    a. `ver`
    a. `eop_id`, `sex_id`, `fgn_id`, `urm_id` (all separate)
1. Scores for each of the A17 group quiz scores (e.g. `q1_grp`) by...
    a. `ver`
    a. `eop_id`, `sex_id`, `fgn_id`, `urm_id` (all separate)
1. Scores for each of the A17 total quiz scores (e.g. `q1_total`) by...
    a. `ver`
    a. `eop_id`, `sex_id`, `fgn_id`, `urm_id` (all separate)

**Note**: The Exam 1, 2, and Final data is formatted in a "long" fashion in 
`master_2s_small_deidentified (NO NOT TOUCH).rds`, whereas the A17 quiz data are 
formatted in a "wide" fashion. This means you will need to use `filter()` on the 
`exam` column to obtain rows relevant to each exam, and you will need to use `select()`
to obtain columns relevant to each quiz. (Here's some info on [long vs. wide data](http://jonathansoma.com/tutorials/d3/wide-vs-long-data/).)

### Create Faceted Histograms

### Create Faceted Boxplots

### Calculate Summary Statistics

