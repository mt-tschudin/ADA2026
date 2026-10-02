# ADA — Lecture 4: Describing Data

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Lecture:** 4 — Describing Data  
**Date:** 30 September 2026

> These notes summarize the key concepts from the lecture slides. The lecture is organized around three main parts: **descriptive statistics**, **quantifying uncertainty**, and **relating two variables**.

***
A small p-value says:
“My result would be unusual if \(H_0\) were true.”

It does not say:
“\(H_0\) itself has a small probability of being true.”

That distinction is the core of p-values.
***

---

# Part 1 — Descriptive statistics

## 1. Why descriptive statistics matter

Statistics is a key ingredient of data analysis. The lecture focuses on statistical summaries, choosing summaries appropriate for the distribution, common pitfalls, and understanding the data before applying models.

> **Understand the distribution of your data before applying any model.**

---

## 2. Means: micro-average vs. macro-average

When data is divided into groups, there are two different ways to compute an average.

### Micro-average

The **micro-average** averages over all individual observations directly:

$$
\bar{x}_{\text{micro}}
=
\frac{\sum_j \sum_{i=1}^{n_j} x_{ij}}
{\sum_j n_j}
$$

Larger groups contribute more because every observation is weighted equally.

### Macro-average

The **macro-average** first computes one mean per group and then averages the group means:

$$
\bar{x}_{\text{macro}}
=
\frac{1}{G}
\sum_{j=1}^{G}
\bar{x}_j
$$

Each group receives equal weight, regardless of its size.

### Key point

Micro- and macro-averages answer different questions:

- **Micro-average:** weight individuals equally.
- **Macro-average:** weight groups equally.

---

## 3. Robust statistics

A statistic is **robust** if it is not very sensitive to extreme values.

### Not robust

- minimum
- maximum
- mean
- standard deviation

### Robust

- median
- quartiles
- quantiles

Robust statistics are especially useful for skewed or heavy-tailed data and in the presence of outliers.

---

## 4. Heavy-tailed distributions

Some distributions are strongly influenced by extreme observations.

For a power-law distribution

$$
f(x)=ax^{-k},
$$

very large values are rare, but still much more common than in thin-tailed distributions.

The lecture notes:

- for \(k \le 3\): variance can be infinite
- for \(k \le 2\): mean and variance can both be infinite

Therefore:

> **Do not automatically report arithmetic mean and variance for power-law-distributed data.**

Prefer robust statistics such as:

- median
- quantiles
- geometric mean

---

## 5. Generalized means

A common trick is:

1. transform the data using \(f\)
2. compute the arithmetic mean in the transformed space
3. transform back using \(f^{-1}\)

General form:

$$
f^{-1}
\left(
\frac{1}{n}
\sum_{i=1}^{n}
f(x_i)
\right)
$$

### Arithmetic mean

For

$$
f(x)=x,
\qquad
f^{-1}(x)=x
$$

we obtain

$$
\bar{x}
=
\frac{1}{n}
\sum_{i=1}^{n} x_i
$$

### Geometric mean

For

$$
f(x)=\log x,
\qquad
f^{-1}(x)=\exp(x)
$$

we obtain

$$
\left(
\prod_{i=1}^{n} x_i
\right)^{1/n}
$$

### Harmonic mean

For

$$
f(x)=\frac{1}{x},
\qquad
f^{-1}(x)=\frac{1}{x}
$$

we obtain

$$
H=
\frac{n}
{\sum_{i=1}^{n} 1/x_i}
$$

### Root mean square

For

$$
f(x)=x^2,
\qquad
f^{-1}(x)=\sqrt{x}
$$

we obtain

$$
RMS=
\sqrt{
\frac{1}{n}
\sum_{i=1}^{n} x_i^2
}
$$

---

## 6. Important distributions

The lecture highlights several important distributions.

### Normal distribution
A common model for continuous, approximately symmetric data.

### Poisson distribution
Models counts of independent events occurring at a certain rate.

