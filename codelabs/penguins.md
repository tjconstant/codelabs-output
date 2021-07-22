summary: Classification of penguins using tidymodels
id: classification-of-penguins-using-tidymodels
categories: r-lang
tags: tutorial
status: Published
authors: Tom
Feedback Link: <http://data.tjconstant.com>

# Classification of Penguins using tidymodels

## Overview

Duration: 1

### What you’ll learn

-   how to install `tidymodels` package and the `palmerpenguins` dataset
-   how to plot and investigate the data
-   how to set up a `tidymodels` workflow, recipe and model
-   evaluating and tuning your model

## Installing required packages

Duration: 3

For this codelab, you will need to install the `tidymodels` and
`palmerpenguins` packages.

The `tidymodels` meta-package is actually a collection of packages for
machine learning, that are designed to work together well.

The `palmerpenguins` package contains a dataset of observed penguin
species and their various physical characteristics. The aim of this
codelab is to build a model to predict the penguin;s species based on
these recorded physical characteristics.

To install the required packages, using the following code in the R
console.

``` r
install.packages(c("tidymodels", "palmerpenguins"))
```

## Explore the penguins

Duration: 3

Hello

## Create Test/Train Datasets

``` r
library(tidymodels, quietly = T)
```

    ## Registered S3 method overwritten by 'tune':
    ##   method                   from   
    ##   required_pkgs.model_spec parsnip

    ## ── Attaching packages ────────────────────────────────────── tidymodels 0.1.3 ──

    ## ✓ broom        0.7.8      ✓ recipes      0.1.16
    ## ✓ dials        0.0.9      ✓ rsample      0.1.0 
    ## ✓ dplyr        1.0.7      ✓ tibble       3.1.2 
    ## ✓ ggplot2      3.3.5      ✓ tidyr        1.1.3 
    ## ✓ infer        0.5.4      ✓ tune         0.1.5 
    ## ✓ modeldata    0.1.0      ✓ workflows    0.2.2 
    ## ✓ parsnip      0.1.6      ✓ workflowsets 0.0.2 
    ## ✓ purrr        0.3.4      ✓ yardstick    0.0.8

    ## ── Conflicts ───────────────────────────────────────── tidymodels_conflicts() ──
    ## x purrr::discard() masks scales::discard()
    ## x dplyr::filter()  masks stats::filter()
    ## x dplyr::lag()     masks stats::lag()
    ## x recipes::step()  masks stats::step()
    ## • Use tidymodels_prefer() to resolve common conflicts.

``` r
library(palmerpenguins, quietly = T)
```

``` r
df_split <- initial_split(penguins, strata = species)

print(df_split)
```

    ## <Analysis/Assess/Total>
    ## <258/86/344>

``` r
df_analysis <- df_split %>% analysis()
df_assess <- df_split %>% assessment()
```

## Preprocessing

Duration: 4

``` r
penguins %>% 
  summarise(across(everything(), ~sum(is.na(.x))))
```

    ## # A tibble: 1 x 8
    ##   species island bill_length_mm bill_depth_mm flipper_length_… body_mass_g   sex
    ##     <int>  <int>          <int>         <int>            <int>       <int> <int>
    ## 1       0      0              2             2                2           2    11
    ## # … with 1 more variable: year <int>

``` r
rcp <- 
  recipe(species ~ ., data = df_analysis)
```

``` r
rcp <- 
  recipe(species ~ ., data = df_analysis) %>% 
  step_impute_mean(all_numeric_predictors()) %>% 
  step_impute_mode(all_nominal_predictors())
```

``` r
rcp %>% prep() %>% bake(df_analysis)
```

    ## # A tibble: 258 x 8
    ##    island  bill_length_mm bill_depth_mm flipper_length_… body_mass_g sex    year
    ##    <fct>            <dbl>         <dbl>            <int>       <int> <fct> <int>
    ##  1 Torger…           39.1          18.7              181        3750 male   2007
    ##  2 Torger…           40.3          18                195        3250 fema…  2007
    ##  3 Torger…           44.0          17.2              201        4212 male   2007
    ##  4 Torger…           39.3          20.6              190        3650 male   2007
    ##  5 Torger…           38.9          17.8              181        3625 fema…  2007
    ##  6 Torger…           39.2          19.6              195        4675 male   2007
    ##  7 Torger…           34.1          18.1              193        3475 male   2007
    ##  8 Torger…           42            20.2              190        4250 male   2007
    ##  9 Torger…           37.8          17.1              186        3300 male   2007
    ## 10 Torger…           37.8          17.3              180        3700 male   2007
    ## # … with 248 more rows, and 1 more variable: species <fct>

