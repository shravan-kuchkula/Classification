ML.Rmd
================
Shravan Kuchkula
11/17/2017

-   [Introduction](#introduction)
-   [RMSE](#rmse)
    -   [Out-of-sample Error.](#out-of-sample-error.)
    -   [Cross validation](#cross-validation)
-   [Classification models: fitting them and evaluating their performance](#classification-models-fitting-them-and-evaluating-their-performance)
-   [Confusion matrix](#confusion-matrix)

Introduction
------------

Let's first load the required libraries.

``` r
source('libraries.R')
```

`caret` is the most widely used packages in R for `Supervised Learning` (aka Predictive modeling). Supervised Learning is machine learning when you have a target variable or something specific that you want to predict. The classic example of Supervised Learning is predicting what species an Iris is based on it's physical measurements. In this case, we have something specific that we want to predict on new data (i.e species).

There are two main kinds of predictive models:

-   **Classification** -&gt; predict Qualitative variables
-   **Regression** -&gt; predict Quantitative variables

Once we have a model, we can use a metric to evaluate how well our model works. For Regression models, we focus on the `RMSE` - our metric of choice, this is the error that our linear regression model seek to minimize. Unfortunately, it is a common practice to calculate the RMSE using the same data that we used to fit the model. This typically leads to over-optimistic model performance, what we call as `over-fitting`. A better approach is to use out-of-sample error to estimate the performance. This is the approach `caret` takes, because it simulates the case that happens in the real world and helps us avoid `over-fitting`. However, it is useful to calculate the in-sample error and then compare this to the out-of-sample error.

RMSE
----

`diamonds` dataset, which is a classic dataset from the ggplot2 package. The dataset contains physical attributes of diamonds as well as the price they sold for. One interesting modeling challenge is predicting diamond price based on their attributes using something like a linear regression.

``` r
# Fit a linear model on the diamonds dataset predicting price using all the variables as predictors.
model <- lm(price ~ . , data = diamonds)
```

Make predictions using model on the full original dataset and save the result to p.

``` r
p <- predict(model)
```

Compute the `errors = predicted - actual`.

``` r
error <- (p - diamonds$price)
```

Calculate the RMSE

``` r
sqrt(mean(error^2))
```

    ## [1] 1129.843

### Out-of-sample Error.

One way you can take a train/test split of a dataset is to order the dataset randomly, then divide it into the two sets. This ensures that the training set and test set are both random samples and that any biases in the ordering of the dataset (e.g. if it had originally been ordered by price or size) are not retained in the samples we take for training and testing your models. You can think of this like shuffling a brand new deck of playing cards before dealing hands.

First, you set a random seed so that your work is reproducible and you get the same random split each time you run your script:c

``` r
# Set seed
set.seed(42)

# Shuffle row indices: rows
rows <- sample(nrow(diamonds))

# Randomly order data
diamonds <- diamonds[rows,]
```

Now that your dataset is randomly ordered, you can split the first 80% of it into a training set, and the last 20% into a test set. You can do this by choosing a split point approximately 80% of the way through your data:

``` r
# Determine row to split on: split
split <- round(nrow(diamonds) * .80)

# Create train
train <- diamonds[1:split,]

# Create test
test <- diamonds[(split+1):nrow(diamonds),]
```

Now that you have a randomly split training set and test set, you can use the lm() function as you did in the first exercise to fit a model to your training set, rather than the entire dataset. Recall that you can use the formula interface to the linear regression function to fit a model with a specified target variable using all other variables in the dataset as predictors. Next, you can use the predict() function to make predictions from that model on new data. The new dataset must have all of the columns from the training data, but they can be in a different order with different values. Here, rather than re-predicting on the training set, you can predict on the test set, which you did not use for training the model. This will allow you to determine the out-of-sample error for the model in the next exercise.

``` r
# Fit lm model on train: model
model <- lm(price ~ . , train)

# Predict on test: p
p <- predict(model, test)
```

Now that you have predictions on the test set, you can use these predictions to calculate an error metric (in this case RMSE) on the test set and see how the model performs out-of-sample, rather than in-sample as you did in the first exercise. You first do this by calculating the errors between the predicted diamond prices and the actual diamond prices by subtracting the predictions from the actual values.

Once you have an error vector, calculating RMSE is as simple as squaring it, taking the mean, then taking the square root

``` r
# Compute errors: error
error <- p - test$price

# Calculate RMSE
sqrt(mean(error ^2))
```

    ## [1] 1136.596

Moral of the story: `Computing the error on the training set is risky because the model may overfit the data used to train it.`

### Cross validation

A better approach to validating models is to use multiple systematic test sets, rather than a single random train/test split. Fortunately, the caret package makes this very easy to do. caret supports many types of cross-validation, and you can specify which type of cross-validation and the number of cross-validation folds with the trainControl() function, which you pass to the trControl argument in train(). It's important to note that you pass the method for modeling to the main train() function and the method for cross-validation to the trainControl() function.

``` r
# Fit lm model using 10-fold CV: model
model <- train(
  price ~ ., diamonds,
  method = "lm",
  trControl = trainControl(
    method = "cv", number = 10,
    verboseIter = TRUE
  )
)
```

    ## + Fold01: intercept=TRUE 
    ## - Fold01: intercept=TRUE 
    ## + Fold02: intercept=TRUE 
    ## - Fold02: intercept=TRUE 
    ## + Fold03: intercept=TRUE 
    ## - Fold03: intercept=TRUE 
    ## + Fold04: intercept=TRUE 
    ## - Fold04: intercept=TRUE 
    ## + Fold05: intercept=TRUE 
    ## - Fold05: intercept=TRUE 
    ## + Fold06: intercept=TRUE 
    ## - Fold06: intercept=TRUE 
    ## + Fold07: intercept=TRUE 
    ## - Fold07: intercept=TRUE 
    ## + Fold08: intercept=TRUE 
    ## - Fold08: intercept=TRUE 
    ## + Fold09: intercept=TRUE 
    ## - Fold09: intercept=TRUE 
    ## + Fold10: intercept=TRUE 
    ## - Fold10: intercept=TRUE 
    ## Aggregating results
    ## Fitting final model on full training set

``` r
# Print model to console
model
```

    ## Linear Regression 
    ## 
    ## 53940 samples
    ##     9 predictor
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (10 fold) 
    ## Summary of sample sizes: 48547, 48546, 48546, 48545, 48545, 48545, ... 
    ## Resampling results:
    ## 
    ##   RMSE      Rsquared   MAE     
    ##   1130.658  0.9197492  740.4646
    ## 
    ## Tuning parameter 'intercept' was held constant at a value of TRUE

Classification models: fitting them and evaluating their performance
--------------------------------------------------------------------

We'll fit classification models with train() and evaluate their out-of-sample performance using cross-validation and area under the curve (AUC).

We'll be working with the Sonar dataset in this chapter, using a 60% training set and a 40% test set. We'll practice making a train/test split one more time, just to be sure you have the hang of it. Recall that you can use the sample() function to get a random permutation of the row indices in a dataset, to use when making train/test splits, and then use those row indices to randomly reorder the dataset. Once your dataset is randomly ordered, you can split off the first 60% as a training set and the last 40% as a test set.

``` r
# Sonar is part of mlbench package.
data(Sonar)

# Shuffle row indices: rows
rows <- sample(nrow(Sonar))

# Randomly order data: Sonar
Sonar <- Sonar[rows,]

# Identify row to split on: split
split <- round(nrow(Sonar) * .6)

# Create train
train <- Sonar[1:split,]

# Create test
test <- Sonar[(split + 1):nrow(Sonar),]
```

> TIP: HOW TO SPLIT A DATAFRAME INTO TRAIN TEST SPLIT IN R ?

Next, we will fit a logistic regression model:

Once you have your random training and test sets you can fit a logistic regression model to your training set using the glm() function. glm() is a more advanced version of lm() that allows for more varied types of regression models, aside from plain vanilla ordinary least squares regression.

Be sure to pass the argument family = "binomial" to glm() to specify that you want to do logistic (rather than linear) regression.

Don't worry about warnings like glm.fit: algorithm did not converge or glm.fit: fitted probabilities numerically 0 or 1 occurred. These are common on smaller datasets and usually don't cause any issues. They typically mean your dataset is perfectly seperable, which can cause problems for the math behind the model, but R's glm() function is almost always robust enough to handle this case with no problems.

Once you have a glm() model fit to your dataset, you can predict the outcome (e.g. rock or mine) on the test set using the predict() function with the argument type = "response".

``` r
# Fit glm model: model
model <- glm(Class ~ ., family = "binomial", data = train)
```

    ## Warning: glm.fit: algorithm did not converge

    ## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred

``` r
# Predict on test: p
p <- predict(model, test, type = "response")
```

Confusion matrix
----------------

A confusion matrix is a very useful tool for calibrating the output of a model and examining all possible outcomes of your predictions (true positive, true negative, false positive, false negative).

Before you make your confusion matrix, you need to "cut" your predicted probabilities at a given threshold to turn probabilities into class predictions. You can do this easily with the ifelse() function

You could make such a contingency table with the table() function in base R, but confusionMatrix() in caret yields a lot of useful ancillary statistics in addition to the base rates in the table. You can calculate the confusion matrix (and the associated statistics) using the predicted outcomes as well as the actual outcomes. `confusionMatrix(predicted, actual)`

``` r
# Calculate class probabilities: p_class
p_class <- ifelse(p > 0.5, "M", "R")

# Create confusion matrix
confusionMatrix(p_class, test$Class)
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction  M  R
    ##          M 11 28
    ##          R 35  9
    ##                                           
    ##                Accuracy : 0.241           
    ##                  95% CI : (0.1538, 0.3473)
    ##     No Information Rate : 0.5542          
    ##     P-Value [Acc > NIR] : 1.0000          
    ##                                           
    ##                   Kappa : -0.5082         
    ##  Mcnemar's Test P-Value : 0.4497          
    ##                                           
    ##             Sensitivity : 0.2391          
    ##             Specificity : 0.2432          
    ##          Pos Pred Value : 0.2821          
    ##          Neg Pred Value : 0.2045          
    ##              Prevalence : 0.5542          
    ##          Detection Rate : 0.1325          
    ##    Detection Prevalence : 0.4699          
    ##       Balanced Accuracy : 0.2412          
    ##                                           
    ##        'Positive' Class : M               
    ##
