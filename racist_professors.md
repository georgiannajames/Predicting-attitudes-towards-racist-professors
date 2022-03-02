Predicting Attitudes Towards Racist Professors
================
Georgianna James
2022-03-02

## Required Packages

``` r
library(rcfss)
library(tidyverse)
library(tidymodels)
library(rsample)
library(knitr)
library(ranger)
library(kknn)
library(glmnet)

set.seed(100)

theme_set(theme_minimal())
```

## Load the data

``` r
data("gss", package = "rcfss")

# create a data set that omits columns that are not predictors

gss_pred <- gss %>%
  select(-id, -wtss)
```

# Creating training and testing sets

``` r
# split data into train and test

gss_split <- initial_split(gss_pred, prop = 3 / 4)
gss_train <- training(gss_split)
gss_test <- testing(gss_split)
```

# Predictions using a logistic regression model

``` r
# define logistic regression model

log_mod <- logistic_reg() %>%
  set_engine("glm") %>%
  set_mode("classification")
```

``` r
# cross validate with 10 folds


# creating folds

gss_fold <- vfold_cv(data = gss_train, v = 10, strata = colrac)
```

``` r
# refitting model to samples and collecting metrics

gss_fold_pred <- log_mod %>%
  fit_resamples(colrac ~ age + black + degree + partyid_3 + sex + south, resamples = gss_fold) %>%
  collect_metrics()
```

## Logistic Regression Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.5276091 |  10 |      0.0123496 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.5278813 |  10 |      0.0138530 | Preprocessor1_Model1 |

# Predictions using a random forest model

``` r
# define gss recipe

gss_rec <- recipe(colrac ~ ., data = gss_pred) %>%
  step_impute_median(all_numeric_predictors()) %>%
  step_impute_mode(all_nominal_predictors()) %>%
  step_naomit(all_outcomes(), skip = TRUE)
```

``` r
# define random forest model

rf_mod <- rand_forest(
  mode = "classification",
  engine = "ranger",
  mtry = NULL,
  trees = NULL,
  min_n = NULL
)
```

``` r
# define workflow

rf_wf <- workflow() %>%
  add_recipe(gss_rec) %>%
  add_model(rf_mod)
```

``` r
# cross validate

gss_fold <- vfold_cv(data = gss_train, v = 10, strata = colrac)
```

``` r
# fit the workflow

rf_wf_results <- rf_wf %>%
  fit_resamples(resamples = gss_fold) %>%
  collect_metrics()
```

## Random Forrest Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.8076406 |  10 |      0.0161311 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.8765782 |  10 |      0.0116015 | Preprocessor1_Model1 |

# Predictions using k-nearest neighbors mdoel

``` r
# define the model

knn_mod <- nearest_neighbor() %>%
  set_engine("kknn") %>%
  set_mode("classification")
```

``` r
# define the recipe

knn_rec <- recipe(colrac ~ ., data = gss_pred) %>%
  step_impute_median(all_numeric_predictors()) %>%
  step_impute_mode(all_nominal_predictors()) %>%
  step_naomit(all_outcomes(), skip = TRUE) %>%
  step_novel(all_nominal_predictors()) %>%
  step_dummy(all_nominal_predictors()) %>%
  step_zv(all_predictors()) %>%
  step_normalize(all_numeric())
```

``` r
# define the workflow

knn_wf <- workflow() %>%
  add_recipe(knn_rec) %>%
  add_model(knn_mod)
```

``` r
# cross validate

gss_fold <- vfold_cv(data = gss_train, v = 10, strata = colrac)
```

``` r
# fit the workflow

knn_wf_results <- knn_wf %>%
  fit_resamples(resamples = gss_fold) %>%
  collect_metrics()
```

## K-Nearest Neighbors Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.5931896 |  10 |      0.0183927 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.6735220 |  10 |      0.0138682 | Preprocessor1_Model1 |

# Predictions using a Ridge Logistic Regression Model

``` r
# define model

ridge_log_mod <- logistic_reg(
  penalty = .01,
  mixture = 0,
  mode = "classification",
  engine = "glmnet"
)
```

``` r
# define recipe

ridge_rec <- recipe(colrac ~ ., data = gss_pred) %>%
  step_impute_median(all_numeric_predictors()) %>%
  step_impute_mode(all_nominal_predictors()) %>%
  step_naomit(all_outcomes(), skip = TRUE) %>%
  step_novel(all_nominal_predictors()) %>%
  step_dummy(all_nominal_predictors()) %>%
  step_zv(all_predictors()) %>%
  step_normalize(all_numeric())
```

``` r
# define the workflow

ridge_wf <- workflow() %>%
  add_recipe(ridge_rec) %>%
  add_model(ridge_log_mod)
```

``` r
# cross validate

gss_fold <- vfold_cv(data = gss_train, v = 10, strata = colrac)
```

``` r
# fit the workflow

ridge_wf_results <- ridge_wf %>%
  fit_resamples(resamples = gss_fold) %>%
  collect_metrics()
```

## Ridge Logistic Regression Model Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.7828173 |  10 |      0.0077069 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.8560039 |  10 |      0.0078405 | Preprocessor1_Model1 |

# Comparing all results

## Logistic Regression Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.5276091 |  10 |      0.0123496 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.5278813 |  10 |      0.0138530 | Preprocessor1_Model1 |

## Random Forrest Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.8076406 |  10 |      0.0161311 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.8765782 |  10 |      0.0116015 | Preprocessor1_Model1 |

## 5-Nearest Neighbors Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.5931896 |  10 |      0.0183927 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.6735220 |  10 |      0.0138682 | Preprocessor1_Model1 |

## Ridge Logistic Regression Model Metrics Table

| Metric   | Estimator |      Mean |   n | Standard Error | Config               |
|:---------|:----------|----------:|----:|---------------:|:---------------------|
| accuracy | binary    | 0.7828173 |  10 |      0.0077069 | Preprocessor1_Model1 |
| roc_auc  | binary    | 0.8560039 |  10 |      0.0078405 | Preprocessor1_Model1 |

The random forest model had the most accurate predictions, so I will
train this recipe/model using the full training set and report the
accuracy using the held-out test set of data.

``` r
# training the rf recipe using the full training set and predicting test set outcomes

rf_test_results <- rf_wf %>%
  fit(data = gss_train) %>%
  predict(new_data = gss_test) %>%
  mutate(true_gss = gss_test$colrac) %>%
  accuracy(truth = true_gss, estimate = .pred_class)
```

## Final Random Forest Model Prediction Results

| Metric   | Estimator |  Estimate |
|:---------|:----------|----------:|
| accuracy | binary    | 0.8213166 |
