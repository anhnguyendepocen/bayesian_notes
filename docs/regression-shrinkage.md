
---
output: html_document
editor_options: 
  chunk_output_type: console
---
# Shrinkage and Regularized Regression

## Prerequisites {-}


```r
library("rstan")
library("rstanarm")
library("bayz")
library("tidyverse")
library("broom")
library("glmnet")
library("recipes")
```


## Introduction

*Shrinkage estimation* deliberately introduces biases into the model to improve
*overall performance, often at the cost of individual estimates
*[@EfronHastie2016a, p. 91].

This is opposed to MLE, which produces unbiased estimates (asymptotically,
given certain regularity conditions). Likewise, the Bayesian estimates with
non- or weakly-informative priors will produce estimates similar to the MLE.
With shrinkage, the priors are used to produce estimates *different* than the
MLE case.

*Regularization* describes any method that reduces variability in high
dimensional estimation or prediction problems [@EfronHastie2016a].

## Penalized Maximum Likelihood Regression

OLS finds the $\beta$ that minimize the in-sample sum of squared errors,
$$
\hat{\beta}_{\text{OLS}} = \arg\min_{\beta} \sum_{i = 1}^n (\vec{x}_i\T \vec{\beta} - y_i)^2
$$

Penalized regressions add a penalty term increasing in the magnitude of $\beta$ to the minimization function.
$$
\hat{\beta}_{\text{penalized}} = \argmin_{\beta} \sum_{i = 1}^n (\vec{x}_i\T \vec{\beta} - y_i)^2 + \underbrace{f(\beta)}_{\text{shrinkage penalty}},
$$
where $f$ is some sort of penalty function on $\beta$ that penalizes larger (in magnitude) values of $\beta$.

Penalized regression purposefully introduces bias into the regression in order 
to reduce variance and improve out-of-sample prediction.  The penalty term, 
when chosen by cross-validation or an approximation thereof, allows for trading
off bias and variance.

Different penalized regression methods use different choices of $f(\beta)$, 
The two most commonly penalty functions are Ridge and Lasso.



### Ridge Regression

Ridge regression uses the following penalty [@HoerlKennard1970a]:
$$
\hat{\beta}_{\text{ridge}} = \arg\min_{\beta} \underbrace{\sum_{i = 1}^n (\vec{x}_i\T \vec{\beta} - y_i)^2}_{\text{RSS}} + \underbrace{\lambda}_{\text{tuning parameter}} \underbrace{\sum_{k} \beta_k^2}_{\ell_2 \text{ norm}^2}
$$
The $\ell_2$ norm of $\beta$ is,
$$
||\beta||_{2} = \sqrt{\sum_{k = 1}^K \beta_k^2} .
$$

The ridge regression coefficients are smaller in magnitude than the OLS coefficients, $|\hat{\beta}_{ridge}| < |\hat{\beta}_{OLS}|$.
However, this bias in the coefficients can be offset by a lower variance, better MSE, and better out-of-sample performance than the OLS estimates.