Example:
- number of visits to a website in a fixed time interval

### Exponential distribution
Models the interval between such events.

### Binomial / multinomial distribution
Models the number of successes or outcomes over repeated trials.

Example:
- number of heads in \(n\) coin flips

### Power-law / Zipf / Pareto / Yule
Examples:
- term frequencies in documents
- city sizes

---

## 7. Identifying a distribution

Useful visual tools include:

- box plots
- histograms
- smoothed histograms
- quantile-quantile (QQ) plots

If a large-sample histogram or box plot remains strongly asymmetric, the data cannot reasonably be described as Normal.

Statistical tools include:

- goodness-of-fit tests
- Kolmogorov-Smirnov test
- normality tests

---

## 8. Recognizing a power law

For

$$
F(x)=10000x^{-2},
$$

taking logarithms gives

$$
\log F(x)
=
\log(10000)-2\log x
$$

So an ideal power law appears as a straight line on **log-log axes**.

---

# Part 2 — Quantifying uncertainty

## 9. Why uncertainty matters

Finite samples introduce uncertainty.

The lecture emphasizes:

> **Whenever you report a statistic, quantify how certain you are in it.**

Two main approaches are discussed:

1. **Hypothesis testing**
2. **Confidence intervals**

---

## 10. Means of binary responses

If a variable is binary:

- \(0\) = no
- \(1\) = yes

then its arithmetic mean equals the fraction of observations equal to \(1\).

For example, the mean response in each group directly gives the proportion of people in that group who like Snickers.

But group means alone do not communicate uncertainty.

---

## 11. Standard deviation vs. uncertainty of the mean

The **standard deviation** describes the spread of individual observations.

The **standard error of the mean** describes uncertainty in the estimated mean:

$$
SE=
\frac{\text{std}}{\sqrt{n}}
$$

where:

- \(\text{std}\) = standard deviation
- \(n\) = sample size

These quantities should not be confused.

---

# Hypothesis testing

## 12. Basic logic

Hypothesis testing compares:

- a **null hypothesis** \(H_0\)
- an **alternative hypothesis** \(H_A\)

The idea is to gain weak and indirect support for \(H_A\) by showing that the data are unlikely under \(H_0\).

A **test statistic** should tend to be:

- small under \(H_0\)
- large under \(H_A\)

---

## 13. Coin example

Suppose a coin is flipped 100 times and 40 heads are observed.

The hypotheses are:

$$
H_0:p=0.5
$$

$$
H_A:p\ne 0.5
$$

A possible test statistic is:

$$
s=
|50-\text{number of observed heads}|
$$

The question is:

> If the coin were fair, how likely is a result at least this extreme?

Here, that means outcomes such as:

- 40 heads or fewer
- 60 heads or more

---

## 14. p-value

The lecture defines the p-value as

$$
\Pr(S\ge s\mid H_0)
$$

where:

- \(S\) is the random test statistic under the null
- \(s\) is the observed value

So the p-value is:

> **The probability of observing a result at least as extreme as the one seen, assuming the null hypothesis is true.**

A p-value is **not**:

- the probability that \(H_0\) is true
- the probability that the alternative is true
- a measure of practical importance

---

## 15. Significance level

Choose a significance level \(\alpha\).

Common choices include:

- \(5\%\)
- \(1\%\)
- \(0.5\%\)
- \(0.1\%\)

Decision rule:

$$
\text{reject } H_0
\quad \text{if} \quad
p<\alpha
$$

The significance level controls the false-rejection rate under \(H_0\).

A larger \(\alpha\) means a higher probability of rejecting a true null hypothesis.

---

## 16. Selecting the right test

The appropriate test depends on:

- the question being asked
- data type
- dimensionality
- number of outcomes
- sample size
- paired vs. independent samples
- assumptions about the null distribution

The lecture's warning is clear:

> **Never use a statistical test without understanding exactly what it is doing.**

