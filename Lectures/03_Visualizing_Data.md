# ADA — Lecture 3: Visualizing Data

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Lecture:** 3 — Visualizing Data  
**Date:** 23 September 2026

> These notes summarize the key concepts from the lecture slides. The lecture is organized around three main parts: **navigating the chart landscape**, **principles and best practices**, and **use cases for data visualization**.

---

## 1. Why visualize data?

Data visualization has three major roles:

### Analysis
Visualization supports reasoning about information by helping us:
- find relationships
- discover structure
- quantify values and influences
- iterate throughout the data-analysis cycle

Visualization should not be treated only as a final presentation step. It is useful during exploration, modeling, and validation.

### Communication
Visualization can:
- capture attention
- engage an audience
- tell a story visually
- emphasize certain aspects of the data
- omit or de-emphasize others

This means that visualization is not neutral: **design choices affect what viewers notice**.

### Decision-making
Visualization can make it easier to compare alternatives and evaluate possible courses of action.

---

## 2. Static vs. interactive visualization

### Static visualization
Static plots are especially useful for:
- data exploration
- papers
- reports
- presentations

### Interactive visualization
Interactive visualization is increasingly common for:
- presenting results
- exploratory analysis
- journalism
- dashboards
- public communication

Modern visualization frameworks make interaction much easier to implement.

---

# Part 1 — Navigating the chart landscape

## 3. Choosing the right chart

Chart choice should depend on:
- the number of variables
- whether variables are categorical or continuous
- the relationship you want to show
- whether the task is comparison, distribution, composition, or relationship analysis

> **Choose the visualization based on the question, not based on which plot looks nicest.**

---

## 4. One variable: histograms

Histograms are useful for understanding the distribution of a single variable.

They can reveal:
- center
- spread
- skewness
- multimodality
- outliers

A smoothed version of a histogram is a **kernel density estimate (KDE)**.

If the data values are

$$
x_1, x_2, \dots, x_n,
$$

then the histogram counts how many values fall into each interval.

---

## 5. One variable: box plots

A box plot summarizes a distribution using:
- lower extreme
- lower quartile $Q_1$
- median
- upper quartile $Q_3$
- upper extreme
- possible outliers

The interquartile range is

$$
IQR = Q_3 - Q_1
$$

Box plots are especially useful for comparing several distributions compactly.

---

## 6. Two variables: scatter plots

Scatter plots are useful for exploring the relationship between two variables.

They can reveal:
- positive association
- negative association
- nonlinear relationships
- clusters
- outliers
- dense regions

For many overlapping points, a **2D histogram / heatmap** can encode local density by color.

---

## 7. Two variables: line plots

Line plots are most appropriate when the relation between variables is functional or ordered.

Typical examples:
- time series
- values after binning
- aggregated measurements

A line suggests continuity between neighboring x-values, so that interpretation should make sense.

---

## 8. More than two variables: scatter plot matrices

A scatter plot matrix shows pairwise relationships among many variables.

It can help reveal:
- correlations
- nonlinear relationships
- clusters
- outliers

---

## 9. More than two variables: stacked plots

Stacked plots can encode several variables at once, for example:
- x-position or stack index
- height as a continuous value
- color as a category

They are useful for composition, but exact comparisons can become difficult.

---

## 10. Dimensionality reduction

High-dimensional continuous data cannot be directly visualized in 2D.

A common strategy is dimensionality reduction.

### PCA

Principal Component Analysis (PCA) projects high-dimensional data into a lower-dimensional space.

The principal components:
- capture directions of high variance
- are mutually orthogonal

Conceptually,

$$
\mathbf{x} \in \mathbb{R}^d
$$

is transformed into a lower-dimensional representation such as

$$
\mathbf{z} \in \mathbb{R}^2
$$

for visualization.

The first principal component captures the largest possible variance; the second captures the largest remaining variance while being orthogonal to the first.

---

## 11. One dataset can be visualized many ways

A dataset does not have one uniquely correct visualization.

Different plots emphasize different aspects.

> A visualization should help the data focus on the specific point you want to investigate or communicate.

---

# Part 2 — Principles and best practices

## 12. Human perception matters

Good visualizations are designed around how humans perceive differences.

Not all graphical encodings are perceived equally accurately.

---

## 13. Just Noticeable Difference (JND)

The lecture introduces **Weber's law**:

$$
\frac{\Delta I}{I} = k
$$

