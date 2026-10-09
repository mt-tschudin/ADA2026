# ADA — Lecture 5: Regression for Disentangling Data

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturers:** Viktor Moskvoretskii, Robert West  
**Lecture:** 5 — Regression for Disentangling Data  
**Date:** 7 October 2026

> These notes summarize the key concepts from the lecture slides. The main theme is how **linear regression can be used not only for prediction, but also for descriptive analysis: comparing groups, accounting for correlated predictors, quantifying uncertainty, and interpreting additive, interaction, and transformed models.**

---

# 1. Linear regression: the basic setup

Suppose we have \(n\) data points

$$
(X_i, y_i),
\qquad i=1,\dots,n,
$$

where:

- \(X_i\) is a \(k\)-dimensional vector of predictors/features
- \(y_i\) is a scalar outcome

Linear regression models the outcome as

$$
y_i = X_i\beta + \epsilon_i,
$$

or equivalently,

$$
y_i
=
\beta_1X_{i1}
+\beta_2X_{i2}
+\cdots
+\beta_kX_{ik}
+\epsilon_i.
$$

Here:

- \(\beta=(\beta_1,\dots,\beta_k)\) is the vector of coefficients
- \(\epsilon_i\) is the error term for observation \(i\)

Usually,

$$
X_{i1}=1,
$$

so \(\beta_1\) acts as the **intercept**.

---

## 2. One-predictor regression

With one predictor \(X\), the model is

$$
y \approx \beta_1+\beta_2X.
$$

Interpretation:

- \(\beta_1\): intercept
- \(\beta_2\): slope

The slope tells us how much the estimated mean outcome changes when \(X\) increases by one unit.

---

# 3. Least squares

The fitted coefficients are chosen to make the prediction errors as small as possible.

For observation \(i\),

$$
\epsilon_i = y_i-X_i\beta.
$$

Least squares minimizes

$$
\sum_{i=1}^{n}
\left(
y_i-X_i\hat\beta
\right)^2.
$$

So regression chooses the coefficient vector \(\hat\beta\) that minimizes the **sum of squared residuals**.

### Why squares?

Squaring:

- penalizes large errors strongly
- prevents positive and negative errors from canceling
- leads to a mathematically convenient optimization problem

---

# 4. Three uses of regression

The lecture distinguishes three broad uses.

### Prediction
Use the fitted model to estimate \(y\) for new predictor values \(X\) not seen during fitting.

### Descriptive data analysis
Compare average outcomes across subgroups of data.

This is the main focus of this lecture.

### Causal modeling
Study how outcomes change when predictors are manipulated.

This requires stronger assumptions and is treated separately in the course.

---

# 5. Regression as comparison of average outcomes

## Binary predictor

Suppose

$$
X_i=\text{mom\_hs}
$$

where

$$
X_i=
\begin{cases}
0 & \text{mother did not finish high school}\\
1 & \text{mother finished high school}
\end{cases}
$$

and

$$
y_i=\text{kid\_score}.
$$

Model:

$$
y_i
=
\beta_1+\beta_2X_i+\epsilon_i.
$$

Example:

$$
\text{kid\_score}
=
78+12\cdot\text{mom\_hs}+\text{error}.
$$

### If \(X=0\)

$$
E[y\mid X=0]
=
\beta_1
=
78.
$$

So \(\beta_1\) is the mean score for children whose mothers did not finish high school.

### If \(X=1\)

$$
E[y\mid X=1]
=
\beta_1+\beta_2
=
78+12
=
90.
$$

Therefore:

$$
\boxed{
\beta_2
=
\text{difference in mean outcome between the two groups}
}
$$

For one binary predictor, linear regression is essentially another way of comparing two group means.

---

# 6. Why the group mean appears automatically

The value \(m\) minimizing

$$
\sum_{i=1}^{n}(y_i-m)^2
$$

is the arithmetic mean.

Differentiate with respect to \(m\):

$$
-2\sum_{i=1}^{n}(y_i-m)=0.
$$

Therefore,

