# ADA — Lecture 09: Unsupervised Learning

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Date:** 11 November 2026  
**Source:** *Learning from data: Unsupervised learning* (51-slide lecture deck)

> **Central idea:** In unsupervised learning, we have observations **without target labels**. We try to discover useful structure in the data. This lecture concentrates on **clustering**: hierarchical methods, k-means, and DBSCAN. Dimensionality reduction is introduced as another unsupervised-learning approach but is the topic of the following lecture.

---

## 1. Supervised vs. unsupervised learning

| | Supervised learning | Unsupervised learning |
|---|---|---|
| Available data | Inputs $X$ **and** known outputs $y$ | Inputs $X$ only |
| Goal | Learn to predict $y$ from $X$ | Discover patterns or a simpler representation of $X$ |
| Examples | Classification, regression | Clustering, dimensionality reduction |

**Clustering** assigns similar data points to groups *without being given their true categories*. For a clustering to be useful, we usually want points **within a cluster** to be similar and points **between clusters** to be dissimilar.

The result depends on how *similarity* is defined: Euclidean, cosine, and Jaccard distances can yield different groupings of the same observations.

### Why use clustering?

- Explore a large or high-dimensional dataset.
- Partition the data for more detailed subsequent analyses.
- Discover customer or content groupings.
- Support labeling or discretization for later supervised learning.

**Example from the lecture:** A movie can be represented by a binary vector with one coordinate per customer ($1$ if the customer watched it, $0$ otherwise). Movies watched by similar users may be treated as similar, even if their published genres differ. Documents can likewise be represented by which words they contain. *(Slides 19–22)*

---

## 2. Clustering is not always the right model

The lecture warns about **cluster bias**: people readily perceive discrete categories even when the underlying structure is a **continuum**. Finding clusters does not prove that nature contains distinct groups.

Clustering is also difficult in high dimensions (**curse of dimensionality**): data become sparse and meaningful nearby neighbors are harder to identify. The chosen features, scales, and distance metric therefore strongly influence the result.

Important distinctions *(slides 15–17)*:

| Distinction | Meaning |
|---|---|
| **Hierarchical** | Clusters are arranged in a nested tree structure. |
| **Flat** | Produces a collection of clusters without a hierarchy. |
| **Hard** | Each point belongs to exactly one cluster. |
| **Soft** | A point has membership weights/probabilities across clusters. |

Two broad approaches covered here are **agglomerative hierarchical clustering** and **flat point-assignment methods** (k-means and DBSCAN).

---

## 3. Agglomerative hierarchical clustering

**Bottom-up procedure** *(slides 23–26)*:

1. Begin with **one cluster per data point**.
2. Find the two **nearest clusters**.
3. Merge them.
4. Repeat until all points are combined, or stop earlier at a useful number of clusters.

The merges form a **dendrogram** (a tree). The vertical height of a merge represents the dissimilarity at which the clusters were combined. Cutting the tree at a chosen height gives a flat partition. Starting from $N$ separate points, a complete binary agglomerative tree contains **$N-1$ merges**.

### How do we represent a cluster?

- **Centroid:** coordinate-wise mean of the points, potentially an *artificial* point not in the dataset.

  $$\mu_C=\frac{1}{|C|}\sum_{x\in C}x$$

- **Clustroid:** an *existing data point* that best represents its cluster under a chosen distance criterion. For example, the **medoid** minimizes the total distance to the other cluster members:

  $$m_C=\underset{x\in C}{\arg\min}\sum_{z\in C}d(x,z)$$

Clustroids are useful when a simple arithmetic mean is not meaningful (e.g., non-Euclidean objects).

### What does “nearest clusters” mean?

It is **not uniquely defined**. The lecture describes choices such as:

- smallest point-to-point distance between clusters;
- average intercluster distance;
- distance between centroids or clustroids;
- cohesion of the merged cluster, such as its diameter (largest internal distance) or average internal distance.

Different definitions can produce different dendrograms, even with identical data.

