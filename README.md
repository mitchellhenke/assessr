
<!-- README.md is generated from README.Rmd. Please edit that file -->

# assessr

A package to manage, distribute, and version control commonly-used CCAO
assessment functions.

## Installation

You can install the released version of assessR directly from GitLab by
running the following R command after installing `remotes`:

``` r
remotes::install_gitlab(repo = "ccao-data-science---modeling/packages/assessr")
```

Once it is installed, you can use it just like any other package. Simply
call `library(assessr)` at the beginning of your script.

## IAAO Functions

This package contains functions to calculate [IAAO sales ratio study
performance
statistics](https://www.iaao.org/media/standards/Standard_on_Ratio_Studies.pdf).
The settings of these functions are governed by CCAO Data Science
Department SOPs. Do not change them without asking.

### Example Usage

These functions can be used to calculate the COD, PRD, or PRB of a set
of ratios.

``` r
library(assessr)
library(dplyr)

data("ratios_sample")

# Calculate COD
cod_func(ratios_sample$ratios, bootstrap_n = 1000)
#> $COD
#> [1] 12.1144
#> 
#> $COD_SE
#> [1] 0.0124
#> 
#> $COD_95CI
#> [1] "(11.3922, 12.8367)"
#> 
#> $COD_N
#> [1] 881

# Calculate PRB
prb_func(
  ratios_sample$ratios,
  ratios_sample$assessed_values,
  ratios_sample$sales,
  bootstrap_n = 1000
)
#> $PRB
#> [1] 0.0164
#> 
#> $PRB_SE
#> [1] 0.0051
#> 
#> $PRB_95CI
#> [1] "(0.0064, 0.0263)"
#> 
#> $PRB_N
#> [1] 881
```

They can also by applied by group. For example, to get each statistic by
township.

``` r
ratios_sample %>%
  group_by(town) %>%
  summarise(
    COD = cod_func(ratios, bootstrap_n = 1000)$COD,
    PRD = prd_func(ratios, sales, bootstrap_n = 1000)$PRD,
    PRB = prb_func(ratios, sales, assessed_values, bootstrap_n = 1000)$PRB
  )
#> # A tibble: 2 x 4
#>   town        COD   PRD     PRB
#>   <chr>     <dbl> <dbl>   <dbl>
#> 1 Evanston   11.4  1.01 -0.0186
#> 2 New Trier  12.8  1.03 -0.0369
```

You can even use `dplyr` witchcraft to calculate every stat for every
group at the same time.

``` r
ratios_sample %>%
  group_by(town) %>%
  group_modify(~ {
    bind_rows(
      cod_func(.x$ratios, bootstrap_n = 1000),
      prd_func(.x$ratios, .x$sales, bootstrap_n = 1000),
      prb_func(.x$ratios, .x$sales, .x$assessed_values, bootstrap_n = 1000)
    )
  }) %>%
  summarise_each(list(~ first(.x[!is.na(.x)]))) 
#> # A tibble: 2 x 13
#>   town    COD COD_SE COD_95CI COD_N   PRD  PRD_SE PRD_95CI PRD_N     PRB PRB_SE
#>   <chr> <dbl>  <dbl> <chr>    <int> <dbl>   <dbl> <chr>    <int>   <dbl>  <dbl>
#> 1 Evan~  11.3 0.0263 (10.257~   421  1.01 3.00e-4 (1.0003~   421 -0.0188 0.0078
#> 2 New ~  12.8 0.0253 (11.708~   458  1.03 2.00e-4 (1.0189~   458 -0.0374 0.0084
#> # ... with 2 more variables: PRB_95CI <chr>, PRB_N <dbl>
```
