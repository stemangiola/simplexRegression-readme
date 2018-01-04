
# simplexRegression BETA

This is an R package that infers trend of change in proportions with the condition that they have to sum to 1. This package (differently from other packages) is able to distinguish between independent changes (names causal) and changes that depend on the sum-to-one normalization (names effectual).

## Example: focus on causality

There are 4 species of birds.

The totality of the birds at time 0 is:

- black brids: 350
- red birds: 350
- blue birds: 200
- green birds: 100

At time 1, the black bird population has increased a lot while all others did not change at all:

- black brids: 2850
- red birds: 350
- blue birds: 200
- green birds: 100

Because under some conditions we cannot know the absolute number of total birds, we necessarily have to think in terms of proportions. For example we would get this picture.

<img src="https://github.com/stemangiola/simplexRegression-readme/blob/master/man/figures/trends.png" width=800 alt="Plot"/>

We can have the (statistical) impression see that all four species populations changed in size, and that all changes can be significant; while the reality is that three population are not decreasing independently, but just as result of the sum-to-one normalization.

The following probabilistic model tries to address this challenge. Understanding what is an independent change and what is not.

### R code example

```r
if (!require(devtools)) {
  install.packages("devtools")
  library(devtools)
}
if (!require(Rcpp)) {
  install.packages("Rcpp")
  library(Rcpp)
}
options(buildtools.check = function(action) TRUE)
install_github("stemangiola/simplexRegression", args = "--preclean", build_vignettes = FALSE, auth_token = "1350e5ec4a849d6c6e975df1dafa219a5ada0dd4", password="x-oauth-basic")
set.seed(1234)
library("simplexRegression")

n_samples = 300
my_design = model.matrix(~cov , data = data.frame(cov=runif(n_samples)))
obj = simulate_proportions(
	30 ,                        ## noise of the data
	c(0.35, 0.35, 0.2, 0.1 ) ,  ## proportions at the intercept
	c(2, 0, 0, 0),              ## rate of change (positive just for population 1)
	my_design,                  ## design matrix
	n_samples = n_samples       ## number of samples
)

fit = simplexRegression(obj$prop, my_design, cov_to_test="cov")
```

<img src="https://github.com/stemangiola/simplexRegression-readme/blob/master/man/figures/coef_ang.png" width=800 alt="Plot"/>


```r
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 1:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.36		0		-28.18		0 ***
cov - causal		1.9		0		-24.37		0 ***   <--- correctly indentified 
cov - effectual		1.58		0		-21.41		0 ***
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 2:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.34		0		-23.95		0 ***
cov - causal		0.1		0		-1.16		0.69 
cov - effectual		-1.21		0		14.7		0 ***
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 3:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.21		0		-7.11		1.12e-06 ***
cov - causal		-0.19		0		1.72		0.52 
cov - effectual		-1.17		0		11.72		0 ***
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 4:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.09		0		7.63		0 ***
cov - causal		0.07		0		-0.38		0.8 
cov - effectual		-0.67		0		5.6		2.18e-08 ***
------------------------------------------------------------------------------

Significance codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
Number of Observations: 300
Link: linear-softmax
```

This model has successfully identified which group had trends in proportions that were independend (variable no. 1, labelled as causal) and which were dependent from the softmax (variable no. 2, 3, 4).

### Difference from the package DirichletReg

This is different from the package DirichletReg, where all changes of the four groups are seen to be significant.

```r
if (!require(DirichletReg)) {
	install.packages("DirichletReg")
	library(DirichletReg)
}

dd = data.frame(obj$prop, cov = my_design[,2])
AL <- DR_data(dd[, 1:4])
summary(DirichReg(AL ~ cov, dd))
```


```r
------------------------------------------------------------------
Beta-Coefficients for variable no. 1: X1
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  2.50592    0.09509  26.353  < 2e-16 ***
cov          0.88813    0.16188   5.486  4.1e-08 ***
------------------------------------------------------------------
Beta-Coefficients for variable no. 2: X2
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  2.50290    0.09643  25.954  < 2e-16 ***
cov         -0.93510    0.16617  -5.627 1.83e-08 ***
------------------------------------------------------------------
Beta-Coefficients for variable no. 3: X3
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  1.86560    0.09455  19.730  < 2e-16 ***
cov         -0.84977    0.16008  -5.308 1.11e-07 ***
------------------------------------------------------------------
Beta-Coefficients for variable no. 4: X4
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  1.17471    0.09963  11.791  < 2e-16 ***
cov         -0.59494    0.17108  -3.477 0.000506 ***
------------------------------------------------------------------
Significance codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Log-likelihood: 1425 on 8 df (98 BFGS + 1 NR Iterations)
AIC: -2835, BIC: -2805
Number of Observations: 300
Link: Log
Parametrization: common
```

