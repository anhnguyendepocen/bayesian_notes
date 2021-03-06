
# Model Comparison

Don't check, but compare.

-   Information criteria
-   Predictive accuracy

Model comparison based on predictive performance

## Models

-   **Model comparison**: defining criteria to rank models for which is best.
-   **Model selection**: choose the *best* model
-   **Model averaging**: combine models into a single meta-model.

## Classes of Model Spaces

See Vehtari and Ojanen (2012) and Piironen and Vehtari (2015).

Let $\mathcal{M} = \{M_1, \dots M_K\}$ be a set of $K$ models.
Let $M_T$ be the model for the true data generating process.
Let $M_R$ be a reference model which is not the true model, but is the best available model to predict future observations.

| Generalization utility estimation |  $\mathcal{M}-open$ | $M_T, M_R \notin \mathcal{M}$ |
| Model Space approach | $\mathcal{M}-closed$ |  $M_T \in \mathcal{M}$ |
| Reference Model approach |  $\mathcal{M}-completed$ |  $M_R \in \mathcal{M}$ |

The $\mathcal{M}$-closed view asserts that there is a true DGP model *and* that model is in the set of models under consideration.

The $\mathcal{M}$-open view either asserts that there is no true DGP or does not care. 
It only compares models in the set against each other.

The $\mathcal{M}$-complete view does not believe there is a true model in the set of models, but still uses a reference model which is believed be the best available description of the future observations.

## Continuous model expansion

Continuous model expansion is embedding the current model in a more general model in which it is a special case.

-   add new parameters
-   broaden the class of models, e.g. normal to a $t$
-   combine different models into a super-model that includes both as special cases
-   add new data. For example, embed the data into a hierarchical model to draw strength from other data.

More formally, suppose that the  old model is a $p(y, \theta)$ is embedded or replaced by $p(y, y^*, \theta, \phi)$, where
$$
p(\theta, \phi | y, y^*) \propto p(\phi) p(\theta | \phi) p(y, y^* | \theta, \phi)  .
$$
This will require a specifying

-   $p(\theta | \phi)$: a new prior on $\theta$ that is conditional on the new parameters, $\phi$.
-   $p(\phi)$: a prior for the new parameters, $\phi$.

Continuous model expansion can also be used to fit 

Some examples of continuous model expansion:

-   The normal distribution can be expanded to the Student-t distribution since
    $$
    \dnorm(\mu, \sigma^2) = \dt(\nu = \infty, ) .
    $$
    Let $\nu$ be a parameter to be estimated instead of imposing $\nu = \infty$
    as in the normal distribution.
    
