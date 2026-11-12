# ADA — Lecture 8: Applied Machine Learning

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Date:** 4 November 2026  
**Source:** *08 — Learning from Data: Applied Machine Learning* (lecture slides)

> **Central idea:** A useful ML system involves much more than choosing an algorithm. Good results depend on collecting and preparing appropriate data, selecting models without overfitting, and evaluating them using metrics that match the real task.

The lecture follows a **classification pipeline**:

```text
Data collection & preparation → Model selection → Model assessment
```

---

## 1. Data collection: features and labels

In supervised learning, observations have **features** (inputs $X$) and **labels** (targets $y$). Before training, we must decide what to measure and what we want to predict. Domain knowledge matters.

Common feature types:

| Type | Example | Meaning |
|---|---|---|
| **Continuous** | Temperature, revenue | Numerical values |
| **Ordinal** | Low / medium / high | Categories with an order |
| **Categorical** | Country, occupation | Categories without a necessary order |

**Feature engineering** creates informative predictors from raw data. In classical ML, much of the work is choosing useful features; in deep learning, feature representations and the predictive model can be learned together.

**Labels can be the bottleneck.** They may require human experts, crowdsourcing, or AI-assisted labeling. For example, judging whether a health-information website is credible may require expertise even when the page text is easy to collect.

---

## 2. Discretization: turning numbers into bins

**Discretization** converts a continuous feature into discrete intervals. This may be useful for classifiers expecting categorical features or for constructing nonlinear decision boundaries from discretized inputs.

Two broad approaches:

- **Unsupervised:** choose boundaries without using the target label. *Equal-width* bins split the numerical range into intervals of equal size; *equal-frequency* bins aim to contain similar numbers of samples. Equal-width bins can be poor for strongly skewed distributions.
- **Supervised:** use the label to choose meaningful boundaries. The lecture describes starting with fine bins and testing whether adjacent bins have different class distributions using a **chi-squared ($\chi^2$) independence test**. If their distributions are not distinguishable, the bins can be merged.

**Key distinction:** unsupervised discretization uses only $X$; supervised discretization also uses $y$.

---

## 3. Feature selection: which predictors should we keep?

More features are **not automatically better**. Removing irrelevant features can improve interpretability, reduce the risk of overfitting, and speed up training.

With $N$ candidate features, there are $2^N$ possible subsets, so exhaustive feature-subset search is expensive.

### Filter / offline feature selection

Rank features by their individual relationship with the label, then keep the best ones.

- **Pearson correlation:** measures *linear* association between continuous variables.
- **Mutual information:** measures statistical dependence, including relationships beyond linear correlation.
- **Chi-squared test:** tests whether a categorical feature and label are independent; a p-value is evidence against independence, **not a measure of effect size**.

**Limitation:** two features might be useful *together* even if neither appears useful alone (as illustrated by the two-feature class pattern on slide 24). Simple individual rankings can miss feature interactions.

### Wrapper / online feature selection

Evaluate feature subsets using the actual classifier and validation performance:

- **Forward selection:** start with no features; greedily add features until performance no longer improves.
- **Backward selection (ablation):** start with all features; greedily remove features until performance no longer improves.

These methods capture interactions with the classifier but are more computationally demanding.

---

## 4. Feature scaling and standardization

If one numerical feature takes values in the millions and another in the hundreds, some classifiers can be dominated by scale differences.

The lecture presents three common transformations:

| Method | Formula | Main use |
|---|---|---|
| **Log scaling** | $x'=\log(x)$ | Compress positive, heavily skewed / heavy-tailed features |
| **Min–max scaling** | $x'=\dfrac{x-x_{\min}}{x_{\max}-x_{\min}}$ | Map training-range values to $[0,1]$ |
| **Standardization** | $z=\dfrac{x-\mu}{\sigma}$ | Obtain mean $0$ and standard deviation $1$ |

Here $\mu$ and $\sigma$ are the feature mean and standard deviation.

**Limitations:**

