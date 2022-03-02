# Predicting-attitudes-towards-racist-professors


Author: Georgianna James

This repo was originally created to complete a homework assignment for the course [MACS 30500](https://cfss.uchicago.edu). See detailed instructions for this homework assignment [here](https://cfss.uchicago.edu/homework/machine-learning/#fn:View-the-documen).

## Required packages



```r
library(rcfss)
library(tidyverse)
library(tidymodels)
library(rsample)
library(knitr)
library(ranger)
library(kknn)
library(glmnet)

```

[`rcfss`](https://github.com/uc-cfss/rcfss) can be installed from GitHub using the command:

```r
if (packageVersion("devtools") < 1.6) {
  install.packages("devtools")
}

devtools::install_github("uc-cfss/rcfss")
```

##  Summary

### Predicting attitudes towards racist college professors 

In these repo, I create several machine learning models to predict attitudes towards racist college prfoessors. The data set ```rcfss::gss``` contains the outcome variable of interest, ```colrac```, which is a factor variable indicating professor's answers to the question “Should a person who believes that Blacks are genetically inferior be allowed to teach in a college or university?” This variable is a factor variable coded as either ```"ALLOWED"``` or ```"NOT ALLOWED"``` according to respondent's answer. 


* [R Markdown file](./racist_professors.Rmd)
* [Markdownn file](./racist_professors.md)

# Data

This data was originally sourced from the [General Social Survey](https://gss.norc.org). The data was condensed, selecting for a few variables from the year 2012, and included in the package ```rcfss``` in a data set named ```gss```. 