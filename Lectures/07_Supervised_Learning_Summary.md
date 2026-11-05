# ADA — Lecture 7: Learning from Data — Supervised Learning

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Date:** 28 October 2026  
**Source:** Lecture 7 slides, *Learning from data: Supervised learning* (65 pages)

> **Central idea:** A supervised-learning model learns from labeled examples, but fitting the training data is not enough. We want accurate predictions on **unseen data**, so model complexity, the bias–variance tradeoff, and reliable evaluation are central.

---

## 1. What is supervised learning?

We observe labeled input–output pairs $(X_i,y_i)$ and try to learn a mapping $f$ that predicts $y$ for a new input $X$.

| Learning setting | Available data | Typical task |
|---|---|---|
| **Supervised: classification** | Inputs $X$ and discrete labels $y$ | Spam/not spam, cat/dog |
| **Supervised: regression** | Inputs $X$ and continuous outcomes $y$ | Predict a rating or numerical value |
| **Unsupervised** | Inputs $X$ only | Clustering, dimensionality reduction |

The lecture focuses on three families of supervised models: **k-nearest neighbors (k-NN), tree-based methods, and linear/logistic regression**.

## 2. k-Nearest Neighbors (k-NN)

**Idea:** For a new input, find the $k$ most similar labeled training examples.

- **Classification:** predict the most frequent class among the $k$ neighbors (majority vote).
- **Regression:** predict the average numerical outcome of the $k$ neighbors.
- Votes or averages may be **weighted by similarity**, so closer neighbors contribute more.

For regression, the unweighted rule is:

$$
\hat y(x)=\frac{1}{k}\sum_{i\in\mathcal N_k(x)}y_i,
$$

where $\mathcal N_k(x)$ is the set of $k$ nearest training examples.

k-NN is conceptually simple: **the stored training data are the model**, rather than a set of fitted coefficients. But its performance depends strongly on the choice of $k$, the **distance metric**, and how neighbors are weighted.

### Choosing a distance

The lecture mentions several possibilities:

| Distance | Expression / use |
|---|---|
| **Euclidean** | $d(x,z)=\sqrt{\sum_j(x_j-z_j)^2}$; numerical vectors |
| **Manhattan** | $d(x,z)=\sum_j\lvert x_j-z_j\rvert$; coordinate-wise differences |
| **Cosine** | $d(x,z)=1-\frac{x\cdot z}{\lVert x\rVert\lVert z\rVert}$; vector orientation, often useful for documents |
| **Jaccard** | $d(A,B)=1-\frac{|A\cap B|}{|A\cup B|}$; sets |
| **Hamming** | Number of differing positions; equal-length strings |
| **Edit** | Number/cost of edits needed to transform one string into another |

**Key lesson:** “Nearest” depends on how distance is defined. A model can make poor predictions if its distance does not capture meaningful similarity.

## 3. Generalization and the bias–variance tradeoff

Our dataset $D$ is only a **sample** from a larger population. Training on $D$ produces a fitted model $f_D$. If we trained on another sample, we might obtain different predictions.

For a fixed input, the lecture decomposes expected squared prediction error conceptually into:

$$
\text{Error}^2=\text{Bias}^2+\text{Variance}.
$$

- **Bias:** systematic difference between the model's average prediction and the true target. High bias means the model is too rigid to capture the pattern.
- **Variance:** how much the model's prediction changes when the **training sample changes**. High variance means the model is sensitive to the particular observations it saw.

*Technical note:* The lecture presents the simplified decomposition relative to a fixed true target. When the observed outcome also contains irreducible random noise, the usual expected prediction-error decomposition includes an additional noise term.

### Underfitting versus overfitting

| | Underfitting | Overfitting |
|---|---|---|
| Typical model | Too simple | Too complex |
| Main problem | High bias | High variance |
| Behavior | Misses real patterns | Learns noise and peculiarities of training data |
| Example | Straight line for a complex curve | Highly wiggly polynomial through noisy points |

**Goal:** Find a balance that predicts well on new observations—not simply minimize training error.

### How $k$ changes k-NN

- **Small $k$:** very local predictions, **low bias / high variance**; one unusual neighbor can strongly influence the result.
- **Large $k$:** smoother, more stable predictions, **higher bias / lower variance**; local patterns may be washed out.

Increasing $k$ therefore generally reduces variance but increases bias.

## 4. Choosing hyperparameters: training, cross-validation, test

The lecture demonstrates **leave-one-out (LOO) cross-validation** to select $k$ without evaluating the final model on the same observations used to tune it.

1. **Split** the dataset into training and a held-out **test** set (for example, 80%/20%).
2. **Tune on training data:** for each candidate $k$, leave out one training example at a time, predict it using the remaining training examples, and aggregate its errors.
3. **Select** the $k$ with the lowest LOO error (e.g., classification error or squared error).
4. **Evaluate once** on the held-out test set to estimate performance on unseen data.

**Important distinction:** Cross-validation is for **model/hyperparameter selection**; the test set is for **final evaluation**. Choosing $k$ based on test performance would make that evaluation overly optimistic.

## 5. Decision trees

A **decision tree** predicts by following successive tests on features:

```text
Age?
├── <= 30: Student?
│   ├── No  -> Do not buy
│   └── Yes -> Buy
├── 31–40   -> Buy
└── > 40: Credit rating?
    ├── Excellent -> Do not buy
    └── Fair      -> Buy
```

- **Internal nodes:** tests on attributes.
- **Branches:** possible outcomes of those tests.
- **Leaves:** predicted labels.

The aim is to create subsets whose labels become increasingly **homogeneous (pure)**. Finding the globally optimal tree is computationally difficult, so practical algorithms generally use **greedy top-down splitting**, then control complexity through pruning.