**Limitation:** Hierarchical clustering can be expensive. The lecture gives $O(N^3)$ for a naive approach and $O(N^2\log N)$ for a careful priority-queue implementation. *(Slide 33)*

---

## 4. K-means: flat, centroid-based clustering

K-means assumes we want **$k$ clusters** and can represent each by a mean (centroid). It tries to make points close to their assigned centroids, usually by minimizing the **sum of squared Euclidean distances**:

$$
J=\sum_{i=1}^{n}\left\|x_i-\mu_{c_i}\right\|_2^2,
$$

where $c_i\in\{1,\ldots,k\}$ is the cluster assigned to point $i$, and $\mu_{c_i}$ is its centroid.

### Lloyd's algorithm *(slides 36–38)*

1. **Initialize** $k$ cluster centers.
2. **Assignment:** assign each point to its nearest centroid.
3. **Update:** replace each centroid with the mean of the points assigned to it.
4. Repeat assignment and update until assignments no longer change, the objective improves very little, or an iteration limit is reached.

```text
Choose k initial centers
       ↓
Assign each point to nearest center
       ↓
Recompute the mean of each cluster
       ↓
Repeat until convergence
```

Each iteration improves or preserves the objective; however, the algorithm may converge to a **local rather than global optimum**.

### Why initialization matters: k-means++

Bad starting centers can produce bad clusters. K-means++ tries to start with **well-spaced** centers *(slides 39–40)*:

1. Choose the first center randomly from the data.
2. For each remaining point $x$, find $D(x)$, its distance to the nearest chosen center.
3. Select the next center randomly, with probability proportional to **$D(x)^2$**.
4. Repeat until $k$ centers are selected, then run ordinary k-means.

This favors points far from existing centers without choosing them deterministically. Multiple restarts can also reduce sensitivity to initialization.

### Strengths and weaknesses

K-means is simple and computationally efficient; the slides give roughly **$O(nk)$ per iteration** when feature dimension is treated as fixed.

But it:
- requires specifying $k$;
- needs meaningful means / Euclidean geometry;
- is sensitive to initialization and outliers;
- favors compact, convex clusters and struggles with curved or irregular shapes.

**Do not confuse:** $k$ in *k-means* is the **number of clusters**, whereas $k$ in *k-nearest neighbors* (Lecture 7) is the **number of neighbors used for prediction**.

---

## 5. Choosing the number of clusters: silhouette score

The lecture proposes evaluating several choices of $k$ using the **silhouette width** *(slide 43)*.

For each point $i$:

- $a(i)$ = average distance from $i$ to other points **in its own cluster**;
- $b(i)$ = **smallest** average distance from $i$ to points in any **other cluster** (the closest alternative cluster).

The silhouette score is

$$
\boxed{s(i)=\frac{b(i)-a(i)}{\max\{a(i),b(i)\}}}
$$

Interpretation:

- **Near $+1$:** point fits its cluster well; other clusters are far away.
- **Near $0$:** point lies near a boundary between clusters.
- **Negative:** point might fit another cluster better.

Take the mean across points:

$$S=\frac{1}{n}\sum_{i=1}^{n}s(i).$$

**Procedure:** Run k-means for different $k$, compute $S$, and prefer a value with a high average silhouette. Silhouette comparisons require at least two clusters and should not override domain knowledge: a high score does not guarantee scientifically meaningful categories.

---

## 6. DBSCAN: density-based clustering

**DBSCAN** (*Density-Based Spatial Clustering of Applications with Noise*) is motivated by the fact that k-means cannot easily follow curved or irregular groups. DBSCAN instead identifies **connected regions of sufficiently dense points** and labels isolated points as **noise/outliers**. *(Slides 44–48)*

Its two main parameters are:

- $\varepsilon$ (epsilon): the neighborhood distance scale (the lecture discusses an $\varepsilon$-sphere);
- **minPts:** the minimum number of nearby points required to qualify as a dense/core point, using the lecture's neighborhood-counting convention.

### Three kinds of points

- **Core point:** has enough neighbors inside its $\varepsilon$-neighborhood.
- **Border point:** lies within reach of a core region but does not itself have enough neighbors to be a core point.
- **Noise point:** is not density-reachable from a cluster's core points.

