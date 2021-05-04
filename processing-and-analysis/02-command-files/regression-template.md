---
# title: "Regression Template"
# author: "Colleen Craig"
# date: "5/3/2021"
output: 
  html_document:
    keep_md: yes
---

# Regression Template



## With the new data file, run the following regressions on the final exam scores:

### Multiple Regression with the "centered" data

1. `final_c ~ experiment1`
1. `final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c`
1. `final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + sex_id`
1. `final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + eop_id` 
1. `final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + fgn_id`
1. `final_c ~ experiment1 + satm_c + satv_c + aleksikc_c + hsgpa_c + urm_id`

### Multiple Regression with the "z-scored" (aka "scaled") data

1. `final_z ~ experiment1`
1. `final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z`
1. `final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + sex_id`
1. `final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + eop_id` 
1. `final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + fgn_id`
1. `final_z ~ experiment1 + satm_z + satv_z + aleksikc_z + hsgpa_z + urm_id`

### Multiple Regression + Interactions with the "centered" data

1. `final_c ~ experiment1*sex_id`
1. `final_c ~ experiment1*sex_id + satm_c + satv_c + aleksikc_c + hsgpa_c`
1. `final_c ~ experiment1*eop_id`
1. `final_c ~ experiment1*eop_id + satm_c + satv_c + aleksikc_c + hsgpa_c`
1. `final_c ~ experiment1*fgn_id`
1. `final_c ~ experiment1*fgn_id + satm_c + satv_c + aleksikc_c + hsgpa_c`
1. `final_c ~ experiment1*urm_id`
1. `final_c ~ experiment1*urm_id + satm_c + satv_c + aleksikc_c + hsgpa_c`

### Multiple Regression + Interactions with the "z-scored" (aka "scaled") data

1. `final_z ~ experiment1*sex_id`
1. `final_z ~ experiment1*sex_id + satm_z + satv_z + aleksikc_z + hsgpa_z`
1. `final_z ~ experiment1*eop_id`
1. `final_z ~ experiment1*eop_id + satm_z + satv_z + aleksikc_z + hsgpa_z`
1. `final_z ~ experiment1*fgn_id`
1. `final_z ~ experiment1*fgn_id + satm_z + satv_z + aleksikc_z + hsgpa_z`
1. `final_z ~ experiment1*urm_id`
1. `final_z ~ experiment1*urm_id + satm_z + satv_z + aleksikc_z + hsgpa_z`

