# mivcausal: A Stata module for testing the hypothesis about the signs of the 2SLS weights

*[Conroy Lau](https://github.com/conroylau/) and [Alexander Torgovitsky](https://a-torgovitsky.github.io/)*

__Table of Contents__  

-  [Introduction](introduction)
-  [Installation](installation)
-  [Syntax](syntax)
-  [Stored results](stored-results)
-  [Examples](examples)
-  [Help, Feature Requests and Bug Reports](#help-feature-requests-and-bug-reports)
-  [References](references)

## Introduction
`mivcausal` is a command that tests the hypothesis about the signs of the two
stage least squares (2SLS) weights. As described in proposition 5 of
[Mogstad, Torgovitsky and Walters (2020)](https://www.nber.org/papers/w25691),
the 2SLS estimand can be written as a positively-weighted average of LATEs under
certain conditions. This module tests whether the weights are positive when
there are two binary instruments and one binary treatment using the following tests:

  1. Bonferroni test
  2. Cox and Shi (2019) test (henceforth, CS test)
  3. Mintest
  4. Romano, Shaikh and Wolf (2014) test (henceforth, RSW test)
  5. Intersection-union test (henceforth, IUT)

Please see appendix C of
[Mogstad, Torgovitsky and Walters (2020)](https://www.nber.org/papers/w25691)
for more details.

## Installation

The `mivcausal` module can be installed in Stata by typing one of the
following commands in Stata:

1. Install `mivcausal` from GitHub:

``` Stata
net install mivcausal, from("https://raw.githubusercontent.com/conroylau/mivcausal/master")
```

2. Install `mivcausal` from SSC:

``` Stata
ssc install mivcausal
```

## Syntax
The syntax of the `mivcausal` command is as follows:

```Stata
mivcausal [(instd = instr1 instr2) indepvars] [if] [in] [, seed(#) precision(#) mtdraws(#) rsw(reps = #) vce(vcetype)]
```

The `[(instd = instr1 instr2) indepvars]` part is optional if it is used
as a post-estimation command after `ivregress 2sls`. If not, this part has
to be specified, where

* `instd` is the binary treatment.
* `instr1` and `instr2` are the two binary instruments.
* `indepvars` are the independent variables.

If the binary treatment or instruments do not equal to either 0 or 1,
the module will run the test by creating a temporary variable that assigns
the larger value as 1 and the smaller value as 0.

In addition, the `mivcausal` command has the following options:

* `seed` specifies the seed used to simulate the data in the Mintest and
the starting seed used in the bootstrap procedure of the RSW test.
The default is `seed(1)`.

* `precision` specifies the number of decimal places in the *p*-value.
This has to be a positive integer. The default is `precision(3)`.

* `mtdraws` specifies the number of draws in the simulations for the Mintest.
This has to be a positive integer. The default is `mtdraws(10000)`.

* `rsw` specifies whether the RSW test is applied and the number of bootstrap
replications in the RSW test. By default, the RSW test is not applied because
of longer computational time. To apply the RSW test, specify the option
`rsw(reps = x)` where `x` is the number of bootstrap replications. This has to
be a nonnegative integer. Specifying the option as `rsw()` (i.e. without
specifying the number of replications) will run the RSW test with 1,000
bootstrap replications.

* `vce(vcetype)` specifies the type of standard error reported. Currently, the
`mivcausal` command supports robust (`robust`) or clustered standard
errors (`cluster(varlist)`).

## Stored results
After applying the `mivcausal` command, the following are stored in `r()`:

**Scalars**  

| Scalar | Description |
| :----- | :----- |
| `r(pval_bon)` | *p*-value of the Bonferroni test |
| `r(pval_cs)` | *p*-value of the CS test |
| `r(pval_mint)` | *p*-value of the Mintest |
| `r(pval_rsw)` | *p*-value of the RSW test (if applicable) |
| `r(pval_iut)` | *p*-value of the IUT |
| `r(seed)` | Seed used |
| `r(precision)` | Number of decimal places in the *p*-value |
| `r(mtdraws)` | Number of simulations in the Mintest |
| `r(rswreps)` | Number of bootstrap replications in the RSW test (if applicable) |

**Macros**  

| Macro | Description |
| :----- | :----- |
| `r(cmd)` | Command - `mivcausal` |
| `r(title)` | Title of the test - Testing hypotheses about the signs of the 2SLS weights |
| `r(depvar)` | Name of dependent variable (if applicable) |
| `r(instd)` | Instrumented variable(s) |
| `r(exogr)` | Exogenous regressor(s) (if applicable) |
| `r(insts)` | Instruments |
| `r(vce)` | `vcetype` specified in `vce()` |


## Examples

Consider the sample data with one covariate (`X`), one binary treatment (`D`),
two binary instruments (`Z1` and `Z2`) and one outcome variable (`Y`).

First, open the data set:
``` Stata
use mivcausal.dta
```
Then, the tests can be run as follows:
``` Stata
mivcausal (D = Z1 Z2) X, rsw(reps = 1000)
```
This gives:
``` Stata
Results from mivcausal:
------------------------------------------------------------------------------
  Null hypothesis         |  Test                             |  p-value
--------------------------+-----------------------------------+---------------
  All weights positive    |  Bonferroni                       |  .038
                          |  Cox and Shi (2019)               |  .038
                          |  Mintest                          |  .037
                          |  Romano, Shaikh and Wolf (2014)   |  .025
--------------------------+-----------------------------------+---------------
  Some weights negative   |  Intersection-union test          |  .981
------------------------------------------------------------------------------
Parameters:
* Precision (decimal places) = 3
* Number of draws in Mintest = 10000
* Number of bootstraps in the Romano, Shaikh and Wolf (2014) test = 1000
```

As described above, the RSW test will be skipped if the following command
is run instead.
``` Stata
mivcausal (D = Z1 Z2) X
```
This gives:
``` Stata
Results from mivcausal:
------------------------------------------------------------------------------
  Null hypothesis         |  Test                             |  p-value
--------------------------+-----------------------------------+---------------
  All weights positive    |  Bonferroni                       |  .038
                          |  Cox and Shi (2019)               |  .038
                          |  Mintest                          |  .037
--------------------------+-----------------------------------+---------------
  Some weights negative   |  Intersection-union test          |  .981
------------------------------------------------------------------------------
Parameters:
* Precision (decimal places) = 3
* Number of draws in Mintest = 10000
```

Alternatively, the `mivcausal` command can be applied as a post-estimation
command after `ivregress 2sls`:
``` Stata
ivregress 2sls Y (D = Z1 Z2) X
```

Then, the tests can be applied without typing the treatment, instruments
and covariates again:
``` Stata
mivcausal, rsw(reps = 1000)
```
This gives the same output as before, i.e.
``` Stata
Results from mivcausal:
------------------------------------------------------------------------------
  Null hypothesis         |  Test                             |  p-value
--------------------------+-----------------------------------+---------------
  All weights positive    |  Bonferroni                       |  .038
                          |  Cox and Shi (2019)               |  .038
                          |  Mintest                          |  .037
                          |  Romano, Shaikh and Wolf (2014)   |  .025
--------------------------+-----------------------------------+---------------
  Some weights negative   |  Intersection-union test          |  .981
------------------------------------------------------------------------------
Parameters:
* Precision (decimal places) = 3
* Number of draws in Mintest = 10000
* Number of bootstraps in the Romano, Shaikh and Wolf (2014) test = 1000
```

## Help, Feature Requests and Bug Reports

Please post an issue on the [GitHub
repository](https://github.com/conroylau/mivcausal/issues). We are happy
to help.

## References

Cox, G. and X. Shi. (2019): "A Simple Uniformly Valid Test for Inequalities," 
*Working paper*.

Mogstad, M., A. Torgovitsky, and C. R. Walters: (2020): "The Causal
Interpretation of Two-Stage Least Squares with Multiple Instrumental Variables,"
*Working Paper*.

Romano, J. P., A. M. Shaikh, and M. Wolf (2014): "A Practical Two-step Method for Testing
Moment Inequalities," *Econometrica*, 82(5), 1979-2002.
