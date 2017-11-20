Logistic Regression
================
Shravan Kuchkula
11/14/2017

-   [Introduction](#introduction)
-   [Problem Statement:](#problem-statement)
-   [Data description:](#data-description)
-   [logit method](#logit-method)

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

Problem Statement:
------------------

A researcher is interested in how variables, such as GRE (Graduate Record Exam scores), GPA (grade point average) and prestige of the undergraduate institution, effect admission into graduate school. The response variable, admit/don’t admit, is a binary variable.

Data description:
-----------------

Read in the dataset.

``` r
mydata <- read.csv("https://stats.idre.ucla.edu/stat/data/binary.csv")
glimpse(mydata)
```

    ## Observations: 400
    ## Variables: 4
    ## $ admit <int> 0, 1, 1, 1, 0, 1, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0,...
    ## $ gre   <int> 380, 660, 800, 640, 520, 760, 560, 400, 540, 700, 800, 4...
    ## $ gpa   <dbl> 3.61, 3.67, 4.00, 3.19, 2.93, 3.00, 2.98, 3.08, 3.39, 3....
    ## $ rank  <int> 3, 3, 1, 4, 4, 2, 1, 2, 3, 2, 4, 1, 1, 2, 1, 3, 4, 3, 2,...

This dataset has a binary response called `admit`. There are three predictor variables: `gre`, `gpa`, `rank`. The first two are continuous variables. Rank takes the value of 1 through 4. Institutions with a rank 1 have the highest prestige, while those with a rank of 4 have the lowest.

Descriptive statistics, sd and contingency table is shown:

``` r
summary(mydata)
```

    ##      admit             gre             gpa             rank      
    ##  Min.   :0.0000   Min.   :220.0   Min.   :2.260   Min.   :1.000  
    ##  1st Qu.:0.0000   1st Qu.:520.0   1st Qu.:3.130   1st Qu.:2.000  
    ##  Median :0.0000   Median :580.0   Median :3.395   Median :2.000  
    ##  Mean   :0.3175   Mean   :587.7   Mean   :3.390   Mean   :2.485  
    ##  3rd Qu.:1.0000   3rd Qu.:660.0   3rd Qu.:3.670   3rd Qu.:3.000  
    ##  Max.   :1.0000   Max.   :800.0   Max.   :4.000   Max.   :4.000

``` r
sapply(mydata, sd)
```

    ##       admit         gre         gpa        rank 
    ##   0.4660867 115.5165364   0.3805668   0.9444602

``` r
# two way contingency table
xtabs(~admit + rank, data = mydata)
```

    ##      rank
    ## admit  1  2  3  4
    ##     0 28 97 93 55
    ##     1 33 54 28 12

logit method
------------

First lets convert the rank variable to a factor.

``` r
mydata$rank <- factor(mydata$rank)
```

Next, let's run the glm function

``` r
mylogit <- glm(admit ~ gre + gpa + rank, data = mydata, family = "binomial")
summary(mylogit)
```

    ## 
    ## Call:
    ## glm(formula = admit ~ gre + gpa + rank, family = "binomial", 
    ##     data = mydata)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -1.6268  -0.8662  -0.6388   1.1490   2.0790  
    ## 
    ## Coefficients:
    ##              Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept) -3.989979   1.139951  -3.500 0.000465 ***
    ## gre          0.002264   0.001094   2.070 0.038465 *  
    ## gpa          0.804038   0.331819   2.423 0.015388 *  
    ## rank2       -0.675443   0.316490  -2.134 0.032829 *  
    ## rank3       -1.340204   0.345306  -3.881 0.000104 ***
    ## rank4       -1.551464   0.417832  -3.713 0.000205 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 499.98  on 399  degrees of freedom
    ## Residual deviance: 458.52  on 394  degrees of freedom
    ## AIC: 470.52
    ## 
    ## Number of Fisher Scoring iterations: 4
