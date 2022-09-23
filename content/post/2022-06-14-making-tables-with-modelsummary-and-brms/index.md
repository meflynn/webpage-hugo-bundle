---
title: Making tables for multinomial models with {modelsummary} and {brms}
author: Michael E. Flynn
date: '2022-06-14'
slug: []
categories:
  - Academia
  - Statistics
  - Models
  - Publishing
  - Methods
  - Data  Science
  - brms
tags:
  - brms
  - stats
  - methods
  - data science
  - blogging
  - publishing
subtitle: ''
summary: ''
authors: []
lastmod: '2022-06-14T15:00:31-05:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />

**tl;dr: Learn how to make some cool and customizable tables for multinomial logit models using {brms} and {modelsummary}.**

**UPDATE: Vincent informed me that the most recent version of `{modelsummary}` relies entirely on the `{parameters}` package. Apparently the `{broom}` package will no longer be actively developing. Keep this in mind if you're trying this approach and get stuck.**

# Background

I'm coming off a couple of long projects where we were using a lot of multinomial logit models, and making publication quality tables was a major challenge. Actually, started using {brms} several years ago partly because I had data where we had 1) lots of individuals making choices, 2) those individuals were all grouped in some pretty clear ways, and 3) we were also interested in modeling group-level characteristics that might relate to individuals' choices. This more or less marked my full transition from Stata to R and from frequentist stats to Bayesian stats. 

Early on the biggest problem I ran into was finding a way to generate tables for multinomial models run using {brms}. Initially most packages didn't support {brms} and/or developing tables for multilevel/hierarchical models required a lot of extra legwork. Building tables to accommodate choice models was another issue. Taken together, these problems meant that I had to write a lot of extensive code by hand to generate clean Latex tables for the models I was using. The process has, thankfully, become much simpler over the last couple of years.

First, for those who aren't familiar {brms} is an amazing package created by [Paul B&uuml;rkner](https://paul-buerkner.github.io/brms/). It stands for Bayesian Regression Modeling using Stan, and, as the name suggests, provides users with a convenient front-end for building regression models using Stan as a back end. This is particularly useful for building multilevel models, but also lots of other stuff.

