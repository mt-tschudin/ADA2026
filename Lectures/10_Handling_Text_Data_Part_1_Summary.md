# ADA — Lecture 10: Handling Text Data (Part 1)

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Date:** 18 November 2026  
**Source:** *Handling text data*, slides **1–42** of the combined Lectures 10+11 deck.

> **Central idea:** Most machine-learning algorithms need numerical feature vectors, whereas raw text consists of variable-length strings. Lecture 10 shows how to turn a collection of documents into a useful **bag-of-words / TF-IDF matrix**, and why the preprocessing decisions matter.

*Scope: Lecture 10 only. Slide 42 ends Part 1; slide 43 starts the next lecture. The notes below are based on the PDF slides; speaker notes from the Google Slides link were not accessible.*

---

## 1. Why analyze text?

Much useful data comes as **unstructured text**: websites, social media posts, news, search queries, etc. To analyze it quantitatively, we often need to extract a more structured representation.

The lecture introduces **four typical text-analysis tasks**:

| Task | Input / objective | Typical ML formulation |
|---|---|---|
| **Document retrieval** | Given a query, rank documents in a *corpus* by similarity to it | Nearest-neighbor search using document distances |
| **Document classification** | Assign a document to one of several known categories | Supervised classification on labeled examples |
| **Sentiment analysis** | Estimate how positive or negative a text is | Supervised classification or regression |
| **Topic detection** | Discover recurring topics and assign documents to them, without predefined labels | Unsupervised clustering; matrix factorization is mentioned as an alternative for the next lecture |

**Example:** For YouTube-related text, retrieval could find videos relevant to a query; classification could identify categories from descriptions; sentiment analysis could analyze opinions; and topic detection could discover themes not specified in advance.

The key prerequisite for all four tasks is a numerical representation of each document.

---

## 2. From text to feature vectors

A standard ML input is a fixed-length feature vector:

$$
\mathbf{x}_d = (x_{d1},x_{d2},\ldots,x_{dV}).
$$

A raw document is not directly suitable:

- documents have different lengths;
- character positions do not correspond to consistent, meaningful features across documents.

Two approaches introduced on the slides are:

- **Bag of words (BoW):** use counts of vocabulary words as features.
- **Text embeddings:** learn a mapping from text to numerical vectors (introduced only as an alternative here).

Lecture 10 concentrates on the traditional **bag-of-words representation**.

### Bag of words: frequency matters, order does not

Consider:

> `what you see is what you get`

Its bag-of-words representation is:

```text
get:  1
is:   1
see:  1
what: 2
you:  2
```

The word `what` appears twice, so its count is 2. However, the order of the words is lost.

For a vocabulary of $V$ unique tokens, each document gets a vector of length $V$:

$$
x_{dw}=\operatorname{tf}(d,w)
$$

where $\operatorname{tf}(d,w)$ is the **term frequency**, i.e. the number of occurrences of word $w$ in document $d$.

All documents use the **same vocabulary / column order**, even though most entries are zero.

**Limitation:** Two sentences with the same word counts but different word order have the same BoW vector. Meaning, syntax, and context can therefore be lost.

---

## 3. The document–term matrix and sparsity

Stack the document vectors into a matrix:

$$
X \in \mathbb{R}^{D\times V}
$$

with:

- $D$ = number of documents (**rows**);
- $V$ = vocabulary size (**columns**);
- $X_{dw}$ = count of word $w$ in document $d$.

A simple example:

| Document | `cat` | `dog` | `runs` |
|---|---:|---:|---:|
| `cat runs` | 1 | 0 | 1 |
| `dog runs` | 0 | 1 | 1 |
| `cat cat` | 2 | 0 | 0 |

Vocabulary sizes can reach hundreds of thousands or millions. Most documents use only a tiny fraction of that vocabulary, so the matrix is **high-dimensional and sparse**.

**Why sparse?** Among other reasons, word use follows approximately **Zipf's law**: a few words are extremely frequent, but very many words are rare.

For the word of frequency rank $i$:

$$
p(w_i)\propto \frac{1}{i}.
$$

Taking logarithms:

$$
\log p(w_i)=C-\log i.
$$

Thus, a rank–frequency power law looks approximately like a **straight line with slope $-1$ on log–log axes**.

### Sparse storage

Instead of storing every zero, a sparse matrix stores only nonzero entries, e.g.:

```text
(document_id, word_id, count)
```

This is much more memory-efficient than a dense matrix. For very large text datasets, sparse representation is essential.

---

## 4. Preparing text before counting words

A BoW matrix is only as useful as the preprocessing used to construct it. The lecture emphasizes that there is **no universally correct preprocessing pipeline**.

### Character encoding

Text must be decoded with the same encoding used to write it.

- Older encodings: ASCII, Latin-1.
- Unicode encodings: UTF-8, UTF-16, UTF-32.

Incorrect decoding can corrupt accented letters and other characters (`à`, `ü`, `ß`, etc.), which then corrupts downstream tokens.