where:
- $I$ = original stimulus intensity
- $\Delta I$ = minimum increase needed to notice a difference
- $k$ = constant

Therefore,

$$
\Delta I = kI
$$

The absolute change required to notice a difference increases with the original intensity.

This means perception often behaves approximately in **relative / multiplicative** rather than absolute steps.

---

## 14. Perception of graphical magnitudes

Approximate ordering from most accurate to least accurate:

1. **Position**
2. **Length**
3. **Slope**
4. **Angle**
5. **Area**
6. **Volume**
7. **Color hue / saturation / density**

### Practical consequence

If precise comparison matters, prefer:
- position
- length

rather than:
- area
- volume
- color intensity

---

## 15. Choose your axes wisely

Axis choice can dramatically change what patterns are visible.

### Linear axes
Useful when:
- absolute differences matter
- values lie in a moderate range

### Logarithmic axes
Useful when:
- values span several orders of magnitude
- relative changes matter
- the distribution is heavy-tailed
- exponential or power-law patterns are present

On a log scale, equal distances correspond to equal ratios rather than equal differences.

---

## 16. Heavy-tailed distributions

A power-law distribution can be written as

$$
p(x) = Cx^{-\alpha}
$$

where:
- $C$ is a normalization constant
- $\alpha$ is the exponent

Very large values are rare, but not extremely rare compared with thin-tailed distributions.

For some small values of $\alpha$, moments such as the mean or variance can diverge, so the median may be more informative.

---

## 17. Power laws on log-log axes

Starting from

$$
y = Cx^{-\alpha},
$$

taking logarithms gives

$$
\log y = \log C - \alpha \log x
$$

which is linear in log-log space.

Thus an ideal power law appears as a straight line on log-log axes.

---

## 18. Complementary cumulative distribution function (CCDF)

The CCDF is defined as

$$
P(x) = \Pr(X \ge x)
$$

For a power law

$$
p(x) = Cx^{-\alpha},
$$

the CCDF is also a power law:

$$
P(x)
=
\int_x^\infty Ct^{-\alpha}\,dt
=
\frac{C}{\alpha-1}x^{-(\alpha-1)}
$$

for $\alpha > 1$.

So the CCDF exponent becomes

$$
\alpha - 1
$$

### Advantages of the CCDF
- monotonically decreasing
- no histogram binning required
- useful for heavy-tailed data

---

## 19. Use consistent axes

Comparing plots with different scales can be misleading.

When comparing plots:
- use consistent axes when possible
- explicitly indicate scale differences
- avoid visually exaggerating differences with incompatible ranges

---

## 20. Label your axes

Every plot should clearly state:
- what the x-axis represents
- what the y-axis represents
- units
- relevant scale information

A viewer should not need to guess what a quantity means.

---

## 21. Logarithmic scales for very different ranges

If two series live on very different numerical scales, separate y-axes can be confusing.

A logarithmic scale can sometimes place both on the same axis when:
- values are positive
- ratios are meaningful
- data spans several orders of magnitude

---

## 22. Show uncertainty

A visualization should communicate uncertainty when it matters.

Common approaches:
- error bars
- confidence intervals
- shaded uncertainty bands

> Do not show only the estimate when uncertainty is an important part of the result.

---

## 23. Small multiples

**Small multiples** show similarly structured plots side by side.

Advantages:
- easier comparison
- less clutter
- consistent visual grammar
- useful subgroup analysis

They should generally use:
- consistent scales
- consistent colors
- consistent ordering

---

## 24. Use colors consistently

If a category is represented by one color in one panel, keep the same color in related panels.

Changing the meaning of colors across plots increases cognitive load and can cause misinterpretation.

---

## 25. Choose colors based on meaning

### Sequential palette
For ordered values from low to high.

Examples:
- density
- probability
- intensity

### Diverging palette
For values around a meaningful midpoint.

Examples:
- deviation from zero
- profit/loss
- anomaly around a reference value

### Categorical palette
For unordered categories.

Examples:
- country
- process type
- experimental group

---

## 26. Use colorblind-safe palettes

Good practice:
- use colorblind-safe palettes
- avoid relying on red vs. green alone
- reinforce differences using markers, line styles, labels, shapes, or position

Accessibility should be part of visualization design.

---

## 27. Data ink and chart junk

The lecture emphasizes using graphical elements efficiently.

### Data ink
Elements that directly represent data.

### Chart junk
Decorative elements that do not improve understanding.