$$
m
=
\frac{1}{n}
\sum_{i=1}^{n}y_i.
$$

This is why least-squares regression with a binary predictor recovers the group means.

---

# 7. Continuous predictors

Suppose

$$
X_i=\text{mom\_iq}
$$

and

$$
y_i=\text{kid\_score}.
$$

Example model:

$$
\text{kid\_score}
=
26+0.6\cdot\text{mom\_iq}+\text{error}.
$$

### Intercept

$$
\beta_1=26
$$

is the estimated mean outcome at

$$
\text{mom\_iq}=0.
$$

This may be mathematically defined but scientifically meaningless if IQ \(=0\) lies far outside the observed range.

### Slope

$$
\beta_2=0.6
$$

means:

> A one-unit increase in mother's IQ is associated with a \(0.6\)-point increase in the estimated mean child score.

For example:

$$
\text{mom\_iq}=100
$$

gives

$$
26+0.6(100)=86.
$$

---

# 8. Multiple predictors

Suppose the model includes both high-school completion and IQ:

$$
y_i
=
\beta_1
+
\beta_2X_{i2}
+
\beta_3X_{i3}
+
\epsilon_i.
$$

Example:

$$
\text{kid\_score}
=
26
+
6\cdot\text{mom\_hs}
+
0.6\cdot\text{mom\_iq}
+
\text{error}.
$$

For mothers who did not finish high school:

$$
\text{kid\_score}
=
26+0.6\cdot\text{mom\_iq}.
$$

For mothers who finished high school:

$$
\text{kid\_score}
=
32+0.6\cdot\text{mom\_iq}.
$$

The two groups therefore have:

- different intercepts
- the same slope

The coefficient \(6\) represents a vertical shift between the two regression lines.

---

# 9. Interaction terms

An interaction allows the effect of one predictor to depend on another predictor.

Model:

$$
y_i
=
\beta_1
+
\beta_2X_{i2}
+
\beta_3X_{i3}
+
\beta_4X_{i2}X_{i3}
+
\epsilon_i.
$$

Example:

$$
\text{kid\_score}
=
-11
+
51\cdot\text{mom\_hs}
+
1.1\cdot\text{mom\_iq}
-
0.5\cdot\text{mom\_hs}\cdot\text{mom\_iq}
+
\text{error}.
$$

### If mom_hs = 0

$$
\text{kid\_score}
=
-11+1.1\cdot\text{mom\_iq}.
$$

So:

- intercept \(=-11\)
- slope \(=1.1\)

### If mom_hs = 1

$$
\text{kid\_score}
=
(-11+51)
+
(1.1-0.5)\cdot\text{mom\_iq}.
$$

Thus:

- intercept \(=40\)
- slope \(=0.6\)

Without interaction, the effect of a predictor is constant. With interaction, the effect depends on another predictor.

---

# 10. Why not just compare group means directly?

Regression becomes useful when predictors are correlated.

The lecture gives a constructed example where:

- mothers who finished high school have higher child scores
- Mercedes ownership is strongly correlated with high-school completion
- Mercedes ownership itself has no difference in child score once high-school completion is fixed

A raw comparison of Mercedes drivers and non-drivers therefore appears misleading.

Regression can include both predictors simultaneously:

$$
\text{kid\_score}
=
78
+
12\cdot\text{mom\_hs}
+
0\cdot\text{mercedes}
+
\text{error}.
$$

So regression helps **disentangle associations among correlated predictors**.

### Important caveat

This does not automatically make the result causal. It describes associations conditional on the included predictors.

---

# 11. Quantifying uncertainty in regression coefficients

Statistical software reports more than the fitted coefficients.

Typical outputs include:

- coefficient estimate
- standard error
- test statistic
- p-value
- residual standard error
- \(R^2\)
- adjusted \(R^2\)

A common null hypothesis for coefficient \(\beta_j\) is

$$
H_0:\beta_j=0.
$$

The p-value asks how extreme the estimated coefficient would be if the true coefficient were zero.

As always:

> A p-value is not the probability that the null hypothesis is true.

---

# 12. Residuals