**Practical rule:** Specify the encoding explicitly when reading and writing files (UTF-8 is a common default).

### Language identification

Different languages need different tokenization and normalization decisions.

- Language metadata may be provided but is not always reliable or present.
- Automatic identification can use patterns such as **character trigrams**.
- Multilingual corpora make preprocessing harder.

### Tokenization

**Tokenization** splits a string into tokens, approximately words and punctuation units.

```text
"Hello! How are you?"
    ↓
["Hello", "!", "How", "are", "you", "?"]
```

Simply splitting on spaces or punctuation fails in many cases:

- abbreviations such as `Mr.`;
- emoticons and unusual punctuation;
- languages without spaces between words (e.g., Chinese);
- compound words (e.g., long German nouns).

The lecture recommends using established libraries such as **spaCy** or **NLTK** rather than inventing a tokenizer.

### Stopword removal

**Stopwords** are very frequent tokens, e.g. `the`, `a`, `is`, which often carry little information for document topics.

Removing them can let more informative words stand out. But:

- the right stopword list depends on the task;
- punctuation and function words can be useful for **author identification** and psychological analysis;
- automatically removing frequent words may discard useful evidence.

**Remember:** Stopword removal is an optional modeling decision, not an obligatory cleaning step.

---

## 5. Normalizing words: what should count as the same token?

The lecture presents several ways to reduce vocabulary size (and hence sparsity), each with tradeoffs.

| Method | Idea | Benefit | Risk |
|---|---|---|---|
| **Casefolding** | Convert to a common case: `Yams` → `yams` | Combines variants | Loses distinctions such as `apple` vs. `Apple` |
| **Stemming** | Heuristically strip affixes: `walking`, `walked` → `walk` | Fewer distinct tokens | May combine unrelated meanings or produce nonwords (`business`, `busy` → `busi`) |
| **Lemmatization** | Map tokens to dictionary / canonical forms | More linguistically informed | Needs language resources and sometimes context |

The lecture cautions that aggressive normalization **discards information**. It specifically notes that casefolding and stemming may be less useful in very large English datasets, while reducing sparsity can help in smaller or morphologically richer datasets.

### Why context matters

Some expressions look similar but mean different things. For example:

- `apple` (fruit) vs. `Apple` (company);
- `operating` in `operating system` vs. other uses of `operate`.

Normalization can help counting but cannot by itself resolve all ambiguities.

### Social media makes this harder

Text such as:

```text
ikr smh he asked fir yo last name so he can add u on fb lololol
```

contains abbreviations, spelling variation, slang, and repeated letters.

Traditional processing tools may not work well. The lecture points to specialized social-media NLP tools and LLMs as alternatives.

---

## 6. Beyond single words: n-grams

A BoW model usually has one column per **unigram** (single token). But some concepts consist of multiple words:

- `United States`
- `operating system`

An **n-gram** is a sequence of $n$ consecutive tokens:

- **unigram:** `operating`
- **bigram:** `operating system`
- **trigram:** `new operating system`

Including n-grams preserves some local word-order information, but greatly increases the number of possible features (a **combinatorial explosion**).

A smarter strategy is **phrase selection**: keep promising multiword expressions instead of every possible sequence.

For bigrams, the slides mention using **mutual information** between constituent words as a criterion for selecting strongly associated word pairs.

---

## 7. Why raw term counts are not enough

Not all words are equally informative. A word appearing in almost every document is usually less useful for distinguishing documents than a relatively uncommon word.

Compare:

- `the`: appears everywhere;
- `perceptron`: appears in a much more specific subset.

Simply deleting stopwords is a crude solution. A more flexible one is to **downweight frequent words**.

### Inverse document frequency (IDF)

Define:

- $D$ (or $N$ on the slides) = total number of documents;
- $\operatorname{df}(w)$ = number of documents containing word $w$.

Then the lecture's IDF formula is

$$
\boxed{
\operatorname{idf}(w)=
-\log\left(\frac{\operatorname{df}(w)}{D}\right)
=
\log\left(\frac{D}{\operatorname{df}(w)}\right)
}
$$

Interpretation:

- If $w$ appears in **every** document, $\operatorname{idf}(w)=0$.
- If $w$ appears in **few** documents, $\operatorname{idf}(w)$ is large.
- IDF therefore weights **rare-across-documents** terms more strongly.

The slides also relate IDF to **information content**: seeing a rare event gives more information than seeing a common one. With logarithm base 2, the information unit is bits.

---

## 8. TF-IDF: combine term counts and rarity

The **TF-IDF weight** of a word in a document is:

$$
\boxed{
\operatorname{tfidf}(d,w)
=
\operatorname{tf}(d,w)\cdot\operatorname{idf}(w)
}
$$

- **TF:** How often does word $w$ appear *in this document*?
- **IDF:** How rare is $w$ *across the whole corpus*?

The TF-IDF matrix is the original document–term count matrix with each **word column** reweighted by its IDF.

### Worked example

Suppose a corpus contains $D=100$ documents.