- With **outliers**, min–max scaling compresses most normal observations into a narrow interval.
- For **heavy-tailed features**, the mean and standard deviation may be poor summaries; the lecture recommends considering log scaling.
- Log scaling requires care with zero or negative observations.

> **Practical implementation note (beyond the slides):** When evaluating a model, fit scaling parameters and feature-selection decisions on training data only, then apply them to validation/test data. Otherwise, information can leak into the evaluation.

---

## 5. Model selection: algorithms and hyperparameters

**High-level selection** chooses the model family: k-NN, decision tree, random forest, boosted trees, logistic regression, deep learning, etc.

**Low-level selection** tunes **hyperparameters**, such as:

- number of neighbors $k$ in k-NN;
- number of trees in a random forest;
- regularization strength;
- learning rate;
- classification threshold.

We need a **loss function** to measure prediction errors.

For classification, the lecture gives the **0–1 loss** (error rate):

$$
J_{0/1}=\frac{1}{n}\sum_{i=1}^{n}\mathbf{1}\left[\hat y_i\neq y_i\right]
=1-\text{accuracy}.
$$

For regression, examples include:

$$
\text{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2,
\qquad
\text{MAE}=\frac{1}{n}\sum_{i=1}^{n}|y_i-\hat y_i|.
$$

MSE emphasizes large errors more strongly than MAE.

---

## 6. Training, validation, and test sets

These datasets have **different jobs**:

| Set | Used for |
|---|---|
| **Training** | Fit model parameters |
| **Validation** | Compare models and tune hyperparameters |
| **Test** | Estimate performance of the *final, selected* model on unseen data |

**Why not select the model on the test set?** Because repeatedly choosing what works best on test data would make that set part of model selection, yielding an overly optimistic estimate of generalization.

### $m$-fold cross-validation

Especially useful when a separate validation set would be too small:

1. Split the development/training data into $m$ folds.
2. Train on $m-1$ folds; validate on the remaining fold.
3. Repeat until every fold has served as validation once.
4. Average the validation errors for each candidate model.
5. Choose the candidate with the lowest average error; evaluate it on a separate held-out test set.

$$
\text{CV error}=\frac{1}{m}\sum_{j=1}^{m}E_j.
$$

**Leave-one-out (LOO)** cross-validation is the special case $m=n$: one observation serves as validation in each round. The lecture illustrates **5-fold CV** as a cheaper alternative.

---

## 7. Model assessment: confusion matrix

For binary classification, first define which class is **positive** (e.g., *fraud*, *cancer*, *channel will become inactive*).

| | **Actual positive** | **Actual negative** |
|---|---:|---:|
| **Predicted positive** | True positive (TP) | False positive (FP) |
| **Predicted negative** | False negative (FN) | True negative (TN) |

Interpretation:

- **FP:** false alarm — model predicts the event, but it does not occur.
- **FN:** missed event — model predicts no event, but it occurs.

Which error is worse depends on the application. In medical screening, for instance, missing a disease can be more costly than issuing a false alarm.

---

## 8. Accuracy, precision, recall, F1

### Accuracy

$$
\text{Accuracy}=\frac{TP+TN}{TP+FP+FN+TN}.
$$

Accuracy is most useful when class frequencies are not too skewed and FP/FN errors have comparable costs.

**Class imbalance trap (lecture example):** if only 10% of transactions are fraudulent, a model that *always* predicts “not fraud” achieves **90% accuracy** while detecting **zero frauds**.

### Precision — reliability of positive predictions

$$
\boxed{\text{Precision}=\frac{TP}{TP+FP}}
$$

> Of everything the model predicted as positive, how many were actually positive?

High precision means **few false alarms** among positive predictions.

### Recall — coverage of actual positives

$$
\boxed{\text{Recall}=\frac{TP}{TP+FN}}
$$

> Of all actual positives, how many did the model find?

High recall means **few missed positive cases**.

### F1 — balance between precision and recall

$$
\boxed{F_1=2\frac{\text{Precision}\cdot\text{Recall}}{\text{Precision}+\text{Recall}}}
$$

F1 is the **harmonic mean**, so a low value of either precision or recall pulls the score down.

**Lecture's cancer-classifier comparison:**

| Metric | Classifier 1 | Classifier 2 |
|---|---:|---:|
| Precision | $45/65\approx0.69$ | $40/50=0.80$ |
| Recall | $45/50=0.90$ | $40/50=0.80$ |
| F1 | $\approx0.78$ | $0.80$ |

Classifier 1 finds more actual cases (higher recall), while classifier 2 produces fewer false alarms relative to its positive predictions (higher precision). **The preferred model depends on which errors matter.**

---

## 9. Classification thresholds, PR curves, and ROC curves

A classifier often outputs a **score or probability**. A threshold converts it into a yes/no prediction:

$$
\hat y=\mathbf{1}[\text{score}\ge t].
$$

As the threshold **decreases**, the model typically predicts positive more often: recall increases, but false positives also tend to increase. Precision can decline.

### Precision–recall (PR) curve

Plots **precision vs. recall** as the decision threshold changes. Helps visualize the tradeoff between finding positives and avoiding false alarms.

### Receiver operating characteristic (ROC) curve

Plots:

$$
\text{TPR}=\frac{TP}{TP+FN}=\text{Recall}
$$

against

$$
\text{FPR}=\frac{FP}{FP+TN}.
$$

**ROC AUC** is the area under the ROC curve:

- $0.5$: chance-level ranking;
- $1.0$: perfect ranking;
- higher AUC generally indicates better overall separation across thresholds.

Unlike a single accuracy/F1 value, the curve summarizes behavior over **many thresholds**. However, the final operating threshold should be chosen to reflect the task's consequences for FP and FN.

---

## 10. Learning curves: bias, variance, and when more data helps

Recall from Lecture 7:

- **High bias / underfitting:** model is too simple to capture the relationship.
- **High variance / overfitting:** model fits the training data well but generalizes poorly.

The lecture recommends comparing **training and validation error as training-set size grows** (learning curves).

| Pattern | Likely problem | Typical response |
|---|---|---|
| Both errors relatively high and close | **High bias** | Consider a more flexible model or better predictors |
| Training error low, validation error substantially higher | **High variance** | More data, regularization, or less model complexity |

The key message of slide 56:

> **More data tends to help high-variance models, but does not automatically fix high bias.**

If the model is fundamentally too simple, gathering more examples may not solve its structural limitations.

---

## 11. Final model assessment

After selecting the model and its hyperparameters:

1. Fix the final modeling procedure.
2. Evaluate on the **held-out test set**, not previously used for tuning.
3. Report metrics appropriate for the problem (not just accuracy).
4. Judge whether the test conditions resemble the real deployment setting.

This is **model assessment**, distinct from **model selection**.

---

## Exam / revision checklist

You should be able to explain:

- Why the ML pipeline begins with careful feature and label definition rather than algorithm selection.
- **Equal-width vs. equal-frequency** discretization, and **unsupervised vs. supervised** binning.
- Why a feature can be uninformative individually but useful in interaction with another feature.
- **Filter vs. forward/backward** feature selection and their tradeoffs.
- **Log, min–max, and z-score** transformations, including their limitations.
- Why **training, validation, and test** must have separate roles.
- How **$m$-fold CV** chooses hyperparameters without using the test set.
- How to read a **confusion matrix** and calculate accuracy, precision, recall, and F1.
- Why **high accuracy can be misleading** for imbalanced classes.
- What a threshold change does to the **PR and ROC curves**, and what **ROC AUC** summarizes.
- How **learning curves** distinguish high bias from high variance and indicate whether more data might help.

### One-sentence takeaway

**Good applied ML means choosing the right data representation, selecting the model with sound validation, and reporting performance in terms that reflect the real problem—not just maximizing a convenient number.**