For data point \(i\), the residual is

$$
r_i
=
y_i-X_i\hat\beta.
$$

Equivalently,

$$
r_i=y_i-\hat y_i.
$$

It is the vertical difference between:

- observed outcome
- predicted outcome

In ordinary least squares with an intercept, the mean residual is zero:

$$
\frac{1}{n}\sum_{i=1}^{n}r_i=0.
$$

So total overestimation and underestimation balance out.

---

# 13. Residual variance

Residual variance measures how much variability remains unexplained by the fitted model.

Conceptually,

$$
\hat\sigma^2
\approx
\text{average squared residual}.
$$

This is the **unexplained variance**.

---

# 14. Coefficient of determination \(R^2\)

The lecture defines the fraction of variance explained by the model as

$$
R^2
=
1-
\frac{\hat\sigma^2}{s_y^2},
$$

where:

- \(\hat\sigma^2\) = residual / unexplained variance
- \(s_y^2\) = variance of the observed outcome

Equivalent common notation is

$$
R^2
=
1-
\frac{SS_{\mathrm{res}}}{SS_{\mathrm{tot}}}.
$$

Interpretation:

- \(R^2\approx0\): the model explains little outcome variation
- \(R^2\approx1\): the model explains most outcome variation

For example,

$$
R^2=0.865
$$

means about \(86.5\%\) of the observed variance is explained by the fitted model.

### Important warning

A single \(R^2\) value does not guarantee that the model is appropriate. Very different datasets can have the same \(R^2\).

Always inspect the data and residual structure.

---

# 15. Regression assumptions

## 15.1 Validity

The lecture highlights three aspects:

- the outcome measure should accurately reflect the phenomenon of interest
- the model should include relevant predictors
- the model should generalize to the cases where it will be applied

---

## 15.2 Linearity

Linear regression requires linearity in the **predictors used in the model**, not necessarily in the original raw variables.

The model has the form

$$
y_i
=
\beta_1X_{i1}
+\cdots+
\beta_kX_{ik}
+\epsilon_i.
$$

But predictors can themselves be transformations of raw inputs, for example:

$$
X=x^2,
$$

$$
X=\log x,
$$

$$
X=x_1x_2,
$$

or indicator variables obtained by discretization.

So the relationship between raw input \(x\) and outcome \(y\) may be nonlinear while the regression remains linear in its coefficients.

---

## 15.3 Independence of errors

Errors should not interact systematically across observations.

This assumption can be violated in:

- time series
- spatial data
- repeated measurements
- clustered observations

---

## 15.4 Equal variance of errors

The variance of residuals should be approximately constant across predictor values.

This is called **homoscedasticity**.

---

## 15.5 Normality of errors

Residuals are often assumed to be approximately Gaussian.

The lecture notes that equal variance and Normality may be less important in practice than validity and correct model specification, depending on the task.

---

# 16. Linear transformations of predictors

A linear transformation of a predictor changes the coefficient but not the fitted predictions or \(R^2\).

For example, expressing height in millimeters versus miles changes the coefficient because the unit changes, but:

- predicted outcomes stay the same
- model fit stays the same
- \(R^2\) stays the same

---

# 17. Mean-centering predictors

Mean-centering means subtracting the predictor mean:

$$
X_{ij}
\leftarrow
X_{ij}
-
\operatorname{mean}(X_{1j},\dots,X_{nj}).
$$

After centering,

$$
\operatorname{mean}(X_j)=0.
$$

The units do not change.

If IQ is centered, then:

- \(0\) means average IQ
- \(+10\) means 10 IQ points above average
- \(-10\) means 10 IQ points below average

---

# 18. Why mean-centering helps interpretation

Without centering, the intercept means the predicted outcome when all predictors equal zero.

That may be meaningless.

For example:

$$
\text{kid\_score}
=
26+0.6\cdot\text{mom\_iq}.
$$

The intercept \(26\) corresponds to mom IQ \(=0\), which is not a realistic reference point.

After centering,

$$
\text{mom\_iq}_c
=
\text{mom\_iq}
-
\overline{\text{mom\_iq}},
$$