## Example: focus on effectuality

Now, let's make an example where the blue population atively keeps the same range of proportion across time

The totality of the birds at time 0 is:

- black brids: 350
- red birds: 350
- blue birds: 200
- green birds: 100

At time 1, the black and blue bird populations have increased a lot while all others did not change at all:

- black brids: 2850
- red birds: 350
- blue birds: 700
- green birds: 100

In this scenario, the blue population effectively preserves its proportion range (effectual), increasing its size (causal). 

<img src="https://github.com/stemangiola/simplexRegression-readme/blob/master/man/figures/trends2.png" width=800 alt="Plot"/>

If the interest of the analysis is around the effective proportions of componets, rather than the growing forces (e.g., a minimum number of blue birds is needed to hold the ecosystem), this effectual trend is something has to be tested as well.

```r
if (!require(devtools)) {
  install.packages("devtools")
  library(devtools)
}
if (!require(Rcpp)) {
  install.packages("Rcpp")
  library(Rcpp)
}
options(buildtools.check = function(action) TRUE)
install_github("stemangiola/simplexRegression", args = "--preclean", build_vignettes = FALSE, auth_token = "1350e5ec4a849d6c6e975df1dafa219a5ada0dd4", password="x-oauth-basic")
set.seed(1234)
library("simplexRegression")

n_samples = 300
my_design = model.matrix(~cov , data = data.frame(cov=runif(n_samples)))

obj = simulate_proportions(
    30 ,
    c(0.35, 0.35, 0.2, 0.1 ) ,
    c(4, 0, 3.25, 0),
    my_design,
    n_samples = n_samples
)

fit = simplexRegression(obj$prop, my_design, cov_to_test="cov")
```

<img src="https://github.com/stemangiola/simplexRegression-readme/blob/master/man/figures/coef_ang2.png" width=800 alt="Plot"/>


```r
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 1:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.36		0		-43.6		0 ***
cov - causal		3.77		0		-39.65		0 ***
cov - effectual		1.5		0		-20.37		0 ***
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 2:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.35		0		-33.91		0 ***
cov - causal		-0.07		0		0.49		0.9 
cov - effectual		-2.72		0		24.75		0 ***
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 3:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.2		0		-22.96		9.02e-14 ***
cov - causal		3.18		0		-26.25		0 ***  <--- actual increase
cov - effectual		0.07		0		-0.89		0.37   <--- effectual stationarity 
------------------------------------------------------------------------------
Beta-Coefficients for variable no. 4:
			Estimate	Std. Error	z value		Pr(>|z|)
(Intercept)		0.1		0		-7.56		0 ***
cov - causal		0.07		0		-0.17		0.9 
cov - effectual		-1.25		0		8.74		0 ***
------------------------------------------------------------------------------

Significance codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
Number of Observations: 300
Link: linear-softmax
```

This type of relation is the (only) one that other algorithms such as DirichReg actually test, as can be seen in the following example.

```r
if (!require(DirichletReg)) {
	install.packages("DirichletReg")
	library(DirichletReg)
}

dd = data.frame(obj$prop, cov = my_design[,2])
AL <- DR_data(dd[, 1:4])
summary(DirichReg(AL ~ cov, dd))
```
```r
------------------------------------------------------------------
Beta-Coefficients for variable no. 1: X1
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  2.66137    0.09649  27.581  < 2e-16 ***
cov          0.69515    0.16991   4.091 4.29e-05 ***
------------------------------------------------------------------
Beta-Coefficients for variable no. 2: X2
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)   2.5234     0.1014   24.89   <2e-16 ***
cov          -2.3152     0.1852  -12.50   <2e-16 ***
------------------------------------------------------------------
Beta-Coefficients for variable no. 3: X3
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  1.91932    0.09954  19.281   <2e-16 ***
cov          0.13153    0.17607   0.747    0.455    
------------------------------------------------------------------
Beta-Coefficients for variable no. 4: X4
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)   1.1368     0.1093  10.402  < 2e-16 ***
cov          -1.1595     0.2008  -5.773 7.79e-09 ***
------------------------------------------------------------------
Significance codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Log-likelihood: 1501 on 8 df (119 BFGS + 2 NR Iterations)
AIC: -2987, BIC: -2957
Number of Observations: 300
Link: Log
Parametrization: common
```
