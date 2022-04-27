---
date: "2022-04-26"
draft: false
featured: true
layout: single
excerpt: "A R package to create analysis-ready datasets from Medicare data"
title: medicareR
subtitle: An internal R package
links:
- icon: github
  icon_pack: fab
  name: code
  url:  https://umcstar.github.io/medicareR/
- icon: book
  icon_pack: fas
  name: website
  url:  https://github.com/UMCSTaR/medicareR
---

An open-sourced internal package that create analytic ready datasets from Medicare data. The code were designed specifically for the IHPI ,University of Michigan, Medicare data format. However, it can also be generalized to other medicare data format. This package is used at MEDLab at the department of surgery, University of Michigan.

---

## Intro to Medicare Data for Research

Medicare data is a claim database managed by the Centers for Medicare & Medicaid Services. It contained millions of claims per year. The data size is usually in a Terabyte unit. It's very challenging to work with medicare because it's not designed for research. Challenges included:

- changing variable names each year
- multiple components that contain very different information, for example PartA, B, C, D, etc.
- hundreds of variales with millions of observations
- variables need to be derived based on previous research findings
- very expensive to buy
- strict CMS rules of forbidding modern big data techniques (medicare will not let you have fun)
- and on and on.... 

If you are learning medicare data for research, I have benefited tremendously from the [ResDAC](https://resdac.org/) and their workshops. 


---

## How medicareR is helping our lab

### Medicare data education

I organized some frequently asked questions regarding variable definitions, data processing steps and other repeatedly happened processed in the package documentation. This makes onboarding new research fellows and analysts much easier. And of course, it helps my future self. :)


For example you can find frequently asked questions at [FAQ - Variable Definitions](https://umcstar.github.io/medicareR/articles/faq_var_def.html)



### Improve readability 

By using this package, I was able to reduce hundreds of lines of code to a pretty descriptive data processing stream line that research fellows and other analysts in our lab can (sort of) understand at the first glance.


For example, the code below were 
 1. Link all MedPAR, MBSF and Carrier files over years into one flat file 
 2. Add patient outcomes like readmission, complications into the analytic file

```
walk(
  years,
  ~ procedure_selection(year = .x) %>%
    # bene info
    bene_info(year = .x,
              filter_only_pt_65 = FALSE) %>%
    # elix
    fac_dx(year = .x) %>% # add dx code
    fac_dx_elix() %>%     # calculate Elixhauser flags based on dx code; remove dx code
    # ses info
    ses_info(ses = ses,
             ses_zscore = ses_zscore) %>%
    # outcomes
    # reop
    reoperation(
      max_year = max_year,
      reop_map = reop_define,
      fac_codes_folder = "fac_clm_hosp/fac_clm_code",
      year = .x,
      emergent_admission_medpar_claim_ids = emergent_claim_ids
    ) %>%
    # readmission
    readmission(
      emergent_admission_medpar_claim_ids = emergent_claim_ids,
      year = .x,
      fac_clm_folder = "fac_clm_hosp/fac_clm",
      max_year = max_year
    ) %>%
    # complication
    complication_flags(
      year = .x,
      cmp_pr = code_list$cmp_pr,
      cmp_dx = code_list$cmp_dx,
      all_n = code_list$all_n,
      max_year = max_year
    ) %>%
    # severe complication
    severe_cmp_flags(perc = 0.75) %>%
    # surgeon flags
    multi_surgeon_proc_assit_flags() %>%
    mutate_at(vars(c(
      dt_dod, val_mhhi, starts_with("e_ses_")
    )), ~ ifelse(is.na(.x), "", .x)) %>%
    save_file(year = .x, save_folder = "bariatric/analytic_phy")
) 

```