the intercept corresponds to the predicted mean child score when mother's IQ is at the sample mean.

---

# 19. Interpreting coefficients after mean-centering

The lecture uses index \(j\) to refer to predictor/coefficient position.

### \(j=1\): intercept

After mean-centering all predictors,

$$
\beta_1
$$

is the estimated mean outcome when every predictor is at its mean value.

### \(j>1\): main predictor coefficients

Without interactions:

$$
\beta_j
$$

is the estimated mean increase in \(y\) for a one-unit increase in predictor \(X_j\).

With interactions:

$$
\beta_j
$$

is the estimated mean increase in \(y\) for a one-unit increase in \(X_j\) **when all other interacting predictors are at their mean values**.

This is one of the main reasons centering is especially useful when interaction terms are present.

---

# 20. Standardization with z-scores

After mean-centering, divide by the standard deviation:

$$
X_{ij}
\leftarrow
\frac{
X_{ij}
-
\operatorname{mean}(X_{1j},\dots,X_{nj})
}{
\operatorname{sd}(X_{1j},\dots,X_{nj})
}.
$$

The resulting variable is a **z-score**.

Then:

- mean \(=0\)
- standard deviation \(=1\)

Interpretation:

$$
z=1
$$

means one standard deviation above the mean.

### Why standardize?

Predictors originally measured in different units become comparable.

Examples:

- IQ points
- Swiss francs
- centimeters

After z-scoring, coefficients refer to changes per one-standard-deviation change in the predictor.

---

# 21. Logarithmic outcomes

If the outcome is non-negative and heavy-tailed, it can be useful to model

$$
\log y
$$

instead of \(y\).

Example:

$$
\log y_i
=
b_0
+
b_1X_{i1}
+
b_2X_{i2}
+\cdots
+
\epsilon_i.
$$

Exponentiating gives

$$
y_i
=
e^{b_0}
e^{b_1X_{i1}}
e^{b_2X_{i2}}
\cdots
e^{\epsilon_i}.
$$

So an additive model in log-space becomes multiplicative in the original outcome scale.

---

# 22. Interpreting coefficients with log outcomes

If

$$
\log y=b_0+b_1X,
$$

then increasing \(X\) by one unit multiplies \(y\) by

$$
e^{b_1}.
$$

For small \(b_1\),

$$
e^{b_1}\approx1+b_1.
$$

So:

$$
b_1=0.05
$$

implies approximately a \(5\%\) increase in \(y\) for a one-unit increase in \(X\).

---

# 23. Logarithmic predictors and outcomes

The bonus slide considers

$$
\log y=a+b\log X.
$$

Then

$$
y=e^aX^b.
$$

Approximately:

> A 1% increase in \(X\) is associated with a \(b\)% increase in \(y\).

Example:

$$
b=2
$$

means a 1% increase in \(X\) is associated with approximately a 2% increase in \(y\).

---

# 24. Beyond linear regression: generalized linear models

The lecture briefly introduces models for outcomes that do not naturally fit ordinary linear regression.

### Logistic regression
Used for binary outcomes.

### Poisson regression
Used for non-negative integer count outcomes.

---

# 25. Difference-in-differences

The lecture ends with a first taste of causal reasoning.

Suppose there are two groups:

- treated group \(P\)
- control group \(S\)

and two time points:

- time 1: before treatment
- time 2: after treatment

Only group \(P\) receives treatment at time 2.

A regression formulation is

$$
y_{it}
=
a
+b\cdot\text{treated}_i
+c\cdot\text{time2}_t
+d\cdot
(\text{treated}_i\cdot\text{time2}_t)
+\text{error}_{it}.
$$

Interpretation:

- \(a\): baseline outcome for control group at time 1
- \(b\): baseline difference between treated and control groups
- \(c\): general time effect
- \(d\): additional change for the treated group

Thus:

$$
\boxed{d=\text{difference-in-differences treatment effect}}
$$

under the assumptions of the design.

---

# 26. Why regression is useful for descriptive analysis