---

## 17. Interpreting p-values carefully

Important points:

- p-values are widely used
- p-values are widely misunderstood
- a large p-value means the data would be quite plausible under the null
- a large p-value says nothing directly about the alternative
- a low p-value suggests the simple null does not explain the data well
- statistical significance is not the same as practical importance

Always inspect the **effect size**, not just the p-value.

---

## 18. Bayes factors

The lecture briefly introduces Bayes factors as an alternative approach.

Conceptually:

$$
\frac{
\Pr(\text{Data}\mid H_0)
}{
\Pr(\text{Data}\mid H_A)
}
$$

This compares how well the observed data are explained under the two competing hypotheses.

---

# Confidence intervals

## 19. Confidence interval: idea

A **confidence interval (CI)** is a range of plausible values for a parameter based on the observed data and the chosen confidence procedure.

A common choice is:

$$
\gamma = 95\%
$$

giving a **95% confidence interval**.

---

## 20. Relationship between CIs and hypothesis tests

Let:

- \(\mu\) = true parameter value
- \(m\) = empirical estimate

The lecture states that a \(\gamma\)-level CI contains the values \(\mu_0\) for which

$$
H_0:\mu=\mu_0
$$

cannot be rejected at significance level

$$
1-\gamma
$$

Thus, confidence intervals and hypothesis tests are tightly connected.

---

## 21. Parametric vs. non-parametric CIs

### Parametric methods

Assume the test statistic follows a known distribution, often approximately Normal.

These assumptions must be checked.

### Non-parametric methods

Make no assumption about the distribution of the test statistic and instead use resampling from the empirical data.

The main example is the **bootstrap**.

---

## 22. Bootstrap resampling

Bootstrap procedure:

1. Start with the observed sample.
2. Draw a new sample of the same size **with replacement**.
3. Compute the statistic of interest.
4. Repeat many times.
5. Use the resulting bootstrap distribution to estimate uncertainty.

If bootstrap estimates are

$$
m_1,m_2,\dots,m_N,
$$

their empirical distribution approximates the sampling distribution of the statistic.

For a 95% interval, the outer tails commonly correspond to about:

$$
2.5\%
\quad \text{and} \quad
97.5\%
$$

---

## 23. Error bars

Error bars may represent different quantities:

- confidence intervals
- standard deviation
- standard error of the mean

Therefore:

> **Always state explicitly what the error bars represent.**

---

## 24. Multiple hypothesis testing

Running many hypothesis tests increases the probability of false positives.

For one test:

$$
P(\text{false positive})=\alpha
$$

so:

$$
P(\text{no false positive})=1-\alpha
$$

For \(k\) independent tests:

$$
P(\text{no false positive in any test})
=
(1-\alpha)^k
$$

Therefore:

$$
P(\text{at least one false positive})
=
1-(1-\alpha)^k
$$

This is the **family-wise error rate**.

---

## 25. Bonferroni correction

The Bonferroni correction uses

$$
\alpha_c=
\frac{\alpha}{k}
$$

where:

- \(\alpha\) = desired family-wise significance level
- \(k\) = number of tests
- \(\alpha_c\) = corrected threshold for each test

It is simple but conservative.

---

## 26. Šidák correction

Assuming independence:

$$
\alpha
=
1-(1-\alpha_c)^k
$$

Therefore:

$$
\alpha_c
=
1-(1-\alpha)^{1/k}
$$

---

# Part 3 — Relating two variables

## 27. Pearson correlation coefficient

Pearson's correlation coefficient measures the amount of **linear dependence** between two variables.

Its range is:

$$
-1\le r\le 1
$$

Interpretation:

- \(r=1\): perfect positive linear relationship
- \(r=0\): no linear relationship
- \(r=-1\): perfect negative linear relationship

A common expression is:

$$
r=
\frac{
\operatorname{cov}(X,Y)
}{
\sigma_X\sigma_Y
}
$$

---