Second, {modelsummary} is another amazing and ever-expanding package created by [Vincent Arel-Bundock](https://vincentarelbundock.github.io/modelsummary/). This packages does a few different things, but most importantly and prominently it helps users to create excellent tables for summarizing data and building tables for regression output. The package is incredibly flexible, supports dozens of different model types, and Vincent is constantly adding new features. 

Both packages are fantastic. If you're interested in Bayesian modeling, or just looking for a new and easy way to build tables for whatever modeling package you already use, you should check out one or both of these packages.


# Multilevel Multinomial Logit Models

Lots of regression models are going to be fairly simple to present in a table format, and there are some fairly easy ways to go about generating those tables. Typically you'll have a single column per model, and each row of your table will be for a single variable. You might also have some summary statistics for the model at the bottom (think $ N $, $ R^2 $, etc.). 

Multinomial models get a little more complicated because you'll typically have multiple outcomes. Specifically, you'll have $k-1 $ columns to present in the table, where $k $ is the number of choices respondents have. In our case we had lots of models where survey respondents offered their assessments of various actors and we condensed those assessments down into four general categories: Positive, Neutral, Negative, and Don't Know. This means we ultimately ended up with three columns per model, with the "Neutral" response serving as the baseline category against which the others were compared.

Multilevel models complicate things slightly because you may also have summary statistics for the groups in your data in addition to the general summary statistics for the model. 

Last, Bayesian models and {brms} specifically provide users with a ton of additional information they might want to present beyond the traditional stuff you'd find in frequentist models. Some of this information can take a *long* time to compile. There's often going to be an efficiency and transparency tradeoff here, and so you may want to customize what you present in your table. Even if you end up presenting lots of information in the end, having the ability to control what's in your table at the outset can be really useful as you run the code to make sure the basic output looks right. 

Anyway, the goal here is rather niche, but it's to talk through the process of building nice and readable tables when we're using these models and have lots of information to present. {modelsummary} lets us do this, but also requires a bit of additional effort to fully customize our output.


# Getting Started

OK, first we're going to load our libraries. The relevance of some of these is immediately obvious given what I've already said, but I'll talk more about some of the additional packages we need below. 


```r
# Load libraries
library(tidyverse)
library(here)
library(modelsummary)
library(brms)
library(parameters)
library(kableExtra)
library(broom)
library(broom.mixed)
```

In addition to loading the libraries we want to load our model objects. In this case I have three models, each with three outcome categories. Each of these ends with a "p1" or a "t1" to denote the reference group for the model's outcome variable (e.g. if the outcome variable is asking about troops, people, or government). 

Then I'm creating a list object to store the three model objects. Note here that I'm leaving the label for the individual models blank by including the empty quotation marks in the list function. Depending on your situation you can go ahead and name these if you want. In my case it makes more sense to keep them empty because I plan to add a grouping header/title later in the final table and based on how {modelsummary} works including the titles here would create some redundancies in the final output and take up extra horizontal space. If you're working with HTML output and you have scrollable tables maybe this doesn't matter, but it can matter a lot for print versions.. 


```r
# Load brms model objects
m.c.t1 <- readRDS(here::here("static/files/data-files/m.c.t1.rds"))
m.c.p1 <- readRDS(here::here("static/files/data-files/m.c.p1.rds"))
m.c.g1 <- readRDS(here::here("static/files/data-files/m.c.g1.rds"))


# Create a list object to store the three separate model objects.
mod.list <- list(" " = m.c.t1,
                 " " = m.c.p1,
                 " " = m.c.g1)
```

Before we move on, let's take a quick look at the models and what they look like.


```r
summary(mod.list[[1]])
```

```{style="max-height: 100px;"}
##  Family: categorical 
##   Links: mudk = logit; muneg = logit; mupos = logit 
## Formula: troops_1_cat ~ contact_pers + contact_nonpers + benefit_pers + benefit_nonpers + age + ed_z + ideology_z + income.5.cat + gender + minority + relig + troops_crime_pers + american_inf_1 + american_inf_2 + basecount_z + gdp_z + pop_z + troops_z + (1 | r | country) 
##    Data: o.data (Number of observations: 38626) 
##   Draws: 4 chains, each with iter = 8000; warmup = 4000; thin = 1;
##          total post-warmup draws = 16000
## 
## Group-Level Effects: 
## ~country (Number of levels: 14) 
##                                      Estimate Est.Error l-95% CI u-95% CI Rhat
## sd(mudk_Intercept)                       0.51      0.14     0.30     0.83 1.00
## sd(muneg_Intercept)                      0.83      0.20     0.53     1.31 1.00
## sd(mupos_Intercept)                      0.57      0.15     0.35     0.94 1.00
## cor(mudk_Intercept,muneg_Intercept)      0.57      0.22     0.05     0.88 1.00
## cor(mudk_Intercept,mupos_Intercept)     -0.48      0.24    -0.84     0.08 1.00
## cor(muneg_Intercept,mupos_Intercept)    -0.15      0.27    -0.63     0.40 1.00
##                                      Bulk_ESS Tail_ESS
## sd(mudk_Intercept)                       8625    10276
## sd(muneg_Intercept)                      8585    10729
## sd(mupos_Intercept)                      7977     9513
## cor(mudk_Intercept,muneg_Intercept)      8920    10405
## cor(mudk_Intercept,mupos_Intercept)      8157    11321
## cor(muneg_Intercept,mupos_Intercept)    10970    11879
## 
## Population-Level Effects: 
##                                                 Estimate Est.Error l-95% CI
## mudk_Intercept                                     -2.21      0.20    -2.62
## muneg_Intercept                                    -0.46      0.24    -0.94
## mupos_Intercept                                    -0.24      0.18    -0.58
## mudk_contact_persDontknowDdeclinetoanswer           0.50      0.13     0.23
## mudk_contact_persYes                               -0.61      0.17    -0.94
## mudk_contact_nonpersDontknowDdeclinetoanswer       -0.39      0.12    -0.64
## mudk_contact_nonpersYes                            -0.72      0.15    -1.02
## mudk_benefit_persDontknowDdeclinetoanswer           0.48      0.11     0.26
## mudk_benefit_persYes                               -0.59      0.22    -1.05
## mudk_benefit_nonpersDontknowDdeclinetoanswer        0.25      0.11     0.03
## mudk_benefit_nonpersYes                            -0.55      0.22    -1.00
## mudk_age25to34years                                -0.12      0.07    -0.26
## mudk_age35to44years                                -0.26      0.08    -0.41
## mudk_age45to54years                                -0.31      0.08    -0.46
## mudk_age55to64years                                -0.59      0.09    -0.76
## mudk_ageAge65orolder                               -0.78      0.10    -0.98
## mudk_ed_z                                          -0.04      0.06    -0.16
## mudk_ideology_z                                    -0.21      0.07    -0.34
## mudk_income.5.cat21M40%                            -0.18      0.08    -0.34
## mudk_income.5.cat41M60%                            -0.07      0.08    -0.23
## mudk_income.5.cat61M80%                            -0.33      0.10    -0.52
## mudk_income.5.cat81M100%                           -0.42      0.11    -0.63
## mudk_genderFemale                                   0.38      0.05     0.28
## mudk_genderNonMbinary                              -1.52      1.36    -4.92
## mudk_genderNoneoftheabove                           0.19      0.56    -0.95
## mudk_minorityYes                                   -0.12      0.09    -0.30
## mudk_minorityDeclinetoanswer                        0.53      0.11     0.31
## mudk_religBuddhism                                  0.09      0.18    -0.27
## mudk_religCatholicism                               0.06      0.08    -0.10
## mudk_religChristianityprotestant                    0.09      0.11    -0.14
## mudk_religDeclinetoanswer                           0.24      0.09     0.06
## mudk_religHinduism                                  0.01      0.35    -0.72
## mudk_religIslam                                     0.04      0.14    -0.25
## mudk_religJudaism                                   0.75      0.37     0.01
## mudk_religLocal                                    -0.68      0.46    -1.63
## mudk_religLocalreligion                            -0.17      0.35    -0.88
## mudk_religMormonism                               -91.22     51.29  -202.31
## mudk_religOther                                     0.17      0.09    -0.01
## mudk_religProtestant                                0.04      0.12    -0.20
## mudk_religShinto                                  -88.49     50.35  -196.62
## mudk_troops_crime_persYes                          -0.22      0.32    -0.89
## mudk_troops_crime_persDontknowDdeclinetoanswer      0.24      0.14    -0.03
## mudk_american_inf_1DontknowDdeclinetoanswer         0.44      0.13     0.20
## mudk_american_inf_1Alittle                         -0.33      0.12    -0.56
## mudk_american_inf_1Some                            -0.21      0.11    -0.42
## mudk_american_inf_1Alot                            -0.29      0.12    -0.52
## mudk_american_inf_2DontknowDdeclinetoanswer         1.90      0.08     1.74
## mudk_american_inf_2Verynegative                     0.62      0.14     0.35
## mudk_american_inf_2Negative                         0.28      0.07     0.14
## mudk_american_inf_2Positive                         0.00      0.09    -0.16
## mudk_american_inf_2Verypositive                     0.62      0.20     0.22
## mudk_basecount_z                                    0.09      0.07    -0.05
## mudk_gdp_z                                          0.82      0.49    -0.18
## mudk_pop_z                                         -0.20      0.37    -0.91
## mudk_troops_z                                      -0.66      0.50    -1.60
## muneg_contact_persDontknowDdeclinetoanswer          0.04      0.11    -0.17
## muneg_contact_persYes                               0.16      0.06     0.05
## muneg_contact_nonpersDontknowDdeclinetoanswer       0.01      0.08    -0.13
## muneg_contact_nonpersYes                            0.09      0.05    -0.01
## muneg_benefit_persDontknowDdeclinetoanswer         -0.44      0.10    -0.62
## muneg_benefit_persYes                              -0.50      0.09    -0.68
## muneg_benefit_nonpersDontknowDdeclinetoanswer      -0.38      0.08    -0.54
## muneg_benefit_nonpersYes                           -0.26      0.08    -0.41
## muneg_age25to34years                                0.09      0.04     0.01
## muneg_age35to44years                               -0.01      0.04    -0.09
## muneg_age45to54years                               -0.11      0.04    -0.20
## muneg_age55to64years                               -0.19      0.05    -0.28
## muneg_ageAge65orolder                              -0.07      0.05    -0.17
## muneg_ed_z                                          0.16      0.03     0.09
## muneg_ideology_z                                   -0.24      0.03    -0.31
## muneg_income.5.cat21M40%                           -0.06      0.05    -0.15
## muneg_income.5.cat41M60%                            0.00      0.05    -0.09
## muneg_income.5.cat61M80%                           -0.03      0.05    -0.13
## muneg_income.5.cat81M100%                           0.04      0.05    -0.07
## muneg_genderFemale                                 -0.03      0.03    -0.08
## muneg_genderNonMbinary                              0.41      0.30    -0.18
## muneg_genderNoneoftheabove                         -0.48      0.47    -1.42
## muneg_minorityYes                                   0.04      0.05    -0.04
## muneg_minorityDeclinetoanswer                      -0.10      0.09    -0.28
## muneg_religBuddhism                                -0.22      0.08    -0.37
## muneg_religCatholicism                             -0.42      0.04    -0.51
## muneg_religChristianityprotestant                  -0.46      0.06    -0.58
## muneg_religDeclinetoanswer                         -0.25      0.06    -0.36
## muneg_religHinduism                                -0.21      0.22    -0.65
## muneg_religIslam                                    0.29      0.08     0.13
## muneg_religJudaism                                 -0.05      0.28    -0.62
## muneg_religLocal                                   -0.37      0.20    -0.76
## muneg_religLocalreligion                           -0.34      0.20    -0.74
## muneg_religMormonism                               -0.21      0.41    -1.03
## muneg_religOther                                   -0.16      0.05    -0.25
## muneg_religProtestant                              -0.39      0.06    -0.52
## muneg_religShinto                                  -0.28      0.23    -0.74
## muneg_troops_crime_persYes                         -0.13      0.13    -0.40
## muneg_troops_crime_persDontknowDdeclinetoanswer    -0.34      0.13    -0.60
## muneg_american_inf_1DontknowDdeclinetoanswer       -0.82      0.10    -1.02
## muneg_american_inf_1Alittle                        -0.30      0.07    -0.44
## muneg_american_inf_1Some                           -0.17      0.07    -0.30
## muneg_american_inf_1Alot                            0.13      0.07    -0.01
## muneg_american_inf_2DontknowDdeclinetoanswer        0.38      0.08     0.22
## muneg_american_inf_2Verynegative                    1.97      0.06     1.85
## muneg_american_inf_2Negative                        1.14      0.03     1.07
## muneg_american_inf_2Positive                       -0.28      0.05    -0.37
## muneg_american_inf_2Verypositive                   -0.21      0.12    -0.44
## muneg_basecount_z                                   0.07      0.04     0.00
## muneg_gdp_z                                        -0.80      0.55    -1.90
## muneg_pop_z                                         0.63      0.53    -0.47
## muneg_troops_z                                      0.60      0.67    -0.66
## mupos_contact_persDontknowDdeclinetoanswer         -0.31      0.09    -0.50
## mupos_contact_persYes                               0.48      0.05     0.39
## mupos_contact_nonpersDontknowDdeclinetoanswer       0.08      0.06    -0.04
## mupos_contact_nonpersYes                            0.27      0.04     0.18
## mupos_benefit_persDontknowDdeclinetoanswer         -0.17      0.08    -0.32
## mupos_benefit_persYes                               0.05      0.06    -0.07
## mupos_benefit_nonpersDontknowDdeclinetoanswer      -0.09      0.07    -0.22
## mupos_benefit_nonpersYes                            0.38      0.06     0.27
## mupos_age25to34years                               -0.11      0.04    -0.19
## mupos_age35to44years                               -0.08      0.04    -0.16
## mupos_age45to54years                               -0.05      0.04    -0.13
## mupos_age55to64years                                0.11      0.04     0.03
## mupos_ageAge65orolder                               0.19      0.04     0.11
## mupos_ed_z                                          0.03      0.03    -0.02
## mupos_ideology_z                                    0.38      0.03     0.32
## mupos_income.5.cat21M40%                           -0.03      0.04    -0.12
## mupos_income.5.cat41M60%                            0.01      0.04    -0.07
## mupos_income.5.cat61M80%                           -0.01      0.04    -0.10
## mupos_income.5.cat81M100%                           0.06      0.05    -0.03
## mupos_genderFemale                                 -0.22      0.02    -0.27
## mupos_genderNonMbinary                             -0.18      0.23    -0.63
## mupos_genderNoneoftheabove                         -0.68      0.39    -1.46
## mupos_minorityYes                                  -0.05      0.04    -0.13
## mupos_minorityDeclinetoanswer                      -0.06      0.08    -0.22
## mupos_religBuddhism                                 0.06      0.07    -0.07
## mupos_religCatholicism                              0.15      0.04     0.07
## mupos_religChristianityprotestant                   0.13      0.05     0.03
## mupos_religDeclinetoanswer                         -0.15      0.05    -0.26
## mupos_religHinduism                                -0.23      0.14    -0.51
## mupos_religIslam                                   -0.17      0.07    -0.31
## mupos_religJudaism                                  0.18      0.15    -0.11
## mupos_religLocal                                   -0.18      0.17    -0.52
## mupos_religLocalreligion                           -0.21      0.16    -0.51
## mupos_religMormonism                                0.47      0.33    -0.17
## mupos_religOther                                   -0.12      0.05    -0.21
## mupos_religProtestant                               0.25      0.05     0.15
## mupos_religShinto                                  -0.03      0.20    -0.41
## mupos_troops_crime_persYes                         -0.11      0.09    -0.28
## mupos_troops_crime_persDontknowDdeclinetoanswer    -0.55      0.11    -0.77
## mupos_american_inf_1DontknowDdeclinetoanswer       -0.35      0.10    -0.54
## mupos_american_inf_1Alittle                         0.04      0.07    -0.09
## mupos_american_inf_1Some                            0.19      0.06     0.06
## mupos_american_inf_1Alot                            0.53      0.07     0.40
## mupos_american_inf_2DontknowDdeclinetoanswer       -0.27      0.08    -0.42
## mupos_american_inf_2Verynegative                   -0.57      0.08    -0.73
## mupos_american_inf_2Negative                       -0.33      0.03    -0.40
## mupos_american_inf_2Positive                        1.24      0.03     1.18
## mupos_american_inf_2Verypositive                    1.94      0.07     1.81
## mupos_basecount_z                                   0.02      0.03    -0.04
## mupos_gdp_z                                         0.41      0.47    -0.46
## mupos_pop_z                                        -0.20      0.39    -1.04
## mupos_troops_z                                     -0.76      0.51    -1.80
##                                                 u-95% CI Rhat Bulk_ESS Tail_ESS
## mudk_Intercept                                     -1.82 1.00     9965    11260
## muneg_Intercept                                     0.02 1.00     7614     9735
## mupos_Intercept                                     0.12 1.00     9111    10223
## mudk_contact_persDontknowDdeclinetoanswer           0.76 1.00    28898    12459
## mudk_contact_persYes                               -0.28 1.00    35071    12285
## mudk_contact_nonpersDontknowDdeclinetoanswer       -0.15 1.00    31078    12476
## mudk_contact_nonpersYes                            -0.43 1.00    36906    12402
## mudk_benefit_persDontknowDdeclinetoanswer           0.69 1.00    32950    12049
## mudk_benefit_persYes                               -0.17 1.00    35339    12214
## mudk_benefit_nonpersDontknowDdeclinetoanswer        0.47 1.00    32827    12413
## mudk_benefit_nonpersYes                            -0.13 1.00    31219    12371
## mudk_age25to34years                                 0.03 1.00    24495    13948
## mudk_age35to44years                                -0.11 1.00    25225    13106
## mudk_age45to54years                                -0.15 1.00    24303    13303
## mudk_age55to64years                                -0.42 1.00    24857    12792
## mudk_ageAge65orolder                               -0.58 1.00    28255    13715
## mudk_ed_z                                           0.08 1.00    33506    12754
## mudk_ideology_z                                    -0.08 1.00    33194    12144
## mudk_income.5.cat21M40%                            -0.01 1.00    22824    13682
## mudk_income.5.cat41M60%                             0.09 1.00    19412    14131
## mudk_income.5.cat61M80%                            -0.14 1.00    20502    12789
## mudk_income.5.cat81M100%                           -0.21 1.00    22830    13131
## mudk_genderFemale                                   0.49 1.00    33841    10953
## mudk_genderNonMbinary                               0.49 1.00    20464     8210
## mudk_genderNoneoftheabove                           1.24 1.00    33952    11270
## mudk_minorityYes                                    0.05 1.00    32060    12384
## mudk_minorityDeclinetoanswer                        0.74 1.00    28095    13069
## mudk_religBuddhism                                  0.43 1.00    26888    11973
## mudk_religCatholicism                               0.23 1.00    18539    13734
## mudk_religChristianityprotestant                    0.31 1.00    18774    13720
## mudk_religDeclinetoanswer                           0.42 1.00    18677    13477
## mudk_religHinduism                                  0.67 1.00    34397    11772
## mudk_religIslam                                     0.32 1.00    18499    12850
## mudk_religJudaism                                   1.45 1.00    32645    11069
## mudk_religLocal                                     0.16 1.00    32044    11584
## mudk_religLocalreligion                             0.47 1.00    34222    12600
## mudk_religMormonism                                -9.58 1.00    24158    13289
## mudk_religOther                                     0.36 1.00    19821    12970
## mudk_religProtestant                                0.28 1.00    27632    11675
## mudk_religShinto                                   -7.75 1.00    19564    11761
## mudk_troops_crime_persYes                           0.38 1.00    32686    12782
## mudk_troops_crime_persDontknowDdeclinetoanswer      0.51 1.00    31428    12367
## mudk_american_inf_1DontknowDdeclinetoanswer         0.69 1.00    15559    13814
## mudk_american_inf_1Alittle                         -0.09 1.00    13818    13275
## mudk_american_inf_1Some                             0.01 1.00    12912    12654
## mudk_american_inf_1Alot                            -0.05 1.00    13781    12670
## mudk_american_inf_2DontknowDdeclinetoanswer         2.06 1.00    28532    13094
## mudk_american_inf_2Verynegative                     0.89 1.00    31676    12350
## mudk_american_inf_2Negative                         0.41 1.00    30795    12359
## mudk_american_inf_2Positive                         0.17 1.00    32015    11768
## mudk_american_inf_2Verypositive                     1.00 1.00    34404    12569
## mudk_basecount_z                                    0.23 1.00    31930    11766
## mudk_gdp_z                                          1.75 1.00     7257    10182
## mudk_pop_z                                          0.55 1.00     8368     9894
## mudk_troops_z                                       0.38 1.00     7180     9511
## muneg_contact_persDontknowDdeclinetoanswer          0.24 1.00    31677    13165
## muneg_contact_persYes                               0.27 1.00    29413    13303
## muneg_contact_nonpersDontknowDdeclinetoanswer       0.16 1.00    30181    12663
## muneg_contact_nonpersYes                            0.20 1.00    29722    12561
## muneg_benefit_persDontknowDdeclinetoanswer         -0.25 1.00    31796    12788
## muneg_benefit_persYes                              -0.31 1.00    29760    12683
## muneg_benefit_nonpersDontknowDdeclinetoanswer      -0.22 1.00    28191    12601
## muneg_benefit_nonpersYes                           -0.10 1.00    30794    13592
## muneg_age25to34years                                0.18 1.00    19021    14170
## muneg_age35to44years                                0.08 1.00    17918    13604
## muneg_age45to54years                               -0.02 1.00    17885    13006
## muneg_age55to64years                               -0.10 1.00    19718    13630
## muneg_ageAge65orolder                               0.03 1.00    19181    13486
## muneg_ed_z                                          0.22 1.00    29969    12788
## muneg_ideology_z                                   -0.18 1.00    29967    13882
## muneg_income.5.cat21M40%                            0.04 1.00    16734    12707
## muneg_income.5.cat41M60%                            0.10 1.00    16085    13400
## muneg_income.5.cat61M80%                            0.07 1.00    15790    13382
## muneg_income.5.cat81M100%                           0.14 1.00    15876    13469
## muneg_genderFemale                                  0.03 1.00    33812    12500
## muneg_genderNonMbinary                              0.99 1.00    32807    12179
## muneg_genderNoneoftheabove                          0.42 1.00    35351    11331
## muneg_minorityYes                                   0.13 1.00    28631    13853
## muneg_minorityDeclinetoanswer                       0.08 1.00    28560    13121
## muneg_religBuddhism                                -0.08 1.00    28084    12375
## muneg_religCatholicism                             -0.34 1.00    21767    13047
## muneg_religChristianityprotestant                  -0.33 1.00    22392    14003
## muneg_religDeclinetoanswer                         -0.14 1.00    24872    13345
## muneg_religHinduism                                 0.22 1.00    33576    12434
## muneg_religIslam                                    0.45 1.00    29433    13237
## muneg_religJudaism                                  0.49 1.00    34033    11571
## muneg_religLocal                                    0.02 1.00    36353    12593
## muneg_religLocalreligion                            0.05 1.00    29412    11751
## muneg_religMormonism                                0.58 1.00    31354    12582
## muneg_religOther                                   -0.07 1.00    21870    13789
## muneg_religProtestant                              -0.27 1.00    27692    12782
## muneg_religShinto                                   0.17 1.00    29869    12719
## muneg_troops_crime_persYes                          0.13 1.00    26836    12721
## muneg_troops_crime_persDontknowDdeclinetoanswer    -0.08 1.00    31219    13322
## muneg_american_inf_1DontknowDdeclinetoanswer       -0.62 1.00    17595    13033
## muneg_american_inf_1Alittle                        -0.17 1.00    12365    12392
## muneg_american_inf_1Some                           -0.05 1.00    12309    11739
## muneg_american_inf_1Alot                            0.26 1.00    13038    12974
## muneg_american_inf_2DontknowDdeclinetoanswer        0.53 1.00    27150    12704
## muneg_american_inf_2Verynegative                    2.09 1.00    26109    12100
## muneg_american_inf_2Negative                        1.20 1.00    28932    12972
## muneg_american_inf_2Positive                       -0.20 1.00    29590    12833
## muneg_american_inf_2Verypositive                    0.01 1.00    29668    11649
## muneg_basecount_z                                   0.14 1.00    28661    12810
## muneg_gdp_z                                         0.25 1.00    13085    11450
## muneg_pop_z                                         1.64 1.00    10633    10608
## muneg_troops_z                                      2.00 1.00    10630    11094
## mupos_contact_persDontknowDdeclinetoanswer         -0.13 1.00    29408    12810
## mupos_contact_persYes                               0.57 1.00    28351    11980
## mupos_contact_nonpersDontknowDdeclinetoanswer       0.20 1.00    28651    12566
## mupos_contact_nonpersYes                            0.35 1.00    29172    13128
## mupos_benefit_persDontknowDdeclinetoanswer         -0.02 1.00    31679    12489
## mupos_benefit_persYes                               0.17 1.00    28144    11998
## mupos_benefit_nonpersDontknowDdeclinetoanswer       0.04 1.00    28630    13105
## mupos_benefit_nonpersYes                            0.48 1.00    27498    12797
## mupos_age25to34years                               -0.04 1.00    20577    14146
## mupos_age35to44years                               -0.01 1.00    19447    13819
## mupos_age45to54years                                0.03 1.00    18261    13498
## mupos_age55to64years                                0.18 1.00    19104    14277
## mupos_ageAge65orolder                               0.27 1.00    20552    13091
## mupos_ed_z                                          0.09 1.00    29661    13353
## mupos_ideology_z                                    0.44 1.00    29576    13265
## mupos_income.5.cat21M40%                            0.05 1.00    17020    13662
## mupos_income.5.cat41M60%                            0.09 1.00    16045    13836
## mupos_income.5.cat61M80%                            0.07 1.00    16733    12897
## mupos_income.5.cat81M100%                           0.15 1.00    16900    13783
## mupos_genderFemale                                 -0.17 1.00    32595    12972
## mupos_genderNonMbinary                              0.27 1.00    36395    12502
## mupos_genderNoneoftheabove                          0.08 1.00    35631    12341
## mupos_minorityYes                                   0.02 1.00    29799    12901
## mupos_minorityDeclinetoanswer                       0.09 1.00    27930    13383
## mupos_religBuddhism                                 0.19 1.00    26507    12840
## mupos_religCatholicism                              0.22 1.00    19101    14110
## mupos_religChristianityprotestant                   0.23 1.00    19766    13453
## mupos_religDeclinetoanswer                         -0.05 1.00    22491    12723
## mupos_religHinduism                                 0.04 1.00    31120    11441
## mupos_religIslam                                   -0.02 1.00    25817    14288
## mupos_religJudaism                                  0.48 1.00    31904    12952
## mupos_religLocal                                    0.15 1.00    38026    11372
## mupos_religLocalreligion                            0.10 1.00    29335    12494
## mupos_religMormonism                                1.12 1.00    30673    12942
## mupos_religOther                                   -0.03 1.00    20788    13895
## mupos_religProtestant                               0.35 1.00    25563    12706
## mupos_religShinto                                   0.36 1.00    31985    12756
## mupos_troops_crime_persYes                          0.07 1.00    28799    13442
## mupos_troops_crime_persDontknowDdeclinetoanswer    -0.33 1.00    30133    13226
## mupos_american_inf_1DontknowDdeclinetoanswer       -0.16 1.00    17919    14365
## mupos_american_inf_1Alittle                         0.17 1.00    13119    12543
## mupos_american_inf_1Some                            0.32 1.00    12624    12508
## mupos_american_inf_1Alot                            0.66 1.00    13174    12358
## mupos_american_inf_2DontknowDdeclinetoanswer       -0.12 1.00    29210    13052
## mupos_american_inf_2Verynegative                   -0.40 1.00    31274    13396
## mupos_american_inf_2Negative                       -0.26 1.00    32656    13653
## mupos_american_inf_2Positive                        1.29 1.00    29010    13253
## mupos_american_inf_2Verypositive                    2.08 1.00    29050    12641
## mupos_basecount_z                                   0.09 1.00    31330    12676
## mupos_gdp_z                                         1.37 1.00    10986    11660
## mupos_pop_z                                         0.53 1.00    10387     9799
## mupos_troops_z                                      0.21 1.00    10625    10867
## 
## Draws were sampled using sample(hmc). For each parameter, Bulk_ESS
## and Tail_ESS are effective sample size measures, and Rhat is the potential
## scale reduction factor on split chains (at convergence, Rhat = 1).
```

There's a lot going on here, but you can see from the summary output that organizing this could be a bit of a bear. We have three outcome categories, lots of categorical variables (some conceptually related), model summary statistics, etc.

# Customization

The first two chunks are pretty boiler plate, and for lots of types of models you can probably just go on to use {modelsummary} directly and get some nice tables. But this section is going to dive into some of the extra steps required to make the tables look nice and clean, but also to save us a ton of time. 

The big issue that we have to address is that {modelsummary} is going to automatically try to generate lots of different types of goodness-of-fit or model summary statistics for the models you're including. If this were OLS it would be super quick. But since we're using {brms} models the defaults can run for a very long time and, depending on your workflow needs, cause some major slowdowns. 

{modelsummary} is using the packages like {tidy}, {parameters}, and {broom} to extract information from the models in your list and to generate a basic data frame with that output that serves as the basis for the final table. Things like WAIC and LOOIC can take a *long* time to calculate, and depending on your needs you might not want them right way.

Beyond that, you might just want to customize how the footer of your table looks, what information is included in which places, etc. This is a good way to do that, but it takes a little extra work.

First, we need to assign a "custom" class to each of the three model objects stored in the `mod.list` object. Assigning the custom class is going allow us to write some custom {tidy} and {glance} functions that will let us select the specific summary stats and other model info that we want to include.


```r
for (i in seq_along(mod.list)){
  class(mod.list[[i]]) <- c("custom", class(mod.list[[i]]))
}
```


Next we can write out custom tidy function, appropriately labeled here `tidy.custom`. Actually, I think you *have* to name it this so {modelsummary} recognizes that you want to use a custom function.

There's a lot going on here, and some of it is going to be specific to the models I'm using as an example.

First, you create the function, where `x` is the object placeholder, and `conf.level=.95` is specifying the default confidence/credible interval threshold. The `...` just leaves it open for other arguments.

Next we create an object named `out` using the {parameters} package. First we start off creating a data frame using this function, then we standardize all of the variable/column names (i.e. make them lower case). So far pretty standard.

The `mutate` chunk is where things get more complicated, and where you'll need to substitute your own model-specific factors. {modelsummary} also has a formula-based argument for helping you to arrange your models in the resulting table, but we need to do a little extra work here to properly identify the outcome variable levels. This is partly a function of the fact that {brms} does some weird things like attaching outcome choices (i.e. the binary outcome variable for the individual equations) as prefixes to each variable name, meaning there are no outcome levels for {parameters} to automatically detect. So we need to create them.

All I'm going here is using `case_when(...)` to identify the outcome variable levels/choices and creating a categorical `y.level` variable containing the full name for each of the choices for the outcome variable. As I mentioned above, those are "Don't know", "Negative", and "Positive" (with "Neutral" as the omitted reference category).

Next, I take the `term` column that {parameters} generates, which contains all of the predictor variables, and I remove the outcome level prefixes that {brms} attaches. This way I have two columns with variable names (`term`) and the outcome variable choice/model equation (i.e. `y.level`).


```r
# tidy method extracts level names into new column
tidy.custom <- function(x, conf.level=.95, ...) {                                   # Create function
  out = parameters::parameters(x, ci=conf.level, verbose=FALSE) |>                 # Call {parameters} to pull model parameter info with specified credible interval
        parameters::standardize_names(style="broom") |>                            # make names lower case
        mutate(y.level = case_when(grepl("mudk", term) ~ "Don't Know",              # Change outcome level values to plain meaning for output table
                                 grepl("muneg", term) ~ "Negative",
                                 grepl("mupos", term) ~ "Positive"),
               term = gsub("dk|neg|pos", "", term))                                 # remove outcome prefix {brms} attaches to variable names
  return(out)
}
```


Next we can move on to writing a custom function to pull the relevant model and summary statistic information. First, we need to tell `glance()` to quiet down since it's going to do lots of stuff we don't necessarily want it to do right now.  I can't remember if the `gof.check` lines are necessary at this point (I seem to recall it had no effect one way or the other when I was initially working on this), but I'll turn those off just to be safe, too.


```r
# Write custom glance function to extract summary information.
glance.custom <- function(x, ...) {
  ret <- tibble::tibble(N = summary(x)$nobs)
  ret
}

# Turn off GOF stuff
gof.check <- modelsummary::gof_map
gof.check$omit <- TRUE
```

Next we can move on to using the {parameters} package to full information on the "random" part of our varying intercepts model.  We'll also use this step to include some basic summary information on the models as mentioned above.

Ultimately the goal here is to generate a clean data frame containing summary information that we want. I've included more specific comments in the code chunk below for readers who want to scrutinize each step, but much of this mirrors what we just did with the `tidy.custom()` function above, it's just doing it to the "random" or "varying" part of the model rather than the population-level coefficients.


```r
# Write function to loop over list of models
rows <- lapply(mod.list, function(x){

  temp.sd <- parameters::parameters(x, effect = "random")  |>                   # start with parameters and pull "random" component of model
    filter(grepl(".*sd.*", Parameter)) |>                                       # filter out parameters containing standard deviation info
    filter(grepl(".*persYes.*|.*nonpersYes.*|.*Intercept.*", Parameter)) |>     # further filter parameters containing SD info for relevant variables
    dplyr::select(Parameter, Median) |>                                         # Keep only the Parameter (name) column and the Median column
    dplyr::mutate(Median = as.character(round(Median, 2)),
      y.level = case_when(                                                       # Like before, create a y.level column for outcome variable level/equation
      grepl(".*mudk.*", Parameter) ~ "dk",
      grepl(".*mupos.*", Parameter) ~ "pos",
      grepl(".*muneg.*", Parameter) ~ "neg",
    ),
    Parameter = gsub("_mudk_|_mupos_|_muneg_", "_", Parameter)) |>              # Remove the {brms} prefix from the Parameter column
    pivot_wider(id_cols = Parameter,                                             # Rearrange  columns from long to wide
                values_from = Median,
                names_from = y.level) |> 
    mutate(Parameter = case_when(                                                # Rename relevant parameters to appear how you want in text
      grepl(".*Intercept.*", Parameter) ~ "sd(Intercept)"
    ))
  
  temp.obs <- tibble::tribble(~Parameter, ~dk, ~neg, ~pos,                       # Create another data frame containing observation count and grouping info
                              "N", as.character(nobs(x)), "", "",
                              "Group", "Country", "", "",
                              "\\# Groups", as.character(length(unique(x$data$country))), "", "")
  

  temp.com <- bind_rows(temp.obs, temp.sd)                                       # Bind two data frames together for consolidated footer data frame

  return(temp.com)
  
  }
)

# Group everything from the three models together
# Also select relevant columns containing information
rows.com <- bind_cols(rows[[1]], rows[[2]], rows[[3]]) |> 
  dplyr::select(1, 2, 3, 4, 6, 7, 8, 10, 11, 12) 

# Rename those columns so they'll match eventual output data frame names
names(rows.com) <- c("term", "col1", "col2", "col3", "col4", "col5", "col6", "col7", "col8", "col9")
```

Great! Next we can start cleaning up the variable names for presenting them in the table, and we can even go a step further to add grouping labels to help readers move between broad categories of predictor variables (e.g. income, age, etc.).

Here's we're going to generate a tribble with the "raw" variable names and the "clean" label for presentation. Note that we have to arrange them in the order in which we want them to appear.


```r
coef.list <- tibble::tribble(~raw, ~clean,
                            "b_mu_contact_persYes", "Personal Contact: Yes",
                            "b_mu_contact_persDontknowDdeclinetoanswer", "Personal Contact: DK/Decline",
                            "b_mu_contact_nonpersYes", "Network Contact: Yes",
                            "b_mu_contact_nonpersDontknowDdeclinetoanswer", "Network Contact: DK/Decline",
                            "b_mu_benefit_persYes", "Personal Benefit: Yes",
                            "b_mu_benefit_persDontknowDdeclinetoanswer", "Personal Benefit: DK/Decline",
                            "b_mu_benefit_nonpersYes", "Network Benefit: Yes",
                            "b_mu_benefit_nonpersDontknowDdeclinetoanswer", "Network Benefit: DK/Decline",
                            "b_mu_age25to34years", "25-34",
                            "b_mu_age35to44years", "35-44",
                            "b_mu_age45to54years", "45-54",
                            "b_mu_age55to64years", "55-65",
                            "b_mu_ageAge65orolder", ">65",
                            "b_mu_income.5.cat21M40%", "21-40",
                            "b_mu_income.5.cat41M60%", "41-60",
                            "b_mu_income.5.cat61M80%", "61-80",
                            "b_mu_income.5.cat81M100%", "81-100",
                            "b_mu_genderFemale", "Female",
                            "b_mu_genderNonMbinary", "Non-binary",
                            "b_mu_genderNoneoftheabove", "None of the above",
                            "b_mu_minorityYes", "Minority: Yes",
                            "b_mu_minorityDeclinetoanswer", "Minoriy: Decline to answer",
                            "b_mu_religCatholicism", "Catholic",
                            "b_mu_religChristianityprotestant", "Protestant",
                            "b_mu_religBuddhism", "Buddhism",
                            "b_mu_religHinduism", "Hindu",
                            "b_mu_religIslam", "Islam",
                            "b_mu_religJudaism", "Judaism",
                            "b_mu_religShinto", "Shinto",
                            "b_mu_religMormonism", "Mormonism",
                            "b_mu_religLocal", "Local Religion",
                            "b_mu_religOther", "Other",
                            "b_mu_religDeclinetoanswer", "Religion: Decline to answer",
                            "b_mu_ed_z", "Education",
                            "b_mu_ideology_z", "Ideology",
                            "b_mu_troops_crime_persYes", "Personal Crime Experience: Yes",
                            "b_mu_american_inf_1DontknowDdeclinetoanswer", "Influence 1: DK/Decline",
                            "b_mu_american_inf_1Alittle", "Influence 1: A little",
                            "b_mu_american_inf_1Some", "Influence 1: Some",
                            "b_mu_american_inf_1Alot", "Influence 1: A lot",
                            "b_mu_american_inf_2DontknowDdeclinetoanswer", "Influence 2: DK/Decline",
                            "b_mu_american_inf_2Veryative", "Influence 2: Very negative",
                            "b_mu_american_inf_2Negative", "Influence 2: Negative",
                            "b_mu_american_inf_2Positive", "Influence 2: Positive",
                            "b_mu_american_inf_2Veryitive", "Influence 2: Very positive",
                            "b_mu_basecount_z", "Base count",
                            "b_mu_gdp_z", "GDP",
                            "b_mu_pop_z", "Population",
                            "b_mu_troops_z", "Troop deployment size",
                            "b_mu_Intercept", "Intercept")
```

Next we're going to add a new column that contains the grouping list. Since we're using lots of categorical predictor variables we want to make sure they're grouped in a sensible way. 


```r
coef.list <- coef.list |> 
  mutate(group = case_when(
           grepl(".*ontact.*", raw) ~ "Contact Status",
           grepl(".*enefit.*", raw) ~ "Economic Benefits",
           grepl(".*age.*", raw) ~ "Age",
           grepl(".*ncome.*", raw) ~ "Income Quintile",
           grepl(".*gender.*", raw) ~ "Gender Identification",
           grepl(".*minority.*", raw) ~ "Minority Self-Identification",
           grepl(".*relig.*", raw) ~ "Religious Identification",
           grepl(".*ed_z.*", raw) ~ "Education",
           grepl(".*ideology_z.*", raw) ~ "Ideology",
           grepl(".*crime.*", raw) ~ "Crime Experience",
           grepl(".*inf_1.*", raw) ~ "American Influence (Amount)",
           grepl(".*inf_2.*", raw) ~ "American Influence (Quality)",
           TRUE ~ "Group-Level Variables"
         )) 
```

Next, because we're dealing with a really wide table we'll have coefficients with standard errors underneath. This means that each variable name is actually going to take up two lines. So we need to do a little extra here to properly format this bit. I have to admit I think Vincent actually came up with this particular solution as I was puttering with it for a while. So more props to him!


```r
# Find how long the coefficient list is for the final table hline
last.line <- length(coef.list[[1]]) * 2

coef_map <- setNames(coef.list$clean, coef.list$raw)
idx <- rle(coef.list$group)
idx <- setNames(idx$lengths  * 2, idx$values)
```

# Putting it all together

Finally, we've done all of the prep work. Now we can generate the actual table itself! I'm going to change just a couple of things here. Since the output is going to appear  on a webpage, I'll change the "output" argument to HTML. I'll also delete the `save_kable()` bit that tells it to save to a latex file, but you can add that in at the end if you want \latex output. 

You can see that we use the `last.line` values from the previous chunk in that last `row_spec()` line to tell it where to put an horizontal line.


```r
modelsummary::modelsummary(mod.list,
                  estimate = "{estimate}",
                  statistic = "conf.int",
                  fmt = 2,
                  group = term ~ model + y.level,
                  gof_map = gof.check,
                  coef_map = coef_map,
                  add_rows = rows.com,
                  stars = FALSE,
                  output = "kableExtra",
                  caption = "Bayesian multilevel multinomial logistic regressions. Population level effects. \\label{tab:contactfull}") |> 
  kable_styling(bootstrap_options = c("striped"), font_size = 10, position = "center", full_width = FALSE) |>
  add_header_above(c(" ", "US Presence" = 3, "US People" = 3, "US Government" = 3)) |> 
  group_rows(index = idx, bold = TRUE, background = "gray", color = "white", hline_after = TRUE)  |>  
  row_spec(last.line, hline_after = TRUE) |> 
  column_spec(1, width = "5cm") |>
  column_spec(2:10, width = "3cm") |> 
  kableExtra::scroll_box(width = "100%", height = "600px") 
```

<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:600px; overflow-x: scroll; width:100%; "><table class="table table table-striped" style="width: auto !important; margin-left: auto; margin-right: auto; font-size: 10px; width: auto !important; margin-left: auto; margin-right: auto;">
<caption style="font-size: initial !important;">(\#tab:table output)Bayesian multilevel multinomial logistic regressions. Population level effects. \label{tab:contactfull}</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;position: sticky; top:0; background-color: #FFFFFF;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; position: sticky; top:0; background-color: #FFFFFF;" colspan="3"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">US Presence</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; position: sticky; top:0; background-color: #FFFFFF;" colspan="3"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">US People</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; position: sticky; top:0; background-color: #FFFFFF;" colspan="3"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">US Government</div></th>
</tr>
  <tr>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;">   </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Don't Know </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Negative </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Positive </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Don't Know  </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Negative  </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Positive  </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Don't Know   </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Negative   </th>
   <th style="text-align:center;position: sticky; top:0; background-color: #FFFFFF;"> / Positive   </th>
  </tr>
 </thead>
<tbody>
  <tr grouplength="8"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Contact Status</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Personal Contact: Yes </td>
   <td style="text-align:center;width: 3cm; "> −0.61 </td>
   <td style="text-align:center;width: 3cm; "> 0.16 </td>
   <td style="text-align:center;width: 3cm; "> 0.48 </td>
   <td style="text-align:center;width: 3cm; "> −0.09 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.26 </td>
   <td style="text-align:center;width: 3cm; "> −0.23 </td>
   <td style="text-align:center;width: 3cm; "> 0.15 </td>
   <td style="text-align:center;width: 3cm; "> 0.21 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.94, −0.28] </td>
   <td style="text-align:center;width: 3cm; "> [0.05, 0.27] </td>
   <td style="text-align:center;width: 3cm; "> [0.39, 0.57] </td>
   <td style="text-align:center;width: 3cm; "> [−0.49, 0.29] </td>
   <td style="text-align:center;width: 3cm; "> [−0.06, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [0.17, 0.34] </td>
   <td style="text-align:center;width: 3cm; "> [−0.61, 0.13] </td>
   <td style="text-align:center;width: 3cm; "> [0.04, 0.25] </td>
   <td style="text-align:center;width: 3cm; "> [0.11, 0.30] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Personal Contact: DK/Decline </td>
   <td style="text-align:center;width: 3cm; "> 0.50 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> −0.31 </td>
   <td style="text-align:center;width: 3cm; "> 0.41 </td>
   <td style="text-align:center;width: 3cm; "> −0.19 </td>
   <td style="text-align:center;width: 3cm; "> −0.41 </td>
   <td style="text-align:center;width: 3cm; "> 0.08 </td>
   <td style="text-align:center;width: 3cm; "> −0.49 </td>
   <td style="text-align:center;width: 3cm; "> −0.40 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.23, 0.76] </td>
   <td style="text-align:center;width: 3cm; "> [−0.17, 0.24] </td>
   <td style="text-align:center;width: 3cm; "> [−0.50, −0.13] </td>
   <td style="text-align:center;width: 3cm; "> [0.08, 0.75] </td>
   <td style="text-align:center;width: 3cm; "> [−0.41, 0.03] </td>
   <td style="text-align:center;width: 3cm; "> [−0.59, −0.24] </td>
   <td style="text-align:center;width: 3cm; "> [−0.26, 0.41] </td>
   <td style="text-align:center;width: 3cm; "> [−0.68, −0.29] </td>
   <td style="text-align:center;width: 3cm; "> [−0.59, −0.20] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Network Contact: Yes </td>
   <td style="text-align:center;width: 3cm; "> −0.72 </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> 0.27 </td>
   <td style="text-align:center;width: 3cm; "> −0.19 </td>
   <td style="text-align:center;width: 3cm; "> 0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.20 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.26 </td>
   <td style="text-align:center;width: 3cm; "> 0.20 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−1.02, −0.43] </td>
   <td style="text-align:center;width: 3cm; "> [−0.01, 0.20] </td>
   <td style="text-align:center;width: 3cm; "> [0.18, 0.35] </td>
   <td style="text-align:center;width: 3cm; "> [−0.57, 0.18] </td>
   <td style="text-align:center;width: 3cm; "> [0.06, 0.28] </td>
   <td style="text-align:center;width: 3cm; "> [0.12, 0.28] </td>
   <td style="text-align:center;width: 3cm; "> [−0.53, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [0.16, 0.36] </td>
   <td style="text-align:center;width: 3cm; "> [0.11, 0.29] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Network Contact: DK/Decline </td>
   <td style="text-align:center;width: 3cm; "> −0.39 </td>
   <td style="text-align:center;width: 3cm; "> 0.01 </td>
   <td style="text-align:center;width: 3cm; "> 0.08 </td>
   <td style="text-align:center;width: 3cm; "> 0.39 </td>
   <td style="text-align:center;width: 3cm; "> 0.02 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> 0.29 </td>
   <td style="text-align:center;width: 3cm; "> −0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.16 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.64, −0.15] </td>
   <td style="text-align:center;width: 3cm; "> [−0.13, 0.16] </td>
   <td style="text-align:center;width: 3cm; "> [−0.04, 0.20] </td>
   <td style="text-align:center;width: 3cm; "> [0.09, 0.69] </td>
   <td style="text-align:center;width: 3cm; "> [−0.14, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.16] </td>
   <td style="text-align:center;width: 3cm; "> [−0.01, 0.58] </td>
   <td style="text-align:center;width: 3cm; "> [−0.19, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [0.03, 0.30] </td>
  </tr>
  <tr grouplength="8"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Economic Benefits</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Personal Benefit: Yes </td>
   <td style="text-align:center;width: 3cm; "> −0.59 </td>
   <td style="text-align:center;width: 3cm; "> −0.50 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.37 </td>
   <td style="text-align:center;width: 3cm; "> 0.00 </td>
   <td style="text-align:center;width: 3cm; "> 0.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.15 </td>
   <td style="text-align:center;width: 3cm; "> 0.20 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−1.05, −0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.68, −0.31] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.06, 0.79] </td>
   <td style="text-align:center;width: 3cm; "> [−0.17, 0.18] </td>
   <td style="text-align:center;width: 3cm; "> [0.04, 0.27] </td>
   <td style="text-align:center;width: 3cm; "> [−0.46, 0.39] </td>
   <td style="text-align:center;width: 3cm; "> [−0.30, 0.00] </td>
   <td style="text-align:center;width: 3cm; "> [0.08, 0.32] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Personal Benefit: DK/Decline </td>
   <td style="text-align:center;width: 3cm; "> 0.48 </td>
   <td style="text-align:center;width: 3cm; "> −0.44 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.32 </td>
   <td style="text-align:center;width: 3cm; "> −0.32 </td>
   <td style="text-align:center;width: 3cm; "> −0.10 </td>
   <td style="text-align:center;width: 3cm; "> 0.23 </td>
   <td style="text-align:center;width: 3cm; "> −0.42 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.26, 0.69] </td>
   <td style="text-align:center;width: 3cm; "> [−0.62, −0.25] </td>
   <td style="text-align:center;width: 3cm; "> [−0.32, −0.02] </td>
   <td style="text-align:center;width: 3cm; "> [0.02, 0.63] </td>
   <td style="text-align:center;width: 3cm; "> [−0.52, −0.11] </td>
   <td style="text-align:center;width: 3cm; "> [−0.25, 0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.05, 0.51] </td>
   <td style="text-align:center;width: 3cm; "> [−0.59, −0.26] </td>
   <td style="text-align:center;width: 3cm; "> [−0.18, 0.14] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Network Benefit: Yes </td>
   <td style="text-align:center;width: 3cm; "> −0.55 </td>
   <td style="text-align:center;width: 3cm; "> −0.26 </td>
   <td style="text-align:center;width: 3cm; "> 0.38 </td>
   <td style="text-align:center;width: 3cm; "> −0.23 </td>
   <td style="text-align:center;width: 3cm; "> 0.10 </td>
   <td style="text-align:center;width: 3cm; "> 0.11 </td>
   <td style="text-align:center;width: 3cm; "> −0.39 </td>
   <td style="text-align:center;width: 3cm; "> −0.22 </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−1.00, −0.13] </td>
   <td style="text-align:center;width: 3cm; "> [−0.41, −0.10] </td>
   <td style="text-align:center;width: 3cm; "> [0.27, 0.48] </td>
   <td style="text-align:center;width: 3cm; "> [−0.73, 0.24] </td>
   <td style="text-align:center;width: 3cm; "> [−0.05, 0.25] </td>
   <td style="text-align:center;width: 3cm; "> [0.01, 0.22] </td>
   <td style="text-align:center;width: 3cm; "> [−0.86, 0.06] </td>
   <td style="text-align:center;width: 3cm; "> [−0.35, −0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.02, 0.21] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Network Benefit: DK/Decline </td>
   <td style="text-align:center;width: 3cm; "> 0.25 </td>
   <td style="text-align:center;width: 3cm; "> −0.38 </td>
   <td style="text-align:center;width: 3cm; "> −0.09 </td>
   <td style="text-align:center;width: 3cm; "> 0.18 </td>
   <td style="text-align:center;width: 3cm; "> 0.08 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> −0.06 </td>
   <td style="text-align:center;width: 3cm; "> 0.01 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.03, 0.47] </td>
   <td style="text-align:center;width: 3cm; "> [−0.54, −0.22] </td>
   <td style="text-align:center;width: 3cm; "> [−0.22, 0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.11, 0.47] </td>
   <td style="text-align:center;width: 3cm; "> [−0.09, 0.25] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.18] </td>
   <td style="text-align:center;width: 3cm; "> [−0.24, 0.31] </td>
   <td style="text-align:center;width: 3cm; "> [−0.20, 0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.13, 0.16] </td>
  </tr>
  <tr grouplength="10"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Age</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 25-34 </td>
   <td style="text-align:center;width: 3cm; "> −0.12 </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> −0.11 </td>
   <td style="text-align:center;width: 3cm; "> 0.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.06 </td>
   <td style="text-align:center;width: 3cm; "> −0.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.16 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.26, 0.03] </td>
   <td style="text-align:center;width: 3cm; "> [0.01, 0.18] </td>
   <td style="text-align:center;width: 3cm; "> [−0.19, −0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.08, 0.37] </td>
   <td style="text-align:center;width: 3cm; "> [−0.26, −0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.01, 0.13] </td>
   <td style="text-align:center;width: 3cm; "> [−0.36, 0.06] </td>
   <td style="text-align:center;width: 3cm; "> [−0.24, −0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.03, 0.14] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 35-44 </td>
   <td style="text-align:center;width: 3cm; "> −0.26 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
   <td style="text-align:center;width: 3cm; "> −0.08 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.41 </td>
   <td style="text-align:center;width: 3cm; "> 0.07 </td>
   <td style="text-align:center;width: 3cm; "> −0.18 </td>
   <td style="text-align:center;width: 3cm; "> −0.22 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.41, −0.11] </td>
   <td style="text-align:center;width: 3cm; "> [−0.09, 0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.16, −0.01] </td>
   <td style="text-align:center;width: 3cm; "> [−0.26, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [−0.50, −0.31] </td>
   <td style="text-align:center;width: 3cm; "> [0.00, 0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.41, 0.03] </td>
   <td style="text-align:center;width: 3cm; "> [−0.31, −0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.04, 0.12] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 45-54 </td>
   <td style="text-align:center;width: 3cm; "> −0.31 </td>
   <td style="text-align:center;width: 3cm; "> −0.11 </td>
   <td style="text-align:center;width: 3cm; "> −0.05 </td>
   <td style="text-align:center;width: 3cm; "> −0.03 </td>
   <td style="text-align:center;width: 3cm; "> −0.37 </td>
   <td style="text-align:center;width: 3cm; "> 0.19 </td>
   <td style="text-align:center;width: 3cm; "> −0.25 </td>
   <td style="text-align:center;width: 3cm; "> −0.10 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.46, −0.15] </td>
   <td style="text-align:center;width: 3cm; "> [−0.20, −0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−0.13, 0.03] </td>
   <td style="text-align:center;width: 3cm; "> [−0.28, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [−0.46, −0.28] </td>
   <td style="text-align:center;width: 3cm; "> [0.12, 0.26] </td>
   <td style="text-align:center;width: 3cm; "> [−0.49, −0.03] </td>
   <td style="text-align:center;width: 3cm; "> [−0.18, −0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−0.04, 0.13] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 55-65 </td>
   <td style="text-align:center;width: 3cm; "> −0.59 </td>
   <td style="text-align:center;width: 3cm; "> −0.19 </td>
   <td style="text-align:center;width: 3cm; "> 0.11 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.55 </td>
   <td style="text-align:center;width: 3cm; "> 0.31 </td>
   <td style="text-align:center;width: 3cm; "> −0.18 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> 0.12 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.76, −0.42] </td>
   <td style="text-align:center;width: 3cm; "> [−0.28, −0.10] </td>
   <td style="text-align:center;width: 3cm; "> [0.03, 0.18] </td>
   <td style="text-align:center;width: 3cm; "> [−0.30, 0.25] </td>
   <td style="text-align:center;width: 3cm; "> [−0.65, −0.45] </td>
   <td style="text-align:center;width: 3cm; "> [0.24, 0.38] </td>
   <td style="text-align:center;width: 3cm; "> [−0.43, 0.06] </td>
   <td style="text-align:center;width: 3cm; "> [−0.04, 0.13] </td>
   <td style="text-align:center;width: 3cm; "> [0.03, 0.21] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> &gt;65 </td>
   <td style="text-align:center;width: 3cm; "> −0.78 </td>
   <td style="text-align:center;width: 3cm; "> −0.07 </td>
   <td style="text-align:center;width: 3cm; "> 0.19 </td>
   <td style="text-align:center;width: 3cm; "> 0.24 </td>
   <td style="text-align:center;width: 3cm; "> −0.38 </td>
   <td style="text-align:center;width: 3cm; "> 0.41 </td>
   <td style="text-align:center;width: 3cm; "> −0.14 </td>
   <td style="text-align:center;width: 3cm; "> 0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.18 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.98, −0.58] </td>
   <td style="text-align:center;width: 3cm; "> [−0.17, 0.03] </td>
   <td style="text-align:center;width: 3cm; "> [0.11, 0.27] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.53] </td>
   <td style="text-align:center;width: 3cm; "> [−0.49, −0.27] </td>
   <td style="text-align:center;width: 3cm; "> [0.33, 0.49] </td>
   <td style="text-align:center;width: 3cm; "> [−0.44, 0.14] </td>
   <td style="text-align:center;width: 3cm; "> [0.08, 0.26] </td>
   <td style="text-align:center;width: 3cm; "> [0.08, 0.27] </td>
  </tr>
  <tr grouplength="8"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Income Quintile</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 21-40 </td>
   <td style="text-align:center;width: 3cm; "> −0.18 </td>
   <td style="text-align:center;width: 3cm; "> −0.06 </td>
   <td style="text-align:center;width: 3cm; "> −0.03 </td>
   <td style="text-align:center;width: 3cm; "> −0.16 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
   <td style="text-align:center;width: 3cm; "> 0.01 </td>
   <td style="text-align:center;width: 3cm; "> −0.25 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.03 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.34, −0.01] </td>
   <td style="text-align:center;width: 3cm; "> [−0.15, 0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.12, 0.05] </td>
   <td style="text-align:center;width: 3cm; "> [−0.40, 0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.12, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.48, −0.03] </td>
   <td style="text-align:center;width: 3cm; "> [−0.05, 0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.12] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 41-60 </td>
   <td style="text-align:center;width: 3cm; "> −0.07 </td>
   <td style="text-align:center;width: 3cm; "> 0.00 </td>
   <td style="text-align:center;width: 3cm; "> 0.01 </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> 0.11 </td>
   <td style="text-align:center;width: 3cm; "> 0.12 </td>
   <td style="text-align:center;width: 3cm; "> −0.25 </td>
   <td style="text-align:center;width: 3cm; "> 0.12 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.23, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.09, 0.10] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.45, 0.03] </td>
   <td style="text-align:center;width: 3cm; "> [0.00, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [0.04, 0.19] </td>
   <td style="text-align:center;width: 3cm; "> [−0.47, −0.03] </td>
   <td style="text-align:center;width: 3cm; "> [0.03, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [−0.10, 0.08] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 61-80 </td>
   <td style="text-align:center;width: 3cm; "> −0.33 </td>
   <td style="text-align:center;width: 3cm; "> −0.03 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
   <td style="text-align:center;width: 3cm; "> −0.53 </td>
   <td style="text-align:center;width: 3cm; "> 0.12 </td>
   <td style="text-align:center;width: 3cm; "> 0.14 </td>
   <td style="text-align:center;width: 3cm; "> −0.62 </td>
   <td style="text-align:center;width: 3cm; "> 0.16 </td>
   <td style="text-align:center;width: 3cm; "> 0.01 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.52, −0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.13, 0.07] </td>
   <td style="text-align:center;width: 3cm; "> [−0.10, 0.07] </td>
   <td style="text-align:center;width: 3cm; "> [−0.85, −0.24] </td>
   <td style="text-align:center;width: 3cm; "> [0.01, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [0.05, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [−0.91, −0.33] </td>
   <td style="text-align:center;width: 3cm; "> [0.07, 0.26] </td>
   <td style="text-align:center;width: 3cm; "> [−0.08, 0.11] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> 81-100 </td>
   <td style="text-align:center;width: 3cm; "> −0.42 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> 0.06 </td>
   <td style="text-align:center;width: 3cm; "> −0.74 </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> 0.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.75 </td>
   <td style="text-align:center;width: 3cm; "> 0.27 </td>
   <td style="text-align:center;width: 3cm; "> 0.11 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.63, −0.21] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.03, 0.15] </td>
   <td style="text-align:center;width: 3cm; "> [−1.12, −0.39] </td>
   <td style="text-align:center;width: 3cm; "> [−0.03, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [0.07, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [−1.10, −0.41] </td>
   <td style="text-align:center;width: 3cm; "> [0.17, 0.37] </td>
   <td style="text-align:center;width: 3cm; "> [0.00, 0.21] </td>
  </tr>
  <tr grouplength="6"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Gender Identification</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Female </td>
   <td style="text-align:center;width: 3cm; "> 0.38 </td>
   <td style="text-align:center;width: 3cm; "> −0.03 </td>
   <td style="text-align:center;width: 3cm; "> −0.22 </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> −0.05 </td>
   <td style="text-align:center;width: 3cm; "> −0.06 </td>
   <td style="text-align:center;width: 3cm; "> 0.21 </td>
   <td style="text-align:center;width: 3cm; "> 0.06 </td>
   <td style="text-align:center;width: 3cm; "> −0.10 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.28, 0.49] </td>
   <td style="text-align:center;width: 3cm; "> [−0.08, 0.03] </td>
   <td style="text-align:center;width: 3cm; "> [−0.27, −0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.24] </td>
   <td style="text-align:center;width: 3cm; "> [−0.11, 0.01] </td>
   <td style="text-align:center;width: 3cm; "> [−0.10, −0.02] </td>
   <td style="text-align:center;width: 3cm; "> [0.07, 0.36] </td>
   <td style="text-align:center;width: 3cm; "> [0.02, 0.11] </td>
   <td style="text-align:center;width: 3cm; "> [−0.15, −0.05] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Non-binary </td>
   <td style="text-align:center;width: 3cm; "> −1.30 </td>
   <td style="text-align:center;width: 3cm; "> 0.41 </td>
   <td style="text-align:center;width: 3cm; "> −0.18 </td>
   <td style="text-align:center;width: 3cm; "> −0.69 </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> −0.30 </td>
   <td style="text-align:center;width: 3cm; "> 1.42 </td>
   <td style="text-align:center;width: 3cm; "> 0.48 </td>
   <td style="text-align:center;width: 3cm; "> −0.18 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−4.92, 0.49] </td>
   <td style="text-align:center;width: 3cm; "> [−0.18, 0.99] </td>
   <td style="text-align:center;width: 3cm; "> [−0.63, 0.27] </td>
   <td style="text-align:center;width: 3cm; "> [−4.07, 1.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.86, 0.38] </td>
   <td style="text-align:center;width: 3cm; "> [−0.71, 0.12] </td>
   <td style="text-align:center;width: 3cm; "> [0.40, 2.29] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 1.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.68, 0.33] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> None of the above </td>
   <td style="text-align:center;width: 3cm; "> 0.20 </td>
   <td style="text-align:center;width: 3cm; "> −0.47 </td>
   <td style="text-align:center;width: 3cm; "> −0.68 </td>
   <td style="text-align:center;width: 3cm; "> 1.24 </td>
   <td style="text-align:center;width: 3cm; "> 0.52 </td>
   <td style="text-align:center;width: 3cm; "> 1.37 </td>
   <td style="text-align:center;width: 3cm; "> −1.50 </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> −0.66 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.95, 1.24] </td>
   <td style="text-align:center;width: 3cm; "> [−1.42, 0.42] </td>
   <td style="text-align:center;width: 3cm; "> [−1.46, 0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.42, 2.75] </td>
   <td style="text-align:center;width: 3cm; "> [−0.94, 1.82] </td>
   <td style="text-align:center;width: 3cm; "> [0.51, 2.30] </td>
   <td style="text-align:center;width: 3cm; "> [−4.99, 0.55] </td>
   <td style="text-align:center;width: 3cm; "> [−0.97, 0.53] </td>
   <td style="text-align:center;width: 3cm; "> [−1.47, 0.16] </td>
  </tr>
  <tr grouplength="4"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Minority Self-Identification</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Minority: Yes </td>
   <td style="text-align:center;width: 3cm; "> −0.12 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> −0.05 </td>
   <td style="text-align:center;width: 3cm; "> −0.04 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> −0.08 </td>
   <td style="text-align:center;width: 3cm; "> −0.05 </td>
   <td style="text-align:center;width: 3cm; "> −0.14 </td>
   <td style="text-align:center;width: 3cm; "> −0.07 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.30, 0.05] </td>
   <td style="text-align:center;width: 3cm; "> [−0.04, 0.13] </td>
   <td style="text-align:center;width: 3cm; "> [−0.13, 0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−0.29, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [−0.06, 0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.15, −0.01] </td>
   <td style="text-align:center;width: 3cm; "> [−0.28, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.22, −0.06] </td>
   <td style="text-align:center;width: 3cm; "> [−0.15, 0.01] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Minoriy: Decline to answer </td>
   <td style="text-align:center;width: 3cm; "> 0.53 </td>
   <td style="text-align:center;width: 3cm; "> −0.10 </td>
   <td style="text-align:center;width: 3cm; "> −0.06 </td>
   <td style="text-align:center;width: 3cm; "> 0.78 </td>
   <td style="text-align:center;width: 3cm; "> 0.00 </td>
   <td style="text-align:center;width: 3cm; "> −0.15 </td>
   <td style="text-align:center;width: 3cm; "> 0.46 </td>
   <td style="text-align:center;width: 3cm; "> −0.11 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.31, 0.74] </td>
   <td style="text-align:center;width: 3cm; "> [−0.28, 0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.22, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [0.51, 1.05] </td>
   <td style="text-align:center;width: 3cm; "> [−0.19, 0.18] </td>
   <td style="text-align:center;width: 3cm; "> [−0.29, −0.01] </td>
   <td style="text-align:center;width: 3cm; "> [0.19, 0.71] </td>
   <td style="text-align:center;width: 3cm; "> [−0.27, 0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.34, −0.01] </td>
  </tr>
  <tr grouplength="22"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Religious Identification</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Catholic </td>
   <td style="text-align:center;width: 3cm; "> 0.06 </td>
   <td style="text-align:center;width: 3cm; "> −0.42 </td>
   <td style="text-align:center;width: 3cm; "> 0.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.06 </td>
   <td style="text-align:center;width: 3cm; "> −0.28 </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> 0.27 </td>
   <td style="text-align:center;width: 3cm; "> −0.39 </td>
   <td style="text-align:center;width: 3cm; "> −0.03 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.10, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [−0.51, −0.34] </td>
   <td style="text-align:center;width: 3cm; "> [0.07, 0.22] </td>
   <td style="text-align:center;width: 3cm; "> [−0.33, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [−0.37, −0.19] </td>
   <td style="text-align:center;width: 3cm; "> [0.02, 0.16] </td>
   <td style="text-align:center;width: 3cm; "> [0.00, 0.54] </td>
   <td style="text-align:center;width: 3cm; "> [−0.48, −0.31] </td>
   <td style="text-align:center;width: 3cm; "> [−0.12, 0.06] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Protestant </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> −0.46 </td>
   <td style="text-align:center;width: 3cm; "> 0.13 </td>
   <td style="text-align:center;width: 3cm; "> 0.16 </td>
   <td style="text-align:center;width: 3cm; "> −0.27 </td>
   <td style="text-align:center;width: 3cm; "> 0.03 </td>
   <td style="text-align:center;width: 3cm; "> 0.41 </td>
   <td style="text-align:center;width: 3cm; "> −0.41 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.14, 0.31] </td>
   <td style="text-align:center;width: 3cm; "> [−0.58, −0.33] </td>
   <td style="text-align:center;width: 3cm; "> [0.03, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [−0.23, 0.53] </td>
   <td style="text-align:center;width: 3cm; "> [−0.40, −0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.06, 0.12] </td>
   <td style="text-align:center;width: 3cm; "> [0.05, 0.75] </td>
   <td style="text-align:center;width: 3cm; "> [−0.52, −0.29] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.16] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Buddhism </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> −0.22 </td>
   <td style="text-align:center;width: 3cm; "> 0.06 </td>
   <td style="text-align:center;width: 3cm; "> 0.03 </td>
   <td style="text-align:center;width: 3cm; "> −0.20 </td>
   <td style="text-align:center;width: 3cm; "> 0.14 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> −0.39 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.27, 0.43] </td>
   <td style="text-align:center;width: 3cm; "> [−0.37, −0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.19] </td>
   <td style="text-align:center;width: 3cm; "> [−0.47, 0.50] </td>
   <td style="text-align:center;width: 3cm; "> [−0.40, 0.00] </td>
   <td style="text-align:center;width: 3cm; "> [0.02, 0.26] </td>
   <td style="text-align:center;width: 3cm; "> [−0.47, 0.54] </td>
   <td style="text-align:center;width: 3cm; "> [−0.53, −0.24] </td>
   <td style="text-align:center;width: 3cm; "> [−0.09, 0.18] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Hindu </td>
   <td style="text-align:center;width: 3cm; "> 0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.20 </td>
   <td style="text-align:center;width: 3cm; "> −0.23 </td>
   <td style="text-align:center;width: 3cm; "> 0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.37 </td>
   <td style="text-align:center;width: 3cm; "> −0.07 </td>
   <td style="text-align:center;width: 3cm; "> 0.44 </td>
   <td style="text-align:center;width: 3cm; "> −0.40 </td>
   <td style="text-align:center;width: 3cm; "> −0.16 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.72, 0.67] </td>
   <td style="text-align:center;width: 3cm; "> [−0.65, 0.22] </td>
   <td style="text-align:center;width: 3cm; "> [−0.51, 0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−1.17, 0.99] </td>
   <td style="text-align:center;width: 3cm; "> [−0.87, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.33, 0.19] </td>
   <td style="text-align:center;width: 3cm; "> [−0.50, 1.28] </td>
   <td style="text-align:center;width: 3cm; "> [−0.77, −0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.44, 0.13] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Islam </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> 0.29 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.23 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> −0.13 </td>
   <td style="text-align:center;width: 3cm; "> 0.41 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> −0.29 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.25, 0.32] </td>
   <td style="text-align:center;width: 3cm; "> [0.13, 0.45] </td>
   <td style="text-align:center;width: 3cm; "> [−0.31, −0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−0.16, 0.62] </td>
   <td style="text-align:center;width: 3cm; "> [−0.11, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [−0.25, 0.00] </td>
   <td style="text-align:center;width: 3cm; "> [0.05, 0.78] </td>
   <td style="text-align:center;width: 3cm; "> [−0.12, 0.20] </td>
   <td style="text-align:center;width: 3cm; "> [−0.44, −0.14] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Judaism </td>
   <td style="text-align:center;width: 3cm; "> 0.76 </td>
   <td style="text-align:center;width: 3cm; "> −0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.18 </td>
   <td style="text-align:center;width: 3cm; "> −0.80 </td>
   <td style="text-align:center;width: 3cm; "> −1.67 </td>
   <td style="text-align:center;width: 3cm; "> 0.09 </td>
   <td style="text-align:center;width: 3cm; "> 1.68 </td>
   <td style="text-align:center;width: 3cm; "> −0.56 </td>
   <td style="text-align:center;width: 3cm; "> 0.29 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.01, 1.45] </td>
   <td style="text-align:center;width: 3cm; "> [−0.62, 0.49] </td>
   <td style="text-align:center;width: 3cm; "> [−0.11, 0.48] </td>
   <td style="text-align:center;width: 3cm; "> [−2.80, 0.72] </td>
   <td style="text-align:center;width: 3cm; "> [−2.81, −0.74] </td>
   <td style="text-align:center;width: 3cm; "> [−0.18, 0.36] </td>
   <td style="text-align:center;width: 3cm; "> [0.88, 2.42] </td>
   <td style="text-align:center;width: 3cm; "> [−1.07, −0.07] </td>
   <td style="text-align:center;width: 3cm; "> [−0.03, 0.60] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Shinto </td>
   <td style="text-align:center;width: 3cm; "> −83.92 </td>
   <td style="text-align:center;width: 3cm; "> −0.28 </td>
   <td style="text-align:center;width: 3cm; "> −0.03 </td>
   <td style="text-align:center;width: 3cm; "> −82.55 </td>
   <td style="text-align:center;width: 3cm; "> −0.14 </td>
   <td style="text-align:center;width: 3cm; "> 0.11 </td>
   <td style="text-align:center;width: 3cm; "> 1.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.71 </td>
   <td style="text-align:center;width: 3cm; "> 0.31 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−196.62, −7.75] </td>
   <td style="text-align:center;width: 3cm; "> [−0.74, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.41, 0.36] </td>
   <td style="text-align:center;width: 3cm; "> [−194.75, −7.27] </td>
   <td style="text-align:center;width: 3cm; "> [−0.83, 0.49] </td>
   <td style="text-align:center;width: 3cm; "> [−0.26, 0.49] </td>
   <td style="text-align:center;width: 3cm; "> [0.05, 2.10] </td>
   <td style="text-align:center;width: 3cm; "> [−1.28, −0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.08, 0.70] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Mormonism </td>
   <td style="text-align:center;width: 3cm; "> −86.86 </td>
   <td style="text-align:center;width: 3cm; "> −0.20 </td>
   <td style="text-align:center;width: 3cm; "> 0.46 </td>
   <td style="text-align:center;width: 3cm; "> −83.83 </td>
   <td style="text-align:center;width: 3cm; "> −0.80 </td>
   <td style="text-align:center;width: 3cm; "> 0.29 </td>
   <td style="text-align:center;width: 3cm; "> 0.27 </td>
   <td style="text-align:center;width: 3cm; "> −0.07 </td>
   <td style="text-align:center;width: 3cm; "> 0.47 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−202.31, −9.58] </td>
   <td style="text-align:center;width: 3cm; "> [−1.03, 0.58] </td>
   <td style="text-align:center;width: 3cm; "> [−0.17, 1.12] </td>
   <td style="text-align:center;width: 3cm; "> [−197.04, −7.49] </td>
   <td style="text-align:center;width: 3cm; "> [−1.78, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.29, 0.89] </td>
   <td style="text-align:center;width: 3cm; "> [−3.24, 2.35] </td>
   <td style="text-align:center;width: 3cm; "> [−0.91, 0.77] </td>
   <td style="text-align:center;width: 3cm; "> [−0.28, 1.27] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Local Religion </td>
   <td style="text-align:center;width: 3cm; "> −0.66 </td>
   <td style="text-align:center;width: 3cm; "> −0.37 </td>
   <td style="text-align:center;width: 3cm; "> −0.19 </td>
   <td style="text-align:center;width: 3cm; "> 0.95 </td>
   <td style="text-align:center;width: 3cm; "> −0.49 </td>
   <td style="text-align:center;width: 3cm; "> −0.11 </td>
   <td style="text-align:center;width: 3cm; "> 1.20 </td>
   <td style="text-align:center;width: 3cm; "> −1.16 </td>
   <td style="text-align:center;width: 3cm; "> −0.06 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−1.63, 0.16] </td>
   <td style="text-align:center;width: 3cm; "> [−0.76, 0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−0.52, 0.15] </td>
   <td style="text-align:center;width: 3cm; "> [0.11, 1.72] </td>
   <td style="text-align:center;width: 3cm; "> [−0.94, −0.07] </td>
   <td style="text-align:center;width: 3cm; "> [−0.44, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [0.43, 1.90] </td>
   <td style="text-align:center;width: 3cm; "> [−1.55, −0.77] </td>
   <td style="text-align:center;width: 3cm; "> [−0.42, 0.30] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Other </td>
   <td style="text-align:center;width: 3cm; "> 0.17 </td>
   <td style="text-align:center;width: 3cm; "> −0.16 </td>
   <td style="text-align:center;width: 3cm; "> −0.12 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> −0.16 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.47 </td>
   <td style="text-align:center;width: 3cm; "> −0.23 </td>
   <td style="text-align:center;width: 3cm; "> −0.06 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.01, 0.36] </td>
   <td style="text-align:center;width: 3cm; "> [−0.25, −0.07] </td>
   <td style="text-align:center;width: 3cm; "> [−0.21, −0.03] </td>
   <td style="text-align:center;width: 3cm; "> [−0.27, 0.36] </td>
   <td style="text-align:center;width: 3cm; "> [−0.26, −0.06] </td>
   <td style="text-align:center;width: 3cm; "> [−0.03, 0.13] </td>
   <td style="text-align:center;width: 3cm; "> [0.16, 0.77] </td>
   <td style="text-align:center;width: 3cm; "> [−0.33, −0.13] </td>
   <td style="text-align:center;width: 3cm; "> [−0.17, 0.05] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Religion: Decline to answer </td>
   <td style="text-align:center;width: 3cm; "> 0.24 </td>
   <td style="text-align:center;width: 3cm; "> −0.25 </td>
   <td style="text-align:center;width: 3cm; "> −0.15 </td>
   <td style="text-align:center;width: 3cm; "> 0.53 </td>
   <td style="text-align:center;width: 3cm; "> −0.31 </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> 0.84 </td>
   <td style="text-align:center;width: 3cm; "> −0.50 </td>
   <td style="text-align:center;width: 3cm; "> −0.24 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.06, 0.42] </td>
   <td style="text-align:center;width: 3cm; "> [−0.36, −0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.26, −0.05] </td>
   <td style="text-align:center;width: 3cm; "> [0.26, 0.79] </td>
   <td style="text-align:center;width: 3cm; "> [−0.43, −0.19] </td>
   <td style="text-align:center;width: 3cm; "> [−0.30, −0.11] </td>
   <td style="text-align:center;width: 3cm; "> [0.57, 1.11] </td>
   <td style="text-align:center;width: 3cm; "> [−0.61, −0.40] </td>
   <td style="text-align:center;width: 3cm; "> [−0.37, −0.12] </td>
  </tr>
  <tr grouplength="2"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Education</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Education </td>
   <td style="text-align:center;width: 3cm; "> −0.04 </td>
   <td style="text-align:center;width: 3cm; "> 0.16 </td>
   <td style="text-align:center;width: 3cm; "> 0.03 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
   <td style="text-align:center;width: 3cm; "> 0.05 </td>
   <td style="text-align:center;width: 3cm; "> 0.17 </td>
   <td style="text-align:center;width: 3cm; "> −0.08 </td>
   <td style="text-align:center;width: 3cm; "> 0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.12 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.16, 0.08] </td>
   <td style="text-align:center;width: 3cm; "> [0.09, 0.22] </td>
   <td style="text-align:center;width: 3cm; "> [−0.02, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.18, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.02, 0.12] </td>
   <td style="text-align:center;width: 3cm; "> [0.11, 0.22] </td>
   <td style="text-align:center;width: 3cm; "> [−0.25, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [0.10, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [0.05, 0.18] </td>
  </tr>
  <tr grouplength="2"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Ideology</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Ideology </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> −0.25 </td>
   <td style="text-align:center;width: 3cm; "> 0.38 </td>
   <td style="text-align:center;width: 3cm; "> −0.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
   <td style="text-align:center;width: 3cm; "> 0.11 </td>
   <td style="text-align:center;width: 3cm; "> −0.22 </td>
   <td style="text-align:center;width: 3cm; "> −0.58 </td>
   <td style="text-align:center;width: 3cm; "> 0.46 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.34, −0.08] </td>
   <td style="text-align:center;width: 3cm; "> [−0.31, −0.18] </td>
   <td style="text-align:center;width: 3cm; "> [0.32, 0.44] </td>
   <td style="text-align:center;width: 3cm; "> [−0.35, 0.04] </td>
   <td style="text-align:center;width: 3cm; "> [−0.08, 0.07] </td>
   <td style="text-align:center;width: 3cm; "> [0.06, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.41, −0.05] </td>
   <td style="text-align:center;width: 3cm; "> [−0.64, −0.51] </td>
   <td style="text-align:center;width: 3cm; "> [0.40, 0.53] </td>
  </tr>
  <tr grouplength="2"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Crime Experience</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Personal Crime Experience: Yes </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> −0.13 </td>
   <td style="text-align:center;width: 3cm; "> −0.11 </td>
   <td style="text-align:center;width: 3cm; "> 0.55 </td>
   <td style="text-align:center;width: 3cm; "> 0.12 </td>
   <td style="text-align:center;width: 3cm; "> −0.28 </td>
   <td style="text-align:center;width: 3cm; "> 0.15 </td>
   <td style="text-align:center;width: 3cm; "> −0.47 </td>
   <td style="text-align:center;width: 3cm; "> −0.18 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.89, 0.38] </td>
   <td style="text-align:center;width: 3cm; "> [−0.40, 0.13] </td>
   <td style="text-align:center;width: 3cm; "> [−0.28, 0.07] </td>
   <td style="text-align:center;width: 3cm; "> [−0.05, 1.11] </td>
   <td style="text-align:center;width: 3cm; "> [−0.11, 0.35] </td>
   <td style="text-align:center;width: 3cm; "> [−0.44, −0.12] </td>
   <td style="text-align:center;width: 3cm; "> [−0.50, 0.73] </td>
   <td style="text-align:center;width: 3cm; "> [−0.69, −0.25] </td>
   <td style="text-align:center;width: 3cm; "> [−0.35, 0.00] </td>
  </tr>
  <tr grouplength="8"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>American Influence (Amount)</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 1: DK/Decline </td>
   <td style="text-align:center;width: 3cm; "> 0.44 </td>
   <td style="text-align:center;width: 3cm; "> −0.82 </td>
   <td style="text-align:center;width: 3cm; "> −0.35 </td>
   <td style="text-align:center;width: 3cm; "> 0.41 </td>
   <td style="text-align:center;width: 3cm; "> −0.75 </td>
   <td style="text-align:center;width: 3cm; "> −0.44 </td>
   <td style="text-align:center;width: 3cm; "> 0.24 </td>
   <td style="text-align:center;width: 3cm; "> −0.61 </td>
   <td style="text-align:center;width: 3cm; "> −0.54 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.20, 0.69] </td>
   <td style="text-align:center;width: 3cm; "> [−1.02, −0.62] </td>
   <td style="text-align:center;width: 3cm; "> [−0.54, −0.16] </td>
   <td style="text-align:center;width: 3cm; "> [0.07, 0.75] </td>
   <td style="text-align:center;width: 3cm; "> [−0.97, −0.54] </td>
   <td style="text-align:center;width: 3cm; "> [−0.61, −0.28] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.55] </td>
   <td style="text-align:center;width: 3cm; "> [−0.79, −0.43] </td>
   <td style="text-align:center;width: 3cm; "> [−0.77, −0.30] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 1: A little </td>
   <td style="text-align:center;width: 3cm; "> −0.33 </td>
   <td style="text-align:center;width: 3cm; "> −0.30 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
   <td style="text-align:center;width: 3cm; "> −0.77 </td>
   <td style="text-align:center;width: 3cm; "> −0.34 </td>
   <td style="text-align:center;width: 3cm; "> −0.10 </td>
   <td style="text-align:center;width: 3cm; "> −1.08 </td>
   <td style="text-align:center;width: 3cm; "> −0.03 </td>
   <td style="text-align:center;width: 3cm; "> 0.22 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.56, −0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.44, −0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.09, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−1.12, −0.42] </td>
   <td style="text-align:center;width: 3cm; "> [−0.47, −0.20] </td>
   <td style="text-align:center;width: 3cm; "> [−0.22, 0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−1.42, −0.74] </td>
   <td style="text-align:center;width: 3cm; "> [−0.16, 0.10] </td>
   <td style="text-align:center;width: 3cm; "> [0.07, 0.38] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 1: Some </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
   <td style="text-align:center;width: 3cm; "> 0.19 </td>
   <td style="text-align:center;width: 3cm; "> −0.75 </td>
   <td style="text-align:center;width: 3cm; "> −0.51 </td>
   <td style="text-align:center;width: 3cm; "> 0.13 </td>
   <td style="text-align:center;width: 3cm; "> −0.67 </td>
   <td style="text-align:center;width: 3cm; "> 0.01 </td>
   <td style="text-align:center;width: 3cm; "> 0.28 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.42, 0.01] </td>
   <td style="text-align:center;width: 3cm; "> [−0.30, −0.05] </td>
   <td style="text-align:center;width: 3cm; "> [0.06, 0.32] </td>
   <td style="text-align:center;width: 3cm; "> [−1.07, −0.42] </td>
   <td style="text-align:center;width: 3cm; "> [−0.64, −0.38] </td>
   <td style="text-align:center;width: 3cm; "> [0.01, 0.24] </td>
   <td style="text-align:center;width: 3cm; "> [−0.96, −0.38] </td>
   <td style="text-align:center;width: 3cm; "> [−0.12, 0.13] </td>
   <td style="text-align:center;width: 3cm; "> [0.13, 0.43] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 1: A lot </td>
   <td style="text-align:center;width: 3cm; "> −0.29 </td>
   <td style="text-align:center;width: 3cm; "> 0.13 </td>
   <td style="text-align:center;width: 3cm; "> 0.53 </td>
   <td style="text-align:center;width: 3cm; "> −0.63 </td>
   <td style="text-align:center;width: 3cm; "> −0.33 </td>
   <td style="text-align:center;width: 3cm; "> 0.55 </td>
   <td style="text-align:center;width: 3cm; "> −0.78 </td>
   <td style="text-align:center;width: 3cm; "> 0.13 </td>
   <td style="text-align:center;width: 3cm; "> 0.61 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.52, −0.05] </td>
   <td style="text-align:center;width: 3cm; "> [−0.01, 0.26] </td>
   <td style="text-align:center;width: 3cm; "> [0.40, 0.66] </td>
   <td style="text-align:center;width: 3cm; "> [−0.98, −0.28] </td>
   <td style="text-align:center;width: 3cm; "> [−0.46, −0.19] </td>
   <td style="text-align:center;width: 3cm; "> [0.43, 0.67] </td>
   <td style="text-align:center;width: 3cm; "> [−1.11, −0.47] </td>
   <td style="text-align:center;width: 3cm; "> [0.00, 0.26] </td>
   <td style="text-align:center;width: 3cm; "> [0.46, 0.77] </td>
  </tr>
  <tr grouplength="10"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>American Influence (Quality)</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 2: DK/Decline </td>
   <td style="text-align:center;width: 3cm; "> 1.90 </td>
   <td style="text-align:center;width: 3cm; "> 0.38 </td>
   <td style="text-align:center;width: 3cm; "> −0.27 </td>
   <td style="text-align:center;width: 3cm; "> 1.69 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
   <td style="text-align:center;width: 3cm; "> −0.28 </td>
   <td style="text-align:center;width: 3cm; "> 1.98 </td>
   <td style="text-align:center;width: 3cm; "> 0.22 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [1.74, 2.06] </td>
   <td style="text-align:center;width: 3cm; "> [0.22, 0.53] </td>
   <td style="text-align:center;width: 3cm; "> [−0.42, −0.12] </td>
   <td style="text-align:center;width: 3cm; "> [1.45, 1.92] </td>
   <td style="text-align:center;width: 3cm; "> [−0.19, 0.16] </td>
   <td style="text-align:center;width: 3cm; "> [−0.40, −0.16] </td>
   <td style="text-align:center;width: 3cm; "> [1.76, 2.20] </td>
   <td style="text-align:center;width: 3cm; "> [0.09, 0.35] </td>
   <td style="text-align:center;width: 3cm; "> [−0.35, 0.01] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 2: Very negative </td>
   <td style="text-align:center;width: 3cm; "> 0.63 </td>
   <td style="text-align:center;width: 3cm; "> 1.97 </td>
   <td style="text-align:center;width: 3cm; "> −0.57 </td>
   <td style="text-align:center;width: 3cm; "> 0.56 </td>
   <td style="text-align:center;width: 3cm; "> 1.88 </td>
   <td style="text-align:center;width: 3cm; "> −0.88 </td>
   <td style="text-align:center;width: 3cm; "> 1.27 </td>
   <td style="text-align:center;width: 3cm; "> 2.83 </td>
   <td style="text-align:center;width: 3cm; "> −0.01 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.35, 0.89] </td>
   <td style="text-align:center;width: 3cm; "> [1.85, 2.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.73, −0.40] </td>
   <td style="text-align:center;width: 3cm; "> [0.15, 0.94] </td>
   <td style="text-align:center;width: 3cm; "> [1.77, 1.99] </td>
   <td style="text-align:center;width: 3cm; "> [−1.00, −0.76] </td>
   <td style="text-align:center;width: 3cm; "> [0.77, 1.74] </td>
   <td style="text-align:center;width: 3cm; "> [2.64, 3.01] </td>
   <td style="text-align:center;width: 3cm; "> [−0.27, 0.24] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 2: Negative </td>
   <td style="text-align:center;width: 3cm; "> 0.28 </td>
   <td style="text-align:center;width: 3cm; "> 1.14 </td>
   <td style="text-align:center;width: 3cm; "> −0.33 </td>
   <td style="text-align:center;width: 3cm; "> −0.15 </td>
   <td style="text-align:center;width: 3cm; "> 1.13 </td>
   <td style="text-align:center;width: 3cm; "> −0.55 </td>
   <td style="text-align:center;width: 3cm; "> 0.54 </td>
   <td style="text-align:center;width: 3cm; "> 1.61 </td>
   <td style="text-align:center;width: 3cm; "> −0.17 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.14, 0.41] </td>
   <td style="text-align:center;width: 3cm; "> [1.07, 1.20] </td>
   <td style="text-align:center;width: 3cm; "> [−0.40, −0.26] </td>
   <td style="text-align:center;width: 3cm; "> [−0.42, 0.12] </td>
   <td style="text-align:center;width: 3cm; "> [1.06, 1.20] </td>
   <td style="text-align:center;width: 3cm; "> [−0.61, −0.49] </td>
   <td style="text-align:center;width: 3cm; "> [0.27, 0.80] </td>
   <td style="text-align:center;width: 3cm; "> [1.54, 1.68] </td>
   <td style="text-align:center;width: 3cm; "> [−0.27, −0.07] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 2: Positive </td>
   <td style="text-align:center;width: 3cm; "> 0.00 </td>
   <td style="text-align:center;width: 3cm; "> −0.28 </td>
   <td style="text-align:center;width: 3cm; "> 1.24 </td>
   <td style="text-align:center;width: 3cm; "> 0.27 </td>
   <td style="text-align:center;width: 3cm; "> −0.16 </td>
   <td style="text-align:center;width: 3cm; "> 1.31 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.41 </td>
   <td style="text-align:center;width: 3cm; "> 1.35 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.16, 0.17] </td>
   <td style="text-align:center;width: 3cm; "> [−0.37, −0.20] </td>
   <td style="text-align:center;width: 3cm; "> [1.18, 1.29] </td>
   <td style="text-align:center;width: 3cm; "> [−0.01, 0.55] </td>
   <td style="text-align:center;width: 3cm; "> [−0.27, −0.05] </td>
   <td style="text-align:center;width: 3cm; "> [1.25, 1.36] </td>
   <td style="text-align:center;width: 3cm; "> [−0.28, 0.22] </td>
   <td style="text-align:center;width: 3cm; "> [−0.48, −0.34] </td>
   <td style="text-align:center;width: 3cm; "> [1.29, 1.41] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Influence 2: Very positive </td>
   <td style="text-align:center;width: 3cm; "> 0.62 </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> 1.94 </td>
   <td style="text-align:center;width: 3cm; "> 1.16 </td>
   <td style="text-align:center;width: 3cm; "> 0.27 </td>
   <td style="text-align:center;width: 3cm; "> 2.24 </td>
   <td style="text-align:center;width: 3cm; "> 1.26 </td>
   <td style="text-align:center;width: 3cm; "> −0.67 </td>
   <td style="text-align:center;width: 3cm; "> 2.31 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [0.22, 1.00] </td>
   <td style="text-align:center;width: 3cm; "> [−0.44, 0.01] </td>
   <td style="text-align:center;width: 3cm; "> [1.81, 2.08] </td>
   <td style="text-align:center;width: 3cm; "> [0.61, 1.67] </td>
   <td style="text-align:center;width: 3cm; "> [−0.01, 0.53] </td>
   <td style="text-align:center;width: 3cm; "> [2.10, 2.38] </td>
   <td style="text-align:center;width: 3cm; "> [0.84, 1.66] </td>
   <td style="text-align:center;width: 3cm; "> [−0.88, −0.47] </td>
   <td style="text-align:center;width: 3cm; "> [2.18, 2.43] </td>
  </tr>
  <tr grouplength="10"><td colspan="10" style="border-bottom: 1px solid;color: white !important;background-color: gray !important;"><strong>Group-Level Variables</strong></td></tr>
<tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Base count </td>
   <td style="text-align:center;width: 3cm; "> 0.10 </td>
   <td style="text-align:center;width: 3cm; "> 0.07 </td>
   <td style="text-align:center;width: 3cm; "> 0.02 </td>
   <td style="text-align:center;width: 3cm; "> 0.07 </td>
   <td style="text-align:center;width: 3cm; "> 0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.08 </td>
   <td style="text-align:center;width: 3cm; "> −0.09 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.05, 0.23] </td>
   <td style="text-align:center;width: 3cm; "> [0.00, 0.14] </td>
   <td style="text-align:center;width: 3cm; "> [−0.04, 0.09] </td>
   <td style="text-align:center;width: 3cm; "> [−0.15, 0.27] </td>
   <td style="text-align:center;width: 3cm; "> [−0.07, 0.11] </td>
   <td style="text-align:center;width: 3cm; "> [−0.14, −0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−0.30, 0.12] </td>
   <td style="text-align:center;width: 3cm; "> [−0.10, 0.05] </td>
   <td style="text-align:center;width: 3cm; "> [−0.09, 0.06] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> GDP </td>
   <td style="text-align:center;width: 3cm; "> 0.83 </td>
   <td style="text-align:center;width: 3cm; "> −0.80 </td>
   <td style="text-align:center;width: 3cm; "> 0.40 </td>
   <td style="text-align:center;width: 3cm; "> −0.49 </td>
   <td style="text-align:center;width: 3cm; "> −0.36 </td>
   <td style="text-align:center;width: 3cm; "> 0.70 </td>
   <td style="text-align:center;width: 3cm; "> 0.60 </td>
   <td style="text-align:center;width: 3cm; "> 0.91 </td>
   <td style="text-align:center;width: 3cm; "> −0.15 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.18, 1.75] </td>
   <td style="text-align:center;width: 3cm; "> [−1.90, 0.25] </td>
   <td style="text-align:center;width: 3cm; "> [−0.46, 1.37] </td>
   <td style="text-align:center;width: 3cm; "> [−1.59, 0.55] </td>
   <td style="text-align:center;width: 3cm; "> [−1.66, 0.68] </td>
   <td style="text-align:center;width: 3cm; "> [−0.14, 1.53] </td>
   <td style="text-align:center;width: 3cm; "> [−0.20, 1.36] </td>
   <td style="text-align:center;width: 3cm; "> [−0.12, 1.91] </td>
   <td style="text-align:center;width: 3cm; "> [−0.99, 0.68] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Population </td>
   <td style="text-align:center;width: 3cm; "> −0.21 </td>
   <td style="text-align:center;width: 3cm; "> 0.64 </td>
   <td style="text-align:center;width: 3cm; "> −0.18 </td>
   <td style="text-align:center;width: 3cm; "> −0.02 </td>
   <td style="text-align:center;width: 3cm; "> −0.04 </td>
   <td style="text-align:center;width: 3cm; "> −0.13 </td>
   <td style="text-align:center;width: 3cm; "> −0.25 </td>
   <td style="text-align:center;width: 3cm; "> −0.63 </td>
   <td style="text-align:center;width: 3cm; "> 0.04 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−0.91, 0.55] </td>
   <td style="text-align:center;width: 3cm; "> [−0.47, 1.64] </td>
   <td style="text-align:center;width: 3cm; "> [−1.04, 0.53] </td>
   <td style="text-align:center;width: 3cm; "> [−0.76, 0.77] </td>
   <td style="text-align:center;width: 3cm; "> [−0.89, 1.20] </td>
   <td style="text-align:center;width: 3cm; "> [−0.95, 0.56] </td>
   <td style="text-align:center;width: 3cm; "> [−0.83, 0.29] </td>
   <td style="text-align:center;width: 3cm; "> [−1.75, 0.50] </td>
   <td style="text-align:center;width: 3cm; "> [−0.66, 0.71] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Troop deployment size </td>
   <td style="text-align:center;width: 3cm; "> −0.68 </td>
   <td style="text-align:center;width: 3cm; "> 0.57 </td>
   <td style="text-align:center;width: 3cm; "> −0.74 </td>
   <td style="text-align:center;width: 3cm; "> 0.53 </td>
   <td style="text-align:center;width: 3cm; "> −0.22 </td>
   <td style="text-align:center;width: 3cm; "> −0.66 </td>
   <td style="text-align:center;width: 3cm; "> −0.30 </td>
   <td style="text-align:center;width: 3cm; "> −0.11 </td>
   <td style="text-align:center;width: 3cm; "> −0.33 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−1.60, 0.38] </td>
   <td style="text-align:center;width: 3cm; "> [−0.66, 2.00] </td>
   <td style="text-align:center;width: 3cm; "> [−1.80, 0.21] </td>
   <td style="text-align:center;width: 3cm; "> [−0.43, 1.55] </td>
   <td style="text-align:center;width: 3cm; "> [−1.36, 0.99] </td>
   <td style="text-align:center;width: 3cm; "> [−1.53, 0.34] </td>
   <td style="text-align:center;width: 3cm; "> [−1.07, 0.58] </td>
   <td style="text-align:center;width: 3cm; "> [−1.38, 1.38] </td>
   <td style="text-align:center;width: 3cm; "> [−1.30, 0.58] </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1"> Intercept </td>
   <td style="text-align:center;width: 3cm; "> −2.21 </td>
   <td style="text-align:center;width: 3cm; "> −0.46 </td>
   <td style="text-align:center;width: 3cm; "> −0.24 </td>
   <td style="text-align:center;width: 3cm; "> −3.24 </td>
   <td style="text-align:center;width: 3cm; "> −0.62 </td>
   <td style="text-align:center;width: 3cm; "> −0.26 </td>
   <td style="text-align:center;width: 3cm; "> −2.73 </td>
   <td style="text-align:center;width: 3cm; "> 0.21 </td>
   <td style="text-align:center;width: 3cm; "> −0.74 </td>
  </tr>
  <tr>
   <td style="text-align:left;padding-left: 2em;width: 5cm; " indentlevel="1">  </td>
   <td style="text-align:center;width: 3cm; "> [−2.62, −1.82] </td>
   <td style="text-align:center;width: 3cm; "> [−0.94, 0.02] </td>
   <td style="text-align:center;width: 3cm; "> [−0.58, 0.12] </td>
   <td style="text-align:center;width: 3cm; "> [−3.74, −2.75] </td>
   <td style="text-align:center;width: 3cm; "> [−1.05, −0.20] </td>
   <td style="text-align:center;width: 3cm; "> [−0.58, 0.06] </td>
   <td style="text-align:center;width: 3cm; "> [−3.17, −2.29] </td>
   <td style="text-align:center;width: 3cm; "> [−0.32, 0.74] </td>
   <td style="text-align:center;width: 3cm; "> [−1.08, −0.40] </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 5cm; "> N </td>
   <td style="text-align:center;width: 3cm; "> 38626 </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; "> 38626 </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; "> 38626 </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 5cm; "> Group </td>
   <td style="text-align:center;width: 3cm; "> Country </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; "> Country </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; "> Country </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 5cm; "> \# Groups </td>
   <td style="text-align:center;width: 3cm; "> 14 </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; "> 14 </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; "> 14 </td>
   <td style="text-align:center;width: 3cm; ">  </td>
   <td style="text-align:center;width: 3cm; ">  </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 5cm; "> sd(Intercept) </td>
   <td style="text-align:center;width: 3cm; "> 0.49 </td>
   <td style="text-align:center;width: 3cm; "> 0.8 </td>
   <td style="text-align:center;width: 3cm; "> 0.54 </td>
   <td style="text-align:center;width: 3cm; "> 0.44 </td>
   <td style="text-align:center;width: 3cm; "> 0.66 </td>
   <td style="text-align:center;width: 3cm; "> 0.51 </td>
   <td style="text-align:center;width: 3cm; "> 0.35 </td>
   <td style="text-align:center;width: 3cm; "> 0.92 </td>
   <td style="text-align:center;width: 3cm; "> 0.51 </td>
  </tr>
</tbody>
</table></div>


# What's left?

There are a few things I'd like to tweak. For example, `{modelsummary}` adds little slashes before the outcome variable name and inserts the name from the model list before that. I leave the list entries blank to avoid this since horizontal space is a premium. 

Blogdown/hugo is also being a bit frustrating with the final table. I've tried making the font larger, but for whatever reason the horizontal scrollbar  isn't working with the kableExtra table in this environment. Not great, but it looks ok for now. Sizing and scale were a littl tough to tweak in Tex files, but I got it more  or less where it needed to be by the end. I'm sure someone with a little more skill than me can figure out some of the hiccups.

There's way more you can do to cusomtize the output of these tables, particularly in the footer. You might want to add particular model fit or summary statistics, and the `glance.custom()` function is a great place to specify what you want. Much is possible, but I'm going to stop here for now since it took me something like a month to get this posted.
