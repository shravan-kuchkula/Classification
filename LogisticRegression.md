Logistic Regression
================
Shravan Kuchkula
11/14/2017

-   [Introduction](#introduction)

Introduction
------------

Logistic regression, also called a logit model, is used to model dichotomous outcome variables. In the logit model the log odds of the outcome is modeled as a linear combination of the predictor variables.

``` r
# Load all the libraries
installRequiredPackages <- function(pkg){
  new.pkg <- pkg[!(pkg %in% installed.packages()[,"Package"])]
  if (length(new.pkg))
    install.packages(new.pkg, dependencies = TRUE)
  sapply(pkg, require, character.only = TRUE)
}

libs <- c("dplyr", "tidyr", "ggplot2",
          "magrittr", "markdown", "knitr", "yaml",
           "broom", "aod")

installRequiredPackages(libs)
```

    ##    dplyr    tidyr  ggplot2 magrittr markdown    knitr     yaml    broom 
    ##     TRUE     TRUE     TRUE     TRUE     TRUE     TRUE     TRUE     TRUE 
    ##      aod 
    ##     TRUE