## 28. Beyond Pearson correlation

More general dependence measures include:

### Spearman rank correlation
Measures monotonic association using ranks.

### Mutual information
Captures more general forms of dependence, including nonlinear relationships.

---

## 29. Correlation is not causation

A strong correlation does not prove causality.

Possible explanations include:

- reverse causality
- confounding
- common causes
- coincidence
- selection effects

---

## 30. Anscombe's quartet

Anscombe's quartet contains four datasets with almost identical basic summary statistics, such as:

- means
- variances
- correlations
- linear regression lines

Yet their visual structures are very different.

The lesson is:

> **Look at the data graphically before relying on summary statistics.**

Simple numerical summaries can hide:

- nonlinear patterns
- outliers
- clustering
- structural differences

---

## 31. Simpson's paradox

Simpson's paradox occurs when:

> **A trend appears within groups but disappears or reverses after the groups are combined.**

The lecture uses the 1973 UC Berkeley admissions example.

The aggregate rates suggested one story, but women tended to apply more often to more competitive departments with lower admission rates.

### Key lesson

> **Beware of aggregates.**

Always check whether subgroup composition or another variable is affecting the apparent relationship.

---

# Practical checklist

Before reporting statistics, ask:

### Distribution
- What does the distribution look like?
- Is it symmetric or skewed?
- Is it heavy-tailed?
- Are there outliers?
- Is a Normal model plausible?

### Summary statistics
- Is the mean meaningful?
- Would median or quantiles be more robust?
- Am I using a micro-average or a macro-average?
- Would another generalized mean be more appropriate?

### Uncertainty
- What is the sample size?
- How uncertain is the estimate?
- Should I report a confidence interval?
- What do the error bars represent?

### Hypothesis testing
- What exactly are \(H_0\) and \(H_A\)?
- What test statistic am I using?
- What assumptions does the test make?
- What does the p-value actually mean?
- Is the effect size meaningful?

### Multiple testing
- How many hypotheses am I testing?
- Do I need a multiple-testing correction?

### Relationships
- Is the relationship linear?
- Am I relying too heavily on Pearson correlation?
- Could there be confounding?
- Have I visualized the data?
- Could Simpson's paradox be present?

---

# Key takeaways

1. **Understand the distribution before choosing statistics or models.**
2. **Choose descriptive statistics that match the distribution.**
3. **Median and quantiles are more robust than mean and standard deviation.**
4. **Heavy-tailed data can make arithmetic mean and variance misleading or undefined.**
5. **Micro- and macro-averages answer different questions.**
6. **Generalized means arise by transforming, averaging, and transforming back.**
7. **Always quantify uncertainty when reporting statistics.**
8. **A p-value is defined under the assumption that the null hypothesis is true.**
9. **A p-value is not the probability that the null hypothesis is true.**
10. **Statistical significance is not the same as practical importance.**
11. **Confidence intervals are a useful way to express uncertainty.**
12. **Bootstrap resampling provides a non-parametric way to estimate confidence intervals.**
13. **Always state what error bars represent.**
14. **Multiple testing increases false positives and requires correction.**
15. **Pearson correlation measures linear dependence only.**
16. **Correlation does not imply causation.**
17. **Anscombe's quartet shows why visualization is essential.**
18. **Simpson's paradox shows why aggregated statistics can be misleading.**

---

# Compact mental model

```text
RAW DATA
   ↓
Inspect the distribution
   ↓
Choose appropriate descriptive statistics
   ↓
Quantify uncertainty
   ├── hypothesis testing
   └── confidence intervals / bootstrap
   ↓
Correct for multiple testing if needed
   ↓
Study relationships between variables
   ↓
Visualize before trusting summary statistics
   ↓
Check for confounding / aggregation effects
```

The central lesson of the lecture can be summarized as:

> **Describing data well requires more than computing averages: choose statistics that fit the distribution, quantify uncertainty, and inspect how variables and subgroups actually behave.**