-   Any case where a distribution is a special case of a more general distribution:

    -   Normal to skew normal
    -   Student-t to skew Student-t
    -   Normal and Laplace to the [Exponential Power Distribution(https://en.wikipedia.org/wiki/Generalized_normal_distribution#Version_1)
    -   Binomial to Beta-Binomial
    -   Poisson to Negative-Binomial
    -   ... and many others
    
-   Linear regression can be thought of a case of model expansion, where 
    a model where all observations have the same mean,
    $$
    y_i \sim \dnorm(\mu, \sigma^2) ,
    $$
    is replaced by one in which each observation has a different mean,
    $$
    y_i \sim \dnorm(\mu_i, \sigma^2)
    $$
    with a particular model,
    $$
    \mu_i = x_i \beta .
    $$
    
-   Given a regression where observation $i$ is a group $k \in \{1, \dots, K \}$,
    a regression where the slope an intercept coefficients are assumed to be
    the same across groups,
    $$
    y_{i,k} \sim \dnorm(\alpha + x_i \beta, \sigma^2) ,
    $$
    can be generalized to a model in which the intercepts and slopes vary across groups,
    $$
    y_{i,k} \sim \dnorm(\alpha_k + x_i \beta_k, \sigma^2) .
    $$
-   A linear regression with heterskedastic errors,
    $$
    y_i \sim \dnorm(\alpha + x_i \beta, \sigma_i^2)
    $$
    is a continuous model expansion of a homoskedastic regression model which 
    assumes $\sigma_i = \sigma$ for all $i$.
    
-   The regression model which adds a variable(s) is a continuous model expansion.
    For example, the regression,
    $$
    y_i \sim \dnorm(\alpha + x_1 \beta_1, \sigma_i^2), 
    $$
    is a special case of the larger regression model,
    $$
    y_i \sim \dnorm(\alpha + x_1 \beta_1 + x_2 \beta_2, \sigma_i^2) ,
    $$
    where $\beta_2 = 0$.
    Adding a variable to a regression is estimating the coefficient of that 
    new variable rather than assuming it is zero.
    
-   Continuous model expansion can also apply to cases where several models
    can be subsumed into a larger model in which they are all special cases.
    
There are several issues with continuous model expansion:

-   Model selection choices can lead to overfitting. Embedding a model inside a
    larger model, incorporates *some*, but not all sources of uncertainty.
    Using continuous model expansion is better than choosing one, "best" model.
    However, the choice of how to expand the model will be but one of many possibilities
    and that choice can subtly overfit the data.

-   Specifying a more general model can be costly in both researcher time 
    and computational time.
    
-   Even disregarding computational constraints, no useful model can be completely general. See the [No Free Lunch Theorem](https://en.wikipedia.org/wiki/No_free_lunch_theorem).

## Discrete Model Expansion

Suppose that you are considering $\mathcal{M} = \{M_1, \dots, M_K\}$ models, estimate a model that is a weighted average of those models.
$$
p(\theta | y) = \sum_{k = 1}^K \pi_k p(\theta_k | y), 
$$
where $\pi_k \geq 0$ and $\sum \pi_k = 1$.
Like continuous model expansion this embeds models in a larger meta-model.
However, whereas the continuous model expansion generally involves models being specific cases of a continuous parameter value in the meta-model, the discrete model expansion is a brute-force approach that treats models as discrete and independent, and averages them.
There are two general approaches to this,

1.   Mixture models estimate $\pi_k$ simultaneously with $p(\theta_k | y)$.
1.   Bayesian model averaging is a two step process.

     1.  Estimate each $p(\theta_k | y)$ 
     1.  Define weights $\pi_k$ and average the models.


## Out-of-sample predictive accuracy 

Consider data $y_1, \dots, y_n$, which is independnet given parameters $\theta$.
Thus the likelihood can be decomposed into a product of pointwise likelihoods,
$$
p(y | \theta) = \prod_{i = 1}^n p(y_i | \theta) .
$$
Suppose a prior distribution $p(\theta)$ and a posterior predictive distribution for new data $\tilde{y}$,
$$
p(\tilde{y} | y) = \int p(\tilde{y}_i | \theta) p(\theta | y)\,d\theta .
$$
The expected log-predictive accuracy for a new point is,
$$
\begin{aligned}[t]
\text{elpd} &= \text{expected log pointwise predictive density for a new dataset} \\
&= \sum_{i = 1}^n \int p_t(\tilde{y}_i) \log p(\tilde{y}_i | y)\,d\tilde{y}_i ,
\end{aligned}
$$
where $p_t(\tilde{y}_i)$ is the distribution of the true DGP for $\tilde{y}_i$.
Since the true DGP is unknown, it will be needed to be approximated.
The most common way to approximate $p_t(\tilde{y}_i)$ is via either cross-validation or information criteria.

$$
\begin{aligned}[t]
\text{lpd} &= \text{log pointwise predictive density} \\
&= \sum_{i = 1}^n \log p(y_i | y) \\
&= \sum_{i = 1}^n \log \int p(y_i | \theta) p(\theta | y) \,d\theta .
\end{aligned}
$$
The lpd of observed data is overly optimistic for future data.
To compute the lpd from $S$ draws from a posterior distribution $p_{post}(\theta)$,
$$
\begin{aligned}[t]
\widehat{\text{lpd}} &= \text{computed log pointwise predictive density} \\
&= \sum_{i = 1}^n \log \left( \frac{1}{S} \sum_{s = 1}^S p(y_i | 
\theta^s) \right) .
\end{aligned}
$$

The Bayesian LOO-CV estimate of elpd is,
$$
\text{elpd}_{\text{loo}} = \sum_{i = 1}^n \log p(y_i | y_{-i}) ,
$$
where
$$
p(y_i | y_{-i}) = \int p(y_i | \theta) p(\theta | y_{-i})\,d\theta .
$$

The value of $\text{elpd}_{\text{loo}}$ can be calculated by cross-validation (running the model $n$ times) or by an approximation of LOO-CV using importance sampling, which PSIS-LOO being the best implementation of this approach.

## Stacking

Stacking is a method for averaging (point) estimates from models.
It proceeds in two steps.

1.  Fit $K$ models where each model where $\hat{y}_i$ is predicted value of $y_i$ from a model trained on data not including $y$ (e.g. LOO-CV).

1.  Calculate a weight for each model by minimizing the LOO-mean squared error,
    $$
    \hat{w} = \arg \min_{w} \sum_{i = 1}^n \left( y_i - \sum_{k} w_k \hat{y}_i \right)^2
    $$

1.  The pint prediction for a new point is,
    $$
    \hat{\tilde{y}} = \sum_{k = 1}^K \hat{w}_k f_k\left(\tilde{x} | \tilde{\theta}_k, y_{1:n} \right)
    $$

Whereas stacking is typically used with point estimates, CITE generalize stacking to use proper scoring rules.
In particular, CITE use the logarithmic scoring rule (e.g. the log predictive distribution).
This is implemented in the **loo** package.

## Posterior Predictive Criteria

Most of these notes summarize the more complete treatment in @GelmanHwangVehtari2013a and @VehtariGelmanGabry2015a.

### Summary and Advice 

Models can be compared using its *expected predictive accuracy* on new data. Ways to evaluate predictive accuracy:

-   log posterior predictive density: $\log p_post(\tilde{y})$. The log probability of observing new 
-   [scoring rules](https://en.wikipedia.org/wiki/Scoring_rule) or [loss functions](https://en.wikipedia.org/wiki/Loss_function) specific to the problem/research question

Several methods to estimate expected log posterior predictive density (elpd)

-   within-sample log-posterior density (biased, too optimistic)
-   information criteria: WAIC, DIC, AIC with correct the bias within-sample log-posterior density with a penalty (number of parameters)
-   cross-validation: estimate it using heldout data

What should you use? 

-   Use the Pareto Smoothed Importance Sampling LOO [@VehtariGelmanGabry2015a] implemented in the **[loo](https://cran.r-project.org/package=loo)** package:

    -   It is computationally efficient as it doesn't require completely 
        re-fitting the model, unlike actual cross-validation

    -   it is fully Bayesian, unlike AIC and DIC

    -   it often perform better than WAIC

    -   it provides indicators for when it is a poor approximation (unlike AIC, 
        DIC, and WAIC)

    -   next best approximation would be the WAIC. No reason to use AIC or DIC ever.

-   For observations which the PSIS-LOO has $\hat{k} > 0.7$ (the estimator has
    infinite variance) and there aren't too many, use LOO-CV.

-   If PSIS-LOO has many observations with with $k > 0.7$, then use LOO-CV or k-fold CV

-   If the likelihood doesn't easily partition into observations or LOO is not 
    an appropriate prediction task, use the appropriate CV method (block k-fold,
    partitioned k-fold, time-series k-fold, rolling forecasts, etc.)
    
-   Note that AIC/DIC/WAIC/CV vs. BIC/Bayes Factors are not different estimators of the same estimand. They are answering fundamentally different questions. 
    Cross validations and its IC approximations are asking a question in a $\mathcal{M}$-open world as to which model predict the best (w.r.t. a loss function).
    Bayes factors, BIC, and marginal likelihood-based measures are used to find the true model, with the assumption that the true model is one of the models under consideration (which unless another human computationally generated the data is unlikely to be the case).

### Expected Log Predictive Density

Let $f$ be the true model, $y$ be the observed data, and $\tilde{y}$ be future data or alternative data not used in fitting the model.
The out-of-sample predictive fit for new data is
$$
\log p_{post}(\tilde{y}_i) = -\log \E_{post}(p(\tilde{y}_i)) = \log \int p(\tilde{y}_i | \theta) p_{post}(\theta) d\,\theta
$$ 
where $p_{post}(\tilde{y}_i)$ is the predictive density for $\tilde{y}_i$ from $p_{post}(\theta)$. $\E_{post}$ is an expectation that averages over the values posterior distribution of $\theta$.

Since the future data $\tilde{y}_i$ are unknown, the **expected out-of-sample log predictive density** (elpd) is,
$$
\begin{aligned}[t]
\mathrm{elpd} &= \text{expected log predictive density for a new data point} \\
&= E_f(\log p_{post}(\tilde{y}_i)) \\
&= \int (\log p_{post}(\tilde{y}_i)) f(\tilde{y}_i) \,d\tilde{y}_i
\end{aligned}
$$

## Bayesian Model Averaging

Suppose there is an exhaustive list of candidate models, $\{M_k\}_{k = 1}^K$, the distribution over the model space is,
$$
p(M | D) \propto p(D | M) p(M).
$$
The predictions from Bayesian Model Averaging (BMA) are
$$
p(\tilde{y} | D) = \sum_{k = 1}^{K} p(\tilde{y} | D, M_k) p(M_k | D)
$$
In BMA each model is weighted by its marginal likelihood,
$$
p(M_k | y) = \frac{p(y | M_k) p(M_k)}{\sum_{k = 1}^K p(y | M_k) p(M_K)},
$$
where
$$
p(y | M) = \int p(y | \theta_k, M_k) p(\theta_k| M_k) \,d\theta_k.
$$

-   In the $\mathcal{M}$-closed case, BMA will asymptotically select the correct model.
-   In the $\mathcal{M}$-open and -complete cases, it will asympmtotically select the closest, in terms of KL-divergence, model to the true model.
-   Since the BMA weights by marginal likelihood, these weights extremely sensitive to the choices of the priors $p(\theta_k)$ for each model. 

The sensitivity to prior distributions make the BMA weights suspect.
The difficulty of computing marginal likelihood generally make the BMA hard to generalize.

BMA has been most successfully implemented in (generalized) linear regression, where a particular choice of prior ([Zellner's g-prior](https://en.wikipedia.org/wiki/G-prior)) provides an analytical solution to the Bayes' Factor with respect to the null model.
However, this is also the area where methods using regularization and sparse shrinkage priors have made extensive progress recently.
Sparse shrinkage priors, e.g. horseshoe priors, and the use of methods that provide a sparse summarization of the prior, e.g. projection-prediction, provide a competitive and more coherent solution to the variable selection problem in regression than BMA.

## Pseudo-BMA

Pseudo-BMA is similar to Bayesian model averaging, but instead of using weighting models by marginal likleihoods, it weights models using an approximation of the predictive distribution: e.g. AIC, DIC, or WAIC.

The use of the predictive distribution rather than the marginal likelihood makes the weights less sensitive to the prior distributions of the priors.

CITE propose using the expected log-pointwise predictive density (elpd) PSIS-LOO weights to weight each model.
$$
w_k = \frac{\exp\left( \widehat{\text{elpd}}_{\text{loo}}^k \right)}{ \sum_{k = 1}^K \exp \left( \widehat{\text{elpd}}_{\text{loo}}^k \right)}
$$
where $\widehat{\text{elpd}}_{\text{loo}}^k$ is estimated using PSIS-LOO.

These point-estimates of the elpd are adjusted by estimates of the uncertainty calculated via a log-normal approximation or Bayesian bootstrap [CITE].
The effect of adjusting for uncertainty is to regularize weights by adjusting them towards equal weighting for each model, and away from weights of 0 or 1.

This is implemented in the **loo** package.

## LOO-CV via importance sampling

Leave-one-out cross validation (LOO-CV) is costly because it requires re-estimating the model $n$ times.
The LOO predictive density is,
$$
p(y_{i} | y_{-i})  = \int p(y_i | \theta) p(\theta | y_{-i})\,d\theta .
$$

However, if the model was computed for $y_{1:n}$ it seems wasteful to ignore it when calculating LOO posterior distributions since $p(\theta | y_{1:n}) \approx p(\theta | y_{-i})$.
Importance sampling can be used to sample from the LOO predictive density using the already estimated posterior density as a proposal distribution.

Suppose that there a $S$ simulation draws from the full-posterior $p(\theta | y)$, the importance sampling weights are
$$
r^{s}_i = \frac{1}{p(y_i | \theta^s)} \propto \frac{p(\theta^s | y_{-i})}{p(\theta^{s} | y)}
$$
The LOO predictive distribution is approximated by,
$$
\begin{aligned}[t]
p(y_i | y_{-i}) &= \int p(y_i | \theta) \frac{p(\theta|y_{-i})}{p(\theta |y)} p(\theta | y)\,d \theta \\
&\approx \frac{\sum_{s = 1}^S r^s_i p(y_i | \theta^{s})}{\sum_{s = 1}^S r_i^s
} .
\end{aligned}
$$

An issue with these proposal weights it that the full posterior distribution is likely to be narrower than the LOO posterior distribution.
This causes problems for importance sampling, and the weights can be unstable.
@VehtariGelmanGabry2017b propose a method to regularize the importance weights called Pareto-Smoothed Importance Sampling (PSIS-LOO).

A useful side-effect of this method for smoothing these importance weights is that it also provides an indicator for when these weights are unstable.
The PSIS is so-called because it estimates a generalized Pareto distribution.
Observations where the estimated shape parameter of that Pareto distribution is $\hat{k} > 0.7$ are unstable, and the PSIS-LOO approximation is poor [@VehtariGelmanGabry2017b].

If a few observations have $\hat{k} > 0.7$, each of those should be re-estimated with LOO-CV.
If many observations have $\hat{k} > 0.7$, then it may be worth re-estiamting the model using a $k$-fold cross validation.


## Selection induced Bias

See Pirronen and Vehtari (2015), p. 10

Using the training set to select models produces an optimistic estimate. High variance.
The model selection process needs to be CV in order to get good estimate of generalization error.

-   CV/WAIC/DIC are highly variable
-   MPP/BMA less so
-   Projection methods are the least.