Examples:
- unnecessary 3D effects
- excessive gridlines
- decorative backgrounds
- redundant legends
- unnecessary borders

Goal:

> Maximize useful information and minimize distracting graphical elements.

---

# Part 3 — Selected use cases

## 28. Presenting scientific results

A good scientific figure should:
- show relevant data clearly
- communicate the main finding
- make comparisons easy
- avoid unnecessary decoration
- show uncertainty where appropriate
- use suitable scales and encodings

---

## 29. Data wrangling: detecting multimodal data

Multiple peaks in a histogram can suggest multiple populations.

But:

> Do not guess the cause immediately.

Investigate further using:
- subgroup colors
- separate histograms
- metadata
- group-wise comparisons

Visualization can reveal problems during data wrangling, not only after analysis.

---

## 30. Data wrangling: weird data

Unexpected patterns should not simply be ignored.

Suggested process:
1. maintain a theory of what the data should look like
2. notice unusual patterns
3. first suspect a bug
4. try to identify and fix it
5. if it is not a bug, investigate further

Unexpected data may indicate:
- measurement errors
- processing bugs
- corrupted data
- unexpected populations
- interesting discoveries

---

## 31. Journalism

Data visualization can be used to:
- explain complex topics
- present large datasets
- support interactive exploration
- tell stories with evidence

The lecture highlights New York Times graphics as examples of visualization practice.

---

## 32. Educating the public

Visualization can compress large amounts of information into an understandable narrative.

The lecture uses an example spanning:
- many countries
- long time periods
- several variables

This shows how visual storytelling can make complex datasets accessible.

---

## 33. Giving new perspectives

Charles Joseph Minard's 1869 visualization of Napoleon's march combines several variables:
- army size
- geographic location
- dates
- direction
- temperature during the retreat

It demonstrates how a strong visualization can encode several dimensions while still telling a coherent story.

---

# Practical visualization checklist

Before finalizing a visualization, ask:

### Purpose
- What question am I answering?
- Is this for exploration, communication, or decision-making?

### Chart type
- Does the chart type match the number and type of variables?
- Am I showing comparison, distribution, relationship, or composition?

### Axes
- Are axes labeled?
- Are units shown?
- Are scales consistent?
- Would a log axis reveal the structure more clearly?

### Perception
- Am I using position or length when precise comparison matters?
- Am I relying too much on area, volume, or color intensity?

### Color
- Is the palette sequential, diverging, or categorical for a reason?
- Are colors consistent?
- Is the palette colorblind-safe?

### Uncertainty
- Does the plot communicate uncertainty where necessary?

### Clutter
- Is there unnecessary chart junk?
- Does each graphical element help communicate information?

### Data quality
- Are there unexpected patterns?
- Could they indicate a bug?
- Could they indicate a meaningful discovery?

---

# Key takeaways

1. **Visualization is part of the analysis process, not only the final presentation.**
2. **Chart choice should follow the analytical question and data type.**
3. **Histograms and box plots are useful for one-variable distributions.**
4. **Scatter plots reveal relationships between two variables.**
5. **Scatter plot matrices and dimensionality reduction help with high-dimensional data.**
6. **Human perception is more accurate for position and length than for area, volume, or color.**
7. **Axis choice can radically affect what patterns are visible.**
8. **Logarithmic scales are especially useful for values spanning orders of magnitude and heavy-tailed data.**
9. **Power laws appear linear on log-log axes.**
10. **The CCDF is useful for visualizing heavy-tailed distributions without histogram binning.**
11. **Use consistent and clearly labeled axes.**
12. **Show uncertainty when it matters.**
13. **Use colors consistently, meaningfully, and accessibly.**
14. **Avoid chart junk and unnecessary decoration.**
15. **Visualization is a powerful tool for detecting data-quality problems and unexpected structure.**
16. **Unexpected data should be investigated, not automatically discarded.**
17. **Good visualization can communicate complex, multidimensional information compactly.**

---

# Compact mental model

```text
QUESTION
   ↓
What type of variables do I have?
   ↓
Choose an appropriate chart
   ↓
Check axes and scales
   ↓
Use perceptually effective encodings
   ↓
Use color carefully
   ↓
Show uncertainty
   ↓
Remove unnecessary clutter
   ↓
Inspect for unexpected patterns
   ↓
Interpret / communicate
```

The central lesson of the lecture can be summarized as:

> **A good visualization is not merely attractive: it makes the right structure in the data easy to see, compare, and understand without misleading the viewer.**