Compared with manually comparing group means, regression provides several advantages:

- handles multiple predictors simultaneously
- helps account for correlations among predictors
- quantifies uncertainty
- handles interactions
- supports transformed variables
- provides diagnostics such as residuals and \(R^2\)

---

# 27. Important caveat: model specification matters

Regression can produce misleading results if the model is badly specified.

Always ask:

- Are the relevant predictors included?
- Is the functional form appropriate?
- Are interactions needed?
- Are transformations needed?
- Are the residual assumptions reasonable?
- Does the model generalize?
- Does the interpretation make substantive sense?

The lecture recommends staying critical and using diagnostics such as:

- \(R^2\)
- residual inspection
- data visualization

---

# Practical interpretation guide

## Binary predictor

$$
y=\beta_1+\beta_2X
$$

with \(X\in\{0,1\}\):

- \(\beta_1\): mean for group \(X=0\)
- \(\beta_2\): difference in group means

## Continuous predictor

$$
y=\beta_1+\beta_2X
$$

- \(\beta_1\): estimated mean outcome at \(X=0\)
- \(\beta_2\): estimated change in mean outcome for +1 in \(X\)

## Multiple predictors

$$
y
=
\beta_1
+
\beta_2X_2
+
\beta_3X_3
$$

Each coefficient describes an association while holding the other included predictors fixed.

## Interaction

$$
y
=
\beta_1
+
\beta_2X_2
+
\beta_3X_3
+
\beta_4X_2X_3
$$

Now the effect of one predictor depends on the other.

## Centered predictor

$$
X_c=X-\bar X
$$

Then \(X_c=0\) means \(X\) is at its mean.

## Standardized predictor

$$
Z=
\frac{X-\bar X}{s_X}
$$

Then coefficients are expressed per one-standard-deviation change.

## Log outcome

$$
\log y=a+bX
$$

Then +1 in \(X\) corresponds approximately to a \(100b\%\) change in \(y\) when \(b\) is small.

## Log-log model

$$
\log y=a+b\log X
$$

Then a 1% increase in \(X\) corresponds approximately to a \(b\)% increase in \(y\).

---

# Key takeaways

1. **Linear regression can be used for descriptive comparison, not only prediction.**
2. **Least squares minimizes the sum of squared residuals.**
3. **With a binary predictor, regression coefficients directly encode group means and mean differences.**
4. **With continuous predictors, slopes describe changes in estimated mean outcomes.**
5. **Multiple regression helps disentangle associations among correlated predictors.**
6. **Interaction terms allow effects to depend on other predictors.**
7. **Regression adjustment does not automatically imply causality.**
8. **Residuals measure observation-level prediction errors.**
9. **\(R^2\) measures the fraction of outcome variance explained by the model.**
10. **A good \(R^2\) does not guarantee a good model.**
11. **Model specification and assumptions matter.**
12. **Linearity is required in the predictors used by the model, not necessarily in raw inputs.**
13. **Mean-centering improves interpretation of intercepts and main effects.**
14. **Standardization converts predictors to common standard-deviation units.**
15. **Logarithmic outcomes turn additive linear models into multiplicative models.**
16. **Generalized linear models extend regression to binary and count outcomes.**
17. **Difference-in-differences can be represented using an interaction term.**
18. **Always inspect diagnostics and interpret coefficients in context.**

---

# Compact mental model

```text
DATA
  ↓
Choose outcome y and predictors X
  ↓
Fit linear model by least squares
  ↓
Interpret coefficients
  ├── binary → group differences
  ├── continuous → slopes
  ├── multiple predictors → conditional associations
  └── interactions → effects depend on other predictors
  ↓
Quantify uncertainty
  ↓
Inspect residuals and R²
  ↓
Transform / center / standardize if useful
  ↓
Check assumptions and model specification
  ↓
Interpret cautiously
```

The central message of the lecture can be summarized as:

> **Regression is a flexible framework for comparing average outcomes while accounting for multiple predictors, but meaningful interpretation depends on choosing the right model, understanding its coefficients, and checking whether its assumptions and specification make sense.**