- `the` occurs in all 100 documents.
- `robotics` occurs in only 5 documents.
- A particular document contains `robotics` twice.

Using natural logarithms:

$$
\operatorname{idf}(\text{the})
=\ln(100/100)=0
$$

$$
\operatorname{idf}(\text{robotics})
=\ln(100/5)=\ln(20)\approx 3.00
$$

Therefore:

$$
\operatorname{tfidf}(d,\text{robotics})
=
2\cdot 3.00
\approx 6.00.
$$

`robotics` receives substantial weight while `the` receives zero weight.

**Key difference:** TF counts occurrences within a document; DF counts how many distinct documents contain the word (not how often it appears in them).

---

## 9. Normalize document vectors (rows)

Long documents usually contain more words and may therefore have **larger vector magnitudes** than short documents. This can affect distance- or dot-product-based methods even if the two documents are topically similar.

The lecture proposes normalizing the **rows** of the TF-IDF matrix.

Let $\mathbf{x}_d$ be a document's TF-IDF vector.

### L2 normalization

$$
\boxed{
\mathbf{x}_d'=
\frac{\mathbf{x}_d}{\|\mathbf{x}_d\|_2},
\qquad
\|\mathbf{x}_d\|_2=
\sqrt{\sum_w x_{dw}^2}
}
$$

For a nonzero vector, the resulting Euclidean norm is 1:

$$
\|\mathbf{x}_d'\|_2=1.
$$

Thus, the normalized document vectors lie on a unit sphere.

### L1 normalization

For the nonnegative TF-IDF entries used here:

$$
\boxed{
\mathbf{x}_d'=
\frac{\mathbf{x}_d}{\|\mathbf{x}_d\|_1},
\qquad
\|\mathbf{x}_d\|_1=\sum_w |x_{dw}|
}
$$

Now the entries sum to 1 and can be interpreted as proportions.

### L1 vs. L2

| | L1 normalization | L2 normalization |
|---|---|---|
| Makes equal | Sum of absolute feature values = 1 | Euclidean vector length = 1 |
| Useful interpretation | Relative feature weights | Direction of the document vector |
| Which to use? | **Depends on task and downstream model** | **Depends on task and downstream model** |

Neither method automatically fixes every problem caused by different document lengths.

---

## 10. Normalize word features (columns)

The lecture also discusses changing the **columns** of the document–term matrix.

- **IDF weighting:** reweights each column based on how common its word is across documents.
- **Other transformations from Lecture 8:** min–max scaling and standardization may also be useful.

Remember the distinction:

| Matrix direction | What it represents | Typical operation |
|---|---|---|
| **Rows** | Documents | L1 or L2 normalization, reducing magnitude differences between documents |
| **Columns** | Words/features | IDF reweighting or other feature scaling |

There is no universal best combination. Decide based on the analysis task and evaluate the downstream model rather than assuming more preprocessing must be better.

---

## 11. End-to-end mental model

```text
Raw textual documents
        |
        v
Correct character encoding + language identification
        |
        v
Tokenization
        |
        v
Optional stopword removal and word normalization
        |
        v
Unigrams and selected n-grams
        |
        v
Build shared vocabulary
        |
        v
Document–term count matrix (sparse BoW)
        |
        v
Apply IDF column weights
        |
        v
TF-IDF matrix
        |
        v
Optional L1/L2 row normalization
        |
        v
Numerical features for retrieval,
classification, sentiment analysis,
or topic detection
```

**Practical modeling note:** If the representation is used in a supervised prediction experiment, learn the vocabulary, document frequencies, and any fitted preprocessing parameters **on the training data only**. Apply that fitted transformation to validation/test data to avoid leakage (connection to Lecture 8).

---

## Quick revision checklist

Make sure you can explain these without looking at the notes:

1. What distinguishes **document retrieval**, **classification**, **sentiment analysis**, and **topic detection**?
2. Why do we need a fixed-length feature vector to apply traditional ML to text?
3. What does **bag of words** retain, and what information does it discard?
4. Why is a document–term matrix sparse? What does **Zipf's law** say?
5. Why can tokenization, stopword removal, stemming, and casefolding help **or hurt**?
6. What does an n-gram represent, and why not use all possible n-grams?
7. What is the difference between **term frequency** and **document frequency**?
8. Why does a word in nearly every document have low IDF?
9. How do you calculate **TF-IDF**, and why is it often preferable to plain counts?
10. What is the difference between **row normalization** and **column weighting**?

### The 5 ideas to remember

- **Text must be represented numerically** before most traditional ML methods can use it.
- **BoW counts tokens but loses their order**, producing high-dimensional sparse data.
- **Preprocessing involves tradeoffs**: merging variants reduces sparsity but may destroy useful distinctions.
- **TF-IDF = within-document frequency × across-document rarity**.
- **Normalization and weighting matter**, and the best choices depend on the downstream task.

---

*Based on the uploaded ADA Lectures 10+11 slide deck, **slides 1–42 only**. Lecture 11 starts at slide 43 and is deliberately excluded.*
