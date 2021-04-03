# Metadata Guide

## master_2s_deidentified (DO NOT TOUCH).csv

1. **Bibliographic citation for the original data file.** _NA_ This file was created from data obtained under _IRB#????????_ from the UW registrar and the instructor for the courses analyzed herein, and then collated by student number which was subsequently redacted.
1. **Digital object identifier (DOI) for the data file.** _DOI TBD_
1. **The date on which the author first downloaded, or obtained in some other way, the original data files.** _NEED TO ADD THIS_
1. **A written explanation of how an interested reader can obtain a copy of the original data file.** _Archive location TBD_
1. **Whatever additional information an independent researcher would need to understand and use the data in the original data file.** The particular information required can vary a great deal depending on the nature of the original data file in question, and deciding what additional information to provide therefore requires thoughtful consideration and judgment.In many cases, the relevant information is similar to what is found in a codebook or users' guide for a dataset: variable names and definitions, coding schemes and units of measurement, and details of the sampling method and weight variables.In some cases, it is also necessary to include information about the file structure (e.g., the delimiters used to separate variables, or, in rectangular files without delimiters, the columns in which the variables are stored).Any other unique or idiosyncratic aspects of the data that an independent user of the data would need to understand should be explained as well.
    - `qtr` (character): Academic quarter. Values: 
        - "A17": autumn quarter 2017
        - "A16": autumn quarter 2016
        - "A18": autumn quarter 2018
        - "20194": autumn quarter 2019. UW Registrar's office refers to autumn quarter as "4", since it is the fourth quarter that occurs in a calendar year.
    - `course` (character): course number and course section
        - "142AB": CHEM 142, course sections A and B. Instructor taught both A and B sections.
        - "142D": CHEM 142, course section D
        - _NA_
    - `course.fullid` (character): Unique course identifier, of the structure "CHEM_142_\<COURSE SECT>_\<YEAR>_4"
        - "CHEM_142_A_2017_4" 
        - "CHEM_142_B_2017_4" 
        - "CHEM_142_A_2016_4" 
        - "CHEM_142_B_2016_4" 
        - "CHEM_142_D_2018_4"
    - `TAsect` (character): two-letter code for TA-lead subsection of a course, containing up to 24 students. 
        - _values in 142A_: "AK" "AH" "AU" "AC" "AB" "AL" "AI" "AA" "AJ" "AW" "AP" "AQ" "AS" "AN" "AV" "AD" "AO" "AF" "AE" "AG" "AT" "AR" "AY" "AZ" "AM" "AX" 
        - _values in 142B_: "BF" "BM" "BH" "BZ" "BA" "BX" "BW" "BV" "BU" "BI" "BB" "BQ" "BJ" "BR" "BE" "BL" "BD" "BP" "BC" "BY" "BN" "BT" "BG" "BS" "BO" "BK" 
        - _values in 142D_: "DW" "DX" "DC" "DL" "DJ" "DR" "DT" "DH" "DB" "DE" "DM" "DN" "DF" "DK" "DG" "DU" "DA" "DO" "DQ" "DS" "DD" "DP" "DI" "DV" 
        - _don't know the course these values refer to_: "T"  "CR" "D" "Q"  "U" NA   "H"  "Z"  "A"  "B"
    - `exam1predict`, `exam2predict`, `finalexampredict` (all numeric; _intended_ range 0 to 100): Score that students predicted they would earn on the exam. This was asked as the last question on the exam, and was worth three points.These points were added to the straight exam score, so a midterm for example was worth 103 points (100 for 20 multiple-choice questions worth 5 pts/ea. + this single free-response question worth 3 pts). The exact wording of the question changed over the years included in the study, but the intention was always that students would predict an integer score between 0% and 100%, even on the final exam, which was scored out of 150 points. (**2021-03-31: NEED TO CHECK HOW WE HANDLED NON-INTEGER DATA SUCH AS LETTERS, THE SPEED OF LIGHT, ETC. CHECK THE DUNNING-KRUGER WORK...**)
            - actual range `exam1predict`: 0 to 100
            - actual range `exam2predict`: 0 to 880
            - actual range `finalexampredict`: 0 to 300
    - `exam1`, `exam2`, `finalexam` (all numeric): Student scores on exams. 
            - `exam1` (20 5-pt MC + 1 3-pt FR; range 10 to 103; max possible 103): 142 Exam 1 coverage: quantum mechanics, Lewis structures, VSEPR. 
            - `exam2` (20 5-pt MC + 1 3-pt FR; range 0 to 103, max possible 103): 142 Exam 2 coverage: stoichiometry, aqueous-phase reaction classes, molarity, differential rate laws
            - `finalexam` (35 **?????** equally-weighted MC + 1 3-pt FR; range 0 to 154.29, max possible 155 (**CHECK THIS VALUE. DOES THIS DEPEND ON THE YEAR??**)): 142 Final Exam coverage: all prior material + gas laws, integrated rate laws, reaction mechanisms, collision theory
    - `instructorgrade` (numeric; range -0.2 to 4.0): Grade assigned to student. (**NOTE: The fact that this range starts at -0.2 makes me wonder what source Michael Mack used when he compiled these data. This looks like a case where the equation I used calculated a literal value of -0.2, but I would have converted this to a 0.0 before submitting to the registrar. I wonder what other minor deviations in the actual posted grades might exist in this dataset?**)
    - `exam` (character): Name of exam for the purposes of the Bloom's and complexity item analysis. Values are "Exam1", "Exam2", "FinalExam" in 2016, 2017, and 2018. Values are "exam1", "exam2", and "finalexam" in 2019.
    - `num_items` (numeric): Number of MC items on each exam. Exams 1 and 2 both had 20 MC items, and the Final Exam had 35 MC items. These numbers were the same in all years of the study.
    - `length_min` (numeric): Number of minutes allowed for the exam. Students were allowed 45 min for each of Exams 1 and 2, and 105 min for the Final exam. These values were the same in all years of the study.
    - `min_per_ques` (numeric): Ratio of `length_min` to `num_items`. Exams 1 and 2: 2.25 min/ques; Final exam: 3 min/ques. These values were the same in all years of the study.
    - `item_num` (numeric): Number of exam item for the purposes of the Bloom's and complexity item analysis.
        - Exams 1 and 2: 1-20
        - Final Exam: 1 -35
    - `stud_ans` (character): Answer that student chose for `item_num` on `exam`. Questions had up to five choices (only one correct answer). Values include: "A", "B", "C", "D", "E", and "Z". The "Z" responses were put in place during the creation of the item analysis data set in 2018. This was a catchall for any blanks or multimarks on the Scantron form. **CHECk THIS!!!**
    - `exam_key` (character): Correct answer to `item_num` on `exam`. Values include: "A", "B", "C", "D", and "E". 
    - `corr` (numeric): Indication of whether the student chose the correct answer to `item_num` on `exam`. Values: "0" (for incorrect), "1" (for correct)
    - `bloom_rating` (numeric): Level of Bloom's taxonomy applied to the question. Ratings were based on the original 1956 version of Bloom's. (For more information about the scheme applied to this question set, see the Chemistry in Bloom matrix in the `/supplements` folder.) Values in this dataset (the highest level of Bloom's, Evaluation, was not observed in this set of multiple-choice questions):
        - 1:   Knowledge
        - 2:   Comprehension
        - 3:   Application, Low-order cognitive skill
        - 3.5: Application, high-order cognitive skill
        - 4:   Analysis
        - 5:   Synthesis
    - `complexity_rating_mean` and `complexity_rating_median` (both numeric): Mean and median of the complexity rating assigned to a given question by three expert raters according to [Knaus, et al.](https://dx.doi.org/10.1021/ed900070y). The higher the rating, the more complex the problem was considered to be. 
            - Values for `complexity_rating_mean` range from 1 to 6, with some fractional values owing to division by three to obtain the mean (e.g., 2.67, 3.33). 
            - Values for `complexity_rating_median` range from 1 to 6, with no fractional values.
    - `item_code` (character): A unique identifier for `item_num` on `exam` in a master list of general chemistry exam questions. This code has the general form `ML-<\d\d\d>`, where "ML" stands for "master list", and the three digit code represents the location of the item in the master list. The questions in the master list were intentionally jumbled across instructors and years, so the `item_code`s in this set are not numerically inclusive.
    - `stem` (character): The test of the question stem from `item_num` on `exam`. (The text of the answer choices was not absorbed into the data set.)
    - `mastered_topics_initial_kc` (numeric; range 0 to 147): The number of topics a student indicated mastery of during the initial Knowledge Check in [ALEKS](https://www.aleks.com/). Students completed the initial KC during the first week of the quarter. The total number of topics included in the ALEKS course varied slightly by year: **LOOK THESE VALUES UP!!!**
    - `time_initial_kc` (character): Time the student took to complete their initial knowledge check in ALEKS. These data come in the unfortunate form "HhMMm", e.g. "1h43m".
    - `satmath` (numeric; range 360 to 800): Student's SAT Math score 
    - `satverbal` (numeric; range 370 to 800): Student's SAT Verbal score 
    - `high_sch_gpa` (numeric; range 2.46 to 4.00): Student's high school GPA, on a 4.0 scale (**IS THIS DATA SELF-REPORTED BY THE STUDENT? HOW DID WE OBTAIN THIS**)
    - `cci_total_1` and `cci_total_2` (both numeric; range for both 0 to 16 (**WHY IS 16 THE MAX? THE CCI HAS 22 QUESITONS**)): Student scores on the [Chemistry Concept Inventory](https://www.chemedx.org/JCEDLib/QBank/collection/CQandChP/CQs/ConceptsInventory/CCIIntro.html) trial 1 (at the beginning of the quarter) and trial 2 (at the end of the quarter).
    -  (numeric):
    - `project1` (character): Indicator of the project this student's data is used in. Values: 
            - "TESTING": refers to the two-stage assessment project, conducted in Autumn 2017. Students in the "TESTING" treatment were taught using a standard lecture approach in class and took standard individual midterm and final exams, but completed two-stage quizzes during their weekly quiz section. In the first stage, students had 20 min to complete a 10-question multiple choice quiz individually, and submitted their answers on a Scantron form. In the second stage, students assembled into assigned groups to complete the same quiz again in 15 min, and submitted their answers on an [IF-AT card](http://www.epsteineducation.com/home/).  
            - "ACTIVATE142": refers to the "activate 142" project in Autumn 2018, in which the course delivery was reformed to explicitly employ active learning techniques in class. Students in this cohort did not complete two-stage quizzes.
            - "ACTIVATE": refers to a second iteration of the "activate 142" project, in Autumn 2019. 
    - `experiment1` (character): Indicates whether this student's data is part of the "EXPERIMENTAL" group (in A17, A18, and A19) or the "CONTROL" group (in A16).
    - `eop.id` (character): Indicator of student's participation in the [Educational Opportunity Program (EOP)](https://depts.washington.edu/omadcs/eop/). This is used as an indicator of a student's socioeconomic status, since students from low-income backgrounds qualify for the program. These data were obtained as-is (**CHECK THIS**) from the UW Registrar's office. Values: "EOP", "non-EOP"
    - `sex.id` (character): Indicator of student's self-reported binary gender identity. These data were obtained as-is (**CHECK THIS**) from the UW Registrar's office. Values: "Female", "Male"
    - `fgn.id` (character): Indicator of student's first- or continuing-generation higher-education status. These data were inferred from information about parents' highest education level, which was obtained from the UW Registrar's office. (**CHECK ON EXACTLY HOW FGN STATUS WAS INFERRED FROM PARENTAL EDUCATION**) Values: "FGN", "non-FGN"
    - `urm.id` (character): Indicator of student's under-represented minority status. These data were inferred from ethnicity and nationality data obtained from the UW Registrar's office. Students from caucasian and asian ethnicities and international students were not classified as under-represented minorities. (**CHECK THIS...AND WHAT WAS THE LIST OF ETHNICITIES THAT WERE INCLUDED AS URM?**) Values: "URM", "non-URM"
    - `ethnicity` (character): Self-reported ethnicity data for student. These data were obtained from the UW Registrars office. Values: "HISPANIC", "ASIAN", "MULTI", "CAUCASN", "INTERNATIONAL", "AFRO-AM", "AMER-IND", "HAW/PAC", "NOT IND" (**HOW DID WE CLASSIFY "MULTI" AND "NOT IND" STUDENTS?**)
    - `group.id` (character): Group student was assigned to for working on the second stage of the two-stage quiz during discussion section. Groups contained up to four members, and were assigned by the instructor to explicitly minimize the number of out-numbered female and URM students whenever possible. We did not attempt to minimize thee number of out-numbered EOP or FGN students, as this would have introduced further complexity, and these characteristics are not necessarily visually apparent in the ways that gender and ethnicity can be. Format of `group.id`: "\<GROUP NUMBER>\<TA SECT>", e.g., "5AL", "2BQ". Each quiz section of 24 students had up to six quiz groups, and these groups persisted for the entire term. If a student was absent in a given week, or if a student dropped the class, the remaining group members were often left to work as a three- or two-some, rather than rearranging the group assignments.
    - `questionnum` (numeric): Number of a quiz question. Quizzes 2-8 had 10 multiple-choice questions. Values: [1-10]
    - `quiznum` (numeric): Number of the quiz delivered in a given week. Quiz 1 had only five questions, so it was not included in this analysis. Quizzes 2-8 all had 10 MC questions. Values: [2-8]
    - `all.response` (character): Record of the group's response(s) in the second stage, which was completed on an [IF-AT card](http://www.epsteineducation.com/home/). Since in most cases we could not tell the order in which the group picked their responses, the `all.response` values are presented in alphabetical order. For example, "CE", BCE", "ABCDE". However, a single response to a questions during the group stage was by far the most common observation. Values: strings of single, double, triple, quadruple, or quintuple letters.
    - `first.response` (character): In cases where at least one incorrect answer was chosen in addition to the correct answer, we attributed all the incorrect answers chosen as the "first" response of the group (assuming that once they fouND the correct answer, they would move on to the neXt question). So, the `first.response` variable is the string leftover once the correct answer is extracted. Values: strings of single, double, triple, quadruple, or quintuple letters. (**NOTE: THE QUINTUPLE LETTER STRINGS MUST BE AN ERROR, SINCE ONCE THE CORRECT ANSWER IS EXTRACTED THE MAXIMUM STRTING LENGTH FOR THIS VARIABLE SHOULD BE FOUR. WHAT OTHER ERRORS LURK??**)
    - `responsenum` (numeric; range 0 to 5): How many tries did it take the group to obtain the correct answer. If `all.response` is a single character, `responsenum = 1`. If `all.response` is three characters, `responsenum = 2`. A "0" value is observed if `all.response = NA`.
    - `ind.response` (character): Students response to `questionnum` during the the first stage of the quiz, individual where they complete it individually. Values: "A", "B", "C", "D", "E", or "0 (**WHY WOULD THERE BE A ZERO??**)
    - `quizversion` (character): Version of the quiz that the stduent took. There were four versions delierved each each. Values: "A", "B", "C", "D"
    - `quiz_key` (character): Answer to `questionnum` on `quiznum`, version `quizversion`. Values: "A", "B", "C", "D", "E"

1. **Supplementary documents with additional metadata.** 

* **ChemistryInBloom_19_07_01.docx, ChemistryInBloom_19_07_01.pdf**: Tool for applying Bloom's taxonomy to multiple-choice general chemistry questions. 