### Entropy and information gain

For a binary classification node with positive-class proportion $p$, its entropy is:

$$
H(p)=-p\log_2p-(1-p)\log_2(1-p).
$$

- $H=0$ for a pure node (all one class).
- $H=1$ when classes are evenly split ($p=0.5$): maximum uncertainty.

A candidate attribute $A$ divides the data into subsets $S_v$. Its weighted entropy after splitting is:

$$
H(S\mid A)=\sum_v\frac{|S_v|}{|S|}H(S_v).
$$

**Information gain** measures how much uncertainty the split removes:

$$
\operatorname{Gain}(A)=H(S)-H(S\mid A).
$$

Choose the attribute with the **largest information gain**. In the lecture's worked example, splitting on **age** yields gain $0.25$, more than student status ($0.16$), credit rating ($0.05$), or income ($0.03$). Other algorithms use **Gini impurity** instead of entropy.

### Pruning and tree complexity

Deep trees can keep splitting until they memorize noisy training data. **Pruning** reduces overfitting:

- **Pre-pruning:** stop splitting when subsets become too small or another stopping condition is met.
- **Post-pruning:** grow a larger tree, then replace branches with simpler leaves if validation performance does not worsen.

As **tree depth increases**, **bias generally decreases** and **variance increases**. Individual decision trees can be sensitive to small changes in the training set.

## 6. Ensemble learning: combining models

Instead of relying on one learner, **ensembles** combine predictions from multiple learners.

| Method | How learners are combined | Main idea |
|---|---|---|
| **Bagging** | Train independently on different data samples; vote/average | Stabilize predictions, reduce variance |
| **Boosting** | Train sequentially, adapting to previous errors | Correct remaining mistakes, often reduce bias |
| **Stacking** | Use a second-stage model to combine base-model predictions | Learn how to combine models |

### Random forests = bagged, randomized decision trees

For a dataset containing $N$ observations and $p$ features:

1. Create $K$ **bootstrap samples**, each containing **$N$ draws with replacement**.
2. Train one decision tree per bootstrap sample.
3. At each split, consider only a randomly chosen subset of $m$ of the $p$ features (the lecture gives $m\approx\sqrt p$ as an example).
4. Combine tree predictions by **majority vote** (classification) or **averaging** (regression).

Why two sources of randomness? Different bootstrap samples and feature subsets make trees **less similar**. Combining diverse trees helps reduce the variance of a single unstable decision tree. Trees can be trained **in parallel**.

> Connection to Lecture 4: a bootstrap sample keeps the **same number of observations as the original dataset**, but can contain duplicates because sampling is with replacement.

### Boosted decision trees

Boosting builds trees **sequentially**. Each new tree is trained to correct errors remaining from previous trees; in the lecture's explanation, later trees model previous residual errors. Final predictions combine the trees' contributions.

The lecture contrasts the typical structures:

- **Random forests:** often tens of relatively deep trees; depth reduces bias, ensemble averaging reduces variance.
- **Boosted trees:** often many shallow trees; their sequential combination reduces remaining error.

Neither is universally better. Results and computational costs depend on the dataset.

## 7. Linear versus logistic regression

**Linear regression** predicts a numerical outcome as a linear combination of features. But for a **binary** outcome, we usually want a valid probability between $0$ and $1$. A straight-line prediction can exceed that range.

**Logistic regression** solves this by modeling **log odds** instead of probability directly.

For event probability $p$:

$$
\text{odds}=\frac{p}{1-p},
\qquad
\log\frac{p}{1-p}=\beta^T X.
$$

Solving for $p$ gives the **sigmoid**:

$$
\boxed{p(X)=\frac{1}{1+e^{-\beta^T X}}}.
$$

The resulting prediction always lies between $0$ and $1$. The lecture describes fitting logistic regression via **maximum likelihood / cross-entropy loss**, rather than ordinary least-squares loss.

**Interpretation:** A coefficient increases the **log odds** by its amount for a one-unit predictor increase (other variables fixed), not the probability by a constant amount.

## 8. A final risk: too many features

More features are **not automatically better**. Increasing flexibility may reduce bias while increasing variance and overfitting.

The lecture briefly mentions:
- **feature selection** (forward/backward selection);
- **regularization** (penalizing large model weights).

The practical message is to evaluate feature choices and model complexity using data the model did **not** fit to.

### Bonus: the curse of dimensionality

In high-dimensional spaces, observations can become sparse, which makes it harder for k-NN to find genuinely close neighbors. This is why distance selection and meaningful representations matter.

The bonus slides illustrate that for independent uniformly random coordinates in an $N$-dimensional unit cube:

$$
\mathbb E\big[\lVert X-Y\rVert_2^2\big]=\frac N6.
$$

Thus characteristic Euclidean distances grow on the order of $\sqrt N$. Real-world high-dimensional data may nevertheless lie near lower-dimensional structures, and suitable similarity measures can still make nearest-neighbor methods useful.

---

## Exam / revision checklist

Make sure you can explain, without looking at formulas:

1. **Supervised vs. unsupervised**, and classification vs. regression.
2. How k-NN predicts, and why the **distance metric** and **$k$** matter.
3. **Bias vs. variance**, and how overfitting differs from underfitting.
4. Why tuning must use **cross-validation** and not the final test set.
5. How a tree chooses a split using **entropy and information gain**, and why pruning helps.
6. Why **random forests** use both bootstrap samples and random feature subsets.
7. Why **boosting** is different from bagging.
8. Why **logistic regression** uses log odds and the sigmoid for binary outcomes.

**One-sentence takeaway:** The most important supervised-learning skill is not merely fitting a model; it is **controlling complexity and evaluating whether the model generalizes beyond the training sample**.