## Choosing a Model

Duration: 1

``` r
classifier <- 
  decision_tree(mode = "classification") %>% 
  set_engine("rpart")
```

## Create and Fit a Workflow

Duration: 3

``` r
wf <-
  workflow() %>% 
  add_recipe(rcp) %>% 
  add_model(classifier)
```

``` r
model <- fit(wf, data = df_analysis)
```

``` r
model
```

    ## ══ Workflow [trained] ══════════════════════════════════════════════════════════
    ## Preprocessor: Recipe
    ## Model: decision_tree()
    ## 
    ## ── Preprocessor ────────────────────────────────────────────────────────────────
    ## 2 Recipe Steps
    ## 
    ## • step_impute_mean()
    ## • step_impute_mode()
    ## 
    ## ── Model ───────────────────────────────────────────────────────────────────────
    ## n= 258 
    ## 
    ## node), split, n, loss, yval, (yprob)
    ##       * denotes terminal node
    ## 
    ## 1) root 258 144 Adelie (0.441860465 0.197674419 0.360465116)  
    ##   2) flipper_length_mm< 207.5 163  49 Adelie (0.699386503 0.294478528 0.006134969)  
    ##     4) bill_length_mm< 44.65 116   5 Adelie (0.956896552 0.043103448 0.000000000) *
    ##     5) bill_length_mm>=44.65 47   4 Chinstrap (0.063829787 0.914893617 0.021276596) *
    ##   3) flipper_length_mm>=207.5 95   3 Gentoo (0.000000000 0.031578947 0.968421053) *

## Evaluate the Model

Duration: 4

``` r
df_pred <-
  bind_cols(
    df_assess,
    predict(model, new_data = df_assess)
    )
```

``` r
df_pred %>% 
  accuracy(truth = species, estimate = .pred_class)
```

    ## # A tibble: 1 x 3
    ##   .metric  .estimator .estimate
    ##   <chr>    <chr>          <dbl>
    ## 1 accuracy multiclass     0.930

``` r
df_pred %>% 
  conf_mat(truth = species, estimate = .pred_class)
```

    ##            Truth
    ## Prediction  Adelie Chinstrap Gentoo
    ##   Adelie        36         1      1
    ##   Chinstrap      0        15      1
    ##   Gentoo         2         1     29

## Improving the Results

Duration: 5

### Better Preprocessing

``` r
rcp <- 
  recipe(species ~ ., data = df_analysis) %>% 
  step_impute_mean(all_numeric_predictors()) %>% 
  step_impute_mode(all_nominal_predictors()) %>% 
  themis::step_upsample(species)
```

    ## Registered S3 methods overwritten by 'themis':
    ##   method                  from   
    ##   bake.step_downsample    recipes
    ##   bake.step_upsample      recipes
    ##   prep.step_downsample    recipes
    ##   prep.step_upsample      recipes
    ##   tidy.step_downsample    recipes
    ##   tidy.step_upsample      recipes
    ##   tunable.step_downsample recipes
    ##   tunable.step_upsample   recipes

### A Different Model

``` r
classifier <- rand_forest(mode = "classification")
```

``` r
wf <- 
  workflow() %>% 
  add_recipe(rcp) %>% 
  add_model(classifier)

model <- fit(wf, data = df_analysis)
```

    ## Warning: Engine set to `ranger`.

### Assess

``` r
df_pred <-
  bind_cols(
    df_assess,
    predict(model, new_data = df_assess)
    )

df_pred %>% 
  accuracy(truth = species, estimate = .pred_class)
```

    ## # A tibble: 1 x 3
    ##   .metric  .estimator .estimate
    ##   <chr>    <chr>          <dbl>
    ## 1 accuracy multiclass     0.988

``` r
df_pred %>% 
  conf_mat(truth = species, estimate = .pred_class)
```

    ##            Truth
    ## Prediction  Adelie Chinstrap Gentoo
    ##   Adelie        38         0      1
    ##   Chinstrap      0        17      0
    ##   Gentoo         0         0     30