Unlike many other penalized regression estimators, ridge regression has a 
close-form solution.
The expected value and variance-covariance matrix of the ridge regression 
coefficients is,
$$
\begin{aligned}[t]
\E[\hat{\beta}_{\text{ridge}}] &= M y \\
\Var[\hat{\beta}_{\text{ridge}}] &=  \sigma^2 M' M \\
M &= (X' X + \lambda I)^{-1} X' .
\end{aligned}
$$

Some implications:

-   $\hat{\vec{\beta}}$ exists even if $\hat{\vec{\beta}}_{\text{OLS}}$
    ($(\mat{X}\T\mat{X})^{-1}$), i.e. cases of $n > p$ and collinearity, does
    not exist.

-   If $\mat{X}$ is orthogonal (mean 0, unit variance, zero correlation),
    $\mat{X}\T \mat{X} = n \mat{I}_p$ then
    $$
    \hat{\vec{\beta}}_{\text{ridge}} = \frac{n}{n + \lambda}
    \hat{\vec{\beta}}_{\text{ols}}
    $$
    meaning,
    $$
    |\hat{\vec{\beta}}_{\text{ols}}| >
    |\hat{\vec{\beta}}_{\text{ridge}}| \geq 0
    $$

-   Ridge does not produce sparse estimates, since
    $(n / (n + \lambda)) \vec{\vec{\beta}}_{ols} = 0$ iff $\vec{\vec{\beta}}_{ols} = 0$

-   If $\lambda = 0$, then the ridge coefficients are the same as the OLS 
    coefficients, 
    $\lambda \to 0 \Rightarrow \hat{beta}_{\text{ridge}} \to \hat{beta}_{OLS}$ 

-   As $\lambda$ increases the coefficients are shrunk to 0, 
    $\lambda \to \infty \Rightarrow \hat{\beta}_{\text{ridge}} = 0$.

### Lasso

The lasso (Least Absolute Shrinkage and Selection Operator) uses an $\ell_1$ norm of $\beta$ as a penalty [@Tibshirani1996a],
$$
\hat{\beta}_{\text{lasso}} = \arg\min_{\beta} \frac{1}{2 \sigma} \sum_{i = 1}^n (\vec{x}_i\T \vec{\beta} - y_i)^2 + \lambda \sum_{k} |\beta_k|
$$
where $\lambda \geq 0$  is a tuning or shrinkage parameter chosen by cross-validation or a plug-in statistic.

The $\ell_1$ norm of $\beta$ is the sum of the absolute values of its elements, 
$$
||\beta||_{1} = \sum_{k = 1}^K |\beta_k| .
$$

Properties:

-   Unlike ridge regression, it sets some coefficients exactly to 0, producing
    sparse solutions.

-   If variables are perfectly correlated, there is no unique solution
    (unlike the ridge regression).

-   Used as the best convex approximation of the "best subset selection"
    regression problem, which finds the number of nonzero entries in a vector.

-   Unlike ridge regression, there is no closed-form solution.
    Since $|\beta_k|$ does not have a derivative, it was a more difficult 
    iterative problem than many other regression functions. However, now there
    are several algorithms to estimate it.
    
### Constrained Optimization Interpretation

### Bayesian Interpretation

The penalty term in regressions can generally be interpreted a prior on the 
coefficients.
Recall that although OLS does not require normal errors, the OLS coefficients are equivalent to the MLE of a probability model with normal errors,
$$
\begin{aligned}
\hat{\beta}_{MLE} &= \arg \max_{\beta} \dnorm(y | x \beta, \sigma) \\
& = \arg \max_{\beta} {(2 \pi \sigma^2)}^{n / 2} \prod_{i = 1}^{n} \exp\left(-\frac{(y_i - x_i' \beta)^2}{2 \sigma^2}\right) \\
&= \arg \max_{\beta} \frac{n}{2} (\log 2 + \log \pi) + n \log \sigma + \sum_{i = 1}^{n} \left( -\frac{(y_i - x_i' \beta)^2}{2 \sigma^2} \right) \\
& = \arg \max_{\beta} \sum_{i = 1}^{n} - (y_i - x'_i \beta)^2 \\
&= \arg \min_{\beta} \sum_{i = 1} (y_i - x'_i \beta)^2 \\
&= \hat{\beta}_{OLS}
\end{aligned}
$$
Likewise the shrinkage prior can be represented as a normal distribution with mean 0 and scale $1 / \lambda$, since the $\beta$ that maximize the probability of that, minimize the $\ell_2$ norm of $\beta$,
$$
\begin{aligned}
\arg \max_{\beta} \dnorm(\beta | 0,  \tau) &= \arg \max_{\beta} {(2 \pi \sigma^2)}^{K / 2} \prod_{k = 1}^{K} \exp\left(- \frac{(0 - \beta_k)^2}{2 \tau^2} \right)
\\
&= \arg \max_{\beta} \sum_{k = 1}^{K} \left(-\frac{\beta_k^2}{2 \tau^2}\right) \\
&= \arg \min_{\beta} \frac{1}{2 \tau^2} \sum_{k = 1}^K \beta_k^2
\end{aligned}
$$
where $\tau^2 = 1 / 2 \lambda$.

Thus ridge regression can be thought of as a MAP estimator of the model
$$
\begin{aligned}[t]
y_i &\sim \dnorm(\alpha + x' \beta, \sigma) \\
\beta_k &\sim \dnorm(0, (2 \lambda)^{-1/2} )
\end{aligned}
$$
Similarly, the $\beta$ that minimize the $\ell_1$ norm also maximize the probability of random variables iid from the Laplace distribution, $\dlaplace(\beta_k | 0, 1 / \lambda)$.
$$
\begin{aligned}
\arg \max_{\beta} \dlaplace(\beta | 0, 1 / \lambda) &= \arg \max_{\beta} \left(\frac{\lambda}{2}\right)^{K} \prod_{k = 1}^{K} \exp\left(- \lambda |0 -
\beta_k)| \right) \\
&= \arg \max_{\beta} \sum_{k = 1}^{K} - \lambda |\beta_k| \\
&= \arg \min_{\beta} \lambda \sum_{k = 1}^K |\beta_k|
\end{aligned}
$$
Thus lasso regression can be thought of as a MAP estimator of the model,
$$
\begin{aligned}[t]
y_i &\sim \dnorm(\alpha + x' \beta, \sigma) \\
\beta_k &\sim \dlaplace(0, 1 / \lambda)
\end{aligned}
$$


## Bayesian Shrinkage

Consider the single output linear Gaussian regression model with several input variables, given by
$$
\begin{aligned}[t]
y_i \sim \dnorm(\vec{x}_i' \vec{\beta}, \sigma^2)
\end{aligned}
$$
where $\vec{x}$ is a $k$-vector of predictors, and $\vec{\beta}$ are the coefficients.

What priors do we put on $\beta$?

-   **Improper priors:** $\beta_k \propto 1$ This produces the equivalent of
    MLE estimates.

-   **Non-informative priors:** These are priors which have such wide variance
    that they have little influence on the posterior, e.g.
    $\beta_k \sim \dnorm(0, 1e6)$. The primary reason for these (as opposed to
    simply using an improper prior) is that some MCMC methods, e.g. Gibbs sampling
    as used in JAGS or BUGS, require proper prior distributions for all parameters.

**Shrinkage priors** have a few characteristics

-   they push $\beta_k \to 0$

-   while in the other cases, the scale of the prior on $\beta$ is fixed, in
    shrinkage priors there is often a hyperprior on it, e.g.,
    $\beta_k \sim \dnorm(0, \tau)$, where $\tau$ is also a parameter to be estimated.

### Priors

Consider the regression:
$$
y_i \sim \dnorm(\alpha + x_i' \beta, \sigma)
$$

It is assumed that the outcome and predictor variables are standardized such that
$$
\begin{aligned}[t]
\E[y_i] &= 0 & \V[y_i] &= 1 \\
\E[x_i] &= 0 & \V[x_i] &= 1. 
\end{aligned}
$$
Then, the default weakly informative priors are,
$$
\begin{aligned}[t]
\alpha &\sim \dnorm(0, 10) \\
\beta_k &\sim \dnorm(0, 2.5) & k \in \{1, \dots, K\}
\end{aligned}
$$

The weakly informative priors will shrink all the coefficients towards zero.

The amount of shrinkage depends on the amount of data as the likelihood dominates the prior as the amount of data increases.
However, the amount of shrinkage is not estimated from the data.
The prior on each coefficient is independent, and its scale is a constant (2.5 in this example).

Regularization/shrinkage methods estimate the amount of shrinkage. The scale of the priors on the coefficients are hyperparameters, which are estimated from the data.

### Spike and Slab prior

$$
\begin{aligned}[t]
\beta_k | \lambda_k, c, \epsilon  &\sim \lambda_k N(0, c^2) + (1 - \lambda_j) N(0, \epsilon^2) \\
\lambda_k &\sim \dbern(\pi)
\end{aligned}
$$

In the case of the linear regression, an alternative to BMA is to use a
spike-and-slab prior [@MitchellBeauchamp1988a, @GeorgeMcCulloch1993a, @IshwaranRao2005a],
which is a prior that is a discrete mixture of a point mass at 0 and a
non-informative distribution.

The spike and slab prior is a "two-group" solution.
$$
p(\beta_k) = (1 - w) \delta_0 + w \pi(\beta_k)
$$
where $\delta_0$ is a Dirac delta function putting a point mass at 0, and $\pi(\beta_k)$ is an uninformative distribution, e.g. $\pi(\beta_k) = \dnorm(\beta_k | 0, \sigma^2)$ where $\sigma$ is large.

The posterior distribution of $w$ is the probability that $\beta_k \neq 0$, and the conditional posterior distribution $p(\beta_k | y, w = 1)$ is the distribution of $\beta_k$ given that $\beta_k \neq 0$.

See the R package **[spikeslab](https://cran.r-project.org/package=spikeslab)** and he accompanying article [@IshwaranKogalurRao2010a] for an implementation and review of spike-and-slab regressions.

### Normal Distribution

We can apply a normal prior to each $\beta_k$. 
Unlike the weakly informative priors, the prior distributions all share a scale parameter $\tau$.
$$
\beta_k | \tau \sim \dnorm(0, \tau)
$$
We need to assign a prior to $\tau$.

In MAP estimation this is often set to be an improper uniform distribution.
Since in the weakly informative prior, $\beta_k \sim \dnorm(0, 2.5$ for all $k$, a prior on $\tau$ in which the central tendency is the same as the weakly informative prior makes sense.
One such prior is
$$
\tau \sim \dexp(2.5)
$$
TODO: look for better guidance for the prior of $\tau$.

-   This is equivalent to Ridge regression.
-   Unlike most shrinkage estimators, there is a closed form solution to the posterior distribution

### Laplace Distribution

We can also use the Laplace distribution as a prior for the coefficients.
This is called Bayesian Lasso, because the MAP estimator of this model is equivalent to the Lasso estimator.

The prior distribution for each coefficient $\beta_k$ is a Laplace (or double exponential) distribution with scale parameter ($\tau$).
$$
\beta_k | \tau \sim \dlaplace(0, \tau)
$$
Like many priors that have been proposed and used for coefficient shrinkage, this can be represented as a local-global scale-mixture of normal distributions.
$$
\begin{aligned}
\beta_k | \tau &\sim \dnorm(0, \tau \lambda_k) \\
\lambda_k^{-2} &\sim \dexp(1/2)
\end{aligned}
$$
The global scale $\tau$ determines the overall amount of shrinkage.
The local scales, $\lambda_1, \dots, \lambda_K$, allow the amount of shrinkage to vary among coefficients.

### Student-t and Cauchy Distributions

We can also use the Student-t distribution as a prior for the coefficients.
The Cauchy distribution is a special case of the Student t distribution where the degrees of freedom is zero.

The prior distribution for each coefficient $\beta_k$ is a Student-t distribution with degrees of freedom $\nu$, location 0, and scale $\tau$,
$$
\beta_k | \tau \sim \dt(\nu, 0, \tau) .
$$
Like many priors that have been proposed and used for coefficient shrinkage, this can be represented as a local-global scale-mixture of normal distributions.
$$
\begin{aligned}
\beta_k | \tau, \lambda &\sim \dnorm(0, \tau \lambda_k) \\
\lambda_k^{-2} &\sim \dgamma(\nu/2, \nu/2)
\end{aligned}
$$

The degrees of freedom parameter $\nu$ can be fixed to a particular value or estimated.
If fixed, then common values are 1 for a Cauchy distribution, 2 to ensure that there is a finite mean, 3 to ensure that there is a finite variance, and 4 ensure that there is a finite kurtosis.

If estimated, then the 
$$
\nu \sim \dgamma(2, 0.1)
$$
Additionally, it may be useful to truncate the values of $\nu$ to be greater
than 2 to ensure a finite variance of the Student t distribution.

### Horseshore Prior

The Horseshoe prior is defined solely in terms of a global-local mixture.
$$
\begin{aligned}
\beta_k | \tau, \lambda &\sim \dnorm(0, \tau \lambda_k) \\
\lambda_k &\sim \dhalfcauchy(1)
\end{aligned}
$$

The Hierarchical Shrinkage prior originally implemented in rstanarm and proposed by ... replaces the half-Cauchy prior on $\lambda_k$ with a half-Student-t distribution with degrees of freedom $\nu$.
$$
\lambda_k \sim \dt(\nu, 0, 1)
$$
The $\nu$ parameter is generally not estimated and fixed to a low value, with $\nu = 4$ being suggested.
The problem with estimating the Horseshoe prior is that the wide tails of the Cauchy prior produced a posterior distribution with problematic geometry that was hard to sample.
Increasing the degrees of freedom helped to regularize the posterior.
The downside of this method is that by increasing the degrees of freedom of the Student-t distribution it would also shrink large parameters, which the 
Horseshoe prior was designed to avoid.

Regularized horseshoe prior
$$
\begin{aligned}
\beta_k | \tau, \lambda &\sim \dnorm(0, \tau \tilde{\lambda}_k) \\
\tilde{\lambda}^2_k &= \frac{c^2 \lambda^2}{c^2 + \lambda^2} \\
\lambda_k &\sim \dhalfcauchy(1)
\end{aligned}
$$
where $c > 0$ is a constant.
Like using a Student-t distribution, this regularizes the posterior distribution of a Horseshoe prior.
However, it is less problematic in terms of shrinking large coefficients.

Since there is little information about $c$, $c$ is treated as a parameter,
and a prior is placed on it.
$$
c \sim \dt(0, s^2)
$$

## Understanding Shrinkage Models

Suppose that $X$ is a $n \times K$ matrix of predictors,
and $y$ is a $n \times 1$ vector of outcomes.
The conditional posterior for $\beta$ given $(X, y)$ is
$$
\begin{aligned}[t]
p(\beta | \Lambda, \tau, \sigma^2, D) &= \dnorm(\beta | \bar{\beta}, \Sigma), \\
\bar{\beta} &= \tau^2 \Lambda (\tau^2 \Lambda + \sigma^2 (X'X)^{-1})^{-1} \hat{\beta}, \\
\Sigma &= (\tau^{-2} \Lambda^{-1} + \frac{1}{\sigma^{2}} X'X)^{-1}, \\
\Lambda &= \diag(\lambda_1^{2}, \dots, \lambda^{2}_D), \\
\hat{\beta} &= (X'X)^{-1} X'y .
\end{aligned}
$$
If the predictors are uncorrelated with zero mean and variances $\Var(x_k) = s_k^2$, then
$$
X'X \approx n \diag(s_1^2, \dots, s^2_K) ,
$$
and we can use the approximations,
$$
\bar{\beta}_k = (1 - \kappa_k) \hat{\beta}_k,  \\
\kappa_k = \frac{1}{1 + n \sigma^{-2} \tau^2 s_k^2 \lambda_k^2} .
$$
The value $\kappa_k$ is called the *shrinkage factor* for coefficient $\beta_k$.
When $\kappa_k = 0$, then there is no shrinkage and the posterior coefficient is the same as the MLE solution, $\bar{\beta} = \hat{\beta}$.
When $\kappa_k = 1$, then there is complete shrinkage and the posterior coefficient is zero, $\bar{\beta} = 0$.
It also follows that $\bar{\beta} \to 0$ as $\tau \to 0$, and $\bar{\beta} \to \hat{\beta}$ as $\tau \to \infty$.


```r
shrinkage_factor <- function(n, sigma = 1, tau = 1, sd_x = 1, lambda = 1) {
  1 / 1 + n * tau ^ 2 * sd_x ^ 2 * lambda ^ 2 / sigma ^ 2
}
```

## Choice of Hyperparameter on $\tau$

The value of $\tau$ and the choice of its hyper-parameter has a big influence on the sparsity of the coefficients.

<!-- @CarvalhoPolsonScott2009a suggest -->
<!-- $$ -->
<!-- \tau \sim \dhalfcauchy(0, \sigma), -->
<!-- $$ -->
<!-- while @PolsonScott2011a suggest, -->
<!-- $$ -->
<!-- \tau \sim \dhalfcauchy(0, 1) . -->
<!-- $$ -->

<!-- @PasKleijnVaart2014a suggest -->
<!-- $$ -->
<!-- \tau \sim \dhalfcauchy(0, p^{*} / n) -->
<!-- $$ -->
<!-- where $p^*$ is the true number of non-zero parameters, -->
<!-- and $n$ is the number of observations. -->
<!-- They suggest $\tau = p^{*} / n$ or $\tau p^{*}  / n \sqrt{log(n / p^{*})}$. -->
<!-- Additionally, they suggest restricting $\tau$ to $[0, 1]$. -->

@PiironenVehtari2017a treat the prior on $\tau$ as the implied prior on the number of effective parameters.
The shrinkage can be understood as its influence on the number of effective parameters, $m_{eff}$,
$$
m_{\text{eff}} = \sum_{j = 1}^K (1 - \kappa_j) .
$$
This is a measure of effective model size.

@PiironenVehtari2017a show that for a given $n$ (data standard deviation), $\tau$, $\lambda_k$, and $\sigma$, the and variance of $m_{eff}$ 
$$
\begin{aligned}[t]
\E[m_{eff} | \tau, \sigma] &= \frac{\sigma^{-1} \tau \sqrt{n}}{1 + \sigma^{-1} \tau \sqrt{n}} K , \\
\Var[m_{eff} | \tau, \sigma] &= \frac{\sigma^{-1} \tau \sqrt{n}}{2 (1 + \sigma^{-1} \tau \sqrt{n})2} K .
\end{aligned}
$$

Given a prior guess about the sparsity $\beta$, a prior should be chosen such that it places mass near that guess.
Let $k_0 \in [0, K]$ be the expected number of non-zero elements of $\beta$, then choose $\tau_0$ such that
$$
\tau_0 = \frac{k_0}{K - k_0}\frac{\sigma}{\sqrt{n}}
$$

This prior depends on the expected sparsity of the solution, which depends on the problem. 
PiironenVehtari2017a provide no guidence on how to select $p_0$.
Perhaps a simpler model, e.g. lasso could be used to estimate $p_0$.

-  @DattaGhosh2013a warn against empirical Bayes estimators of $\tau$ for the horseshoe prior as it can collapse to 0.
-  @ScottBerger2010a consider marginal maximum likelihood estimates of $\tau$. -->
-  @PasKleijnVaart2014a suggest that an empirical Bayes estimator truncated below at $1 / n$.

<!-- Densities of the shrinkage parameter, $\kappa$, for various shrinkage distributions where $\sigma^2 = 1$, $\tau = 1$, for $n = 1$. -->

<img src="regression-shrinkage_files/figure-html/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" />

## Differences between Bayesian and Penalized ML

$$
\log p(\theta|y, x) \propto \frac{1}{2 \sigma} \sum_{i = 1}^n (\vec{x}_i\T \vec{\beta} - y_i)^2 + \lambda \sum_{k} \beta_k^2
$$
In the first case, the log density of a normal distribution is,
$$
\log p(y | \mu, x) \propto \frac{1}{2 \sigma} (x - \mu)^2
$$
The first regression term is the produce of normal distributions (sum of their log probabilities),
$$
y_i \sim \dnorm(\vec{x}_i\T \vec{\beta}, \sigma)
$$
The second term, $\lambda \sum_{k} \beta_k^2$ is also the sum of the log of densities of i.i.d. normal densities, with mean 0, and scale $\tau = 1 / 2 \lambda$,
$$
\beta_k \sim \dnorm(0, \tau^2)
$$

The only difference in the LASSO is the penalty term, which uses an absolute value penalty for $\beta_k$.
That term corresponds to a sum of log densities of i.i.d. double exponential (Laplace) distributions.
The double exponential distribution density is similar to a normal distribution,
$$
\log p(y | \mu, \sigma) \propto - \frac{|y - \mu|}{\sigma}
$$
So the LASSO penalty is equivalent to the log density of a double exponential distribution with location $0$, and scale $1 / \lambda$.
$$
\beta_k \sim \dlaplace(0, \tau)
$$

There are several differences between Bayesian approaches to shrinkage and penalized ML approaches.

The point estimates:

-   ML: mode
-   Bayesian: posterior mean (or median)

In Lasso

-   ML: the mode produces exact zeros and sparsity
-   Bayesian: posterior mean is not sparse (zero)

Choosing the shrinkage penalty:

-   ML: cross-validation
-   Bayesian: a prior is placed on the shrinkage penalty, and it is estimated as part of the posterior.

## Examples