### Reachability and clusters

- **Direct density reachability:** a core point can directly reach points within its $\varepsilon$-neighborhood; non-core points do not expand a cluster further.
- **Density reachability:** a point can be reached through a chain of steps, where each step is directly reachable from a core point.
- **Density connectivity:** two points are connected if both can be reached from some common point.

A DBSCAN cluster contains a density-connected region, including reachable border points. In the lecture's diagram, red points form the dense core region, **B and C** are connected border points, and **N** is noise. *(Slides 45–47)*

### Algorithm intuition

```text
Pick an unvisited point
       ↓
Find neighbors within ε
       ↓
Enough nearby points?
   ├── No → mark as possible noise
   └── Yes → start/expand a cluster
               through reachable points
       ↓
Repeat for remaining unvisited points
```

**Advantages:** Finds non-convex/irregular clusters; explicitly identifies noise; does not require specifying $k$ in advance.

**Limitations:** Results depend on $\varepsilon$ and minPts; clusters with very different densities can be difficult to capture with a single setting. Spatial indexing can make DBSCAN efficient (the lecture gives **$O(n\log n)$** under favorable indexing assumptions), but neighbor search becomes harder in high dimensions. *(Slide 49)*

---

## 7. Comparing the three methods

| Method | Main principle | Main choices | Especially useful when… | Main limitation |
|---|---|---|---|---|
| **Hierarchical (agglomerative)** | Repeatedly merge nearest clusters | Distance/linkage, cut level | You want nested groupings or a dendrogram | Expensive on large datasets |
| **K-means** | Assign points to their nearest mean | $k$, initialization | Groups are compact and roughly convex | Outliers, irregular shapes, local optima |
| **DBSCAN** | Grow connected dense regions | $\varepsilon$, minPts | Shapes are irregular or noise matters | Sensitive to density parameters; variable densities |

There is **no universally best clustering algorithm**. The quality of the representation, similarity measure, and assumptions about the data matter as much as the optimization procedure.

---

## 8. Study checklist / typical exam questions

**1. What distinguishes supervised from unsupervised learning?**  
Whether target labels are provided during training. Clustering discovers groups; classification predicts known categories.

**2. Centroid vs. clustroid?**  
A centroid is the average (possibly not an observed point). A clustroid is an actual member chosen to represent the cluster.

**3. Why might two hierarchical clusterings differ?**  
Their definitions of intercluster distance/cohesion may differ.

**4. How does k-means work?**  
Alternate between assigning points to nearest centroids and recalculating means; minimize within-cluster squared distance.

**5. What is k-means++ for?**  
Selecting dispersed initial centers to reduce poor local solutions.

**6. How do you evaluate $k$?**  
Use the mean silhouette score, interpreting both within-cluster and between-cluster distances.

**7. What can DBSCAN do that k-means struggles with?**  
Find curved/irregular dense regions and leave outliers unclustered.

**8. Why should we be skeptical of clusters?**  
Discrete-looking groups may be artifacts of the algorithm, feature choice, or distance measure, when the underlying phenomenon is continuous.

---

## Key takeaways

1. **Unsupervised learning discovers structure without target labels.** This lecture focuses on clustering; dimensionality reduction comes next.
2. **Similarity is a modeling decision.** The feature representation and distance metric can fundamentally change the result.
3. **Hierarchical clustering** constructs nested groups by successively merging nearby clusters.
4. **K-means** is an efficient assign/update algorithm that minimizes distance to means, but requires $k$ and favors compact convex clusters.
5. **K-means++** improves initialization; **silhouette scores** provide one way to compare candidate values of $k$.
6. **DBSCAN** finds density-connected clusters and identifies noise, making it useful for irregular shapes.
7. **Cluster bias is real:** a clustering result is an analytical representation, not proof that objectively distinct categories exist.

> **One-sentence summary:** Choose a useful representation and similarity metric, then pick a clustering method whose assumptions match the geometry and purpose of your data—and do not mistake its output for ground truth.
