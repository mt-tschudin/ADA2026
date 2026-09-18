# ADA — Lecture 2: Handling Data

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Lecture:** 2 — Handling Data  
**Date:** 16 September 2026

> These notes summarize the key concepts from the lecture slides. The lecture is organized around three main parts: **data models**, **data sources**, and **data wrangling**.

---

## 1. Big picture: “Cooking with data”

A useful way to think about data analysis is as a pipeline:

1. **Data models** — How do we represent and store the world?
2. **Data sources** — Where does the data come from?
3. **Data wrangling** — How do we transform raw data into usable data?

A central idea of the lecture is that the **representation of data strongly affects how easy it is to store, process, combine, and analyze it**.

---

# Part 1 — Data models

## 2. What is a data model?

A data model specifies **how we think about the world**.

A conceptual representation may distinguish:

- **Entities** — objects or things of interest
- **Attributes** — properties of entities
- **Relationships** — connections between entities

An entity–relationship diagram is one way to describe such a view of the world.

The key question is then:

> **How should this conceptual model be stored on a computer?**

The lecture distinguishes four major models.

| View of the world | Data model |
|---|---|
| One type of entity with the same attributes | **Flat model** |
| Multiple entity types connected by relationships | **Relational model** |
| Hierarchy of entities | **Document model** |
| Complex network of entities | **Network model** |

---

## 3. Flat model

The **flat model** is appropriate when the data consists mainly of one entity type with a common set of attributes.

Typical examples:

- log files
- CSV files

Example: a web-server log may contain one line per HTTP request.

Conceptually:

```text
request_1
request_2
request_3
...
```

Each line describes the same general type of entity.

### CSV

CSV is one of the most common flat-data formats.

Example:

```csv
user_id,age,country
1,24,CH
2,31,FR
3,19,DE
```

Advantages:

- simple
- human-readable
- widely supported

Limitations:

- weak representation of nested or relational structure
- types often need to be inferred during parsing

---

## 4. Relational model

The **relational model** represents data using **tables**.

It is well suited when the world contains:

- several types of entities
- relationships between those entities

Common relational database systems include:

- MySQL
- PostgreSQL
- Oracle
- DB2
- SQLite

### Example

One table can describe entities:

```text
president
+----+-------+
| id | name  |
+----+-------+
| 1  | Bush  |
| 2  | Trump |
| 3  | Obama |
+----+-------+
```

Another table can describe relationships:

```text
successor
+-----------+-----------+
| president | successor |
+-----------+-----------+
| 1         | 3         |
| 3         | 2         |
+-----------+-----------+
```

This separation lets us represent relationships explicitly.

---

## 5. SQL

SQL is the standard language used to manipulate relational data.

A key property of SQL is that it is **declarative**:

> You specify **what** result you want rather than the exact algorithm for computing it.

Example:

```sql
SELECT *
FROM dogs
INNER JOIN owners
    ON dogs.owner_id = owners.id;
```

### Important SQL concepts

You should know the basics of:

- `SELECT`
- `UPDATE`
- `DELETE`
- unique keys
- sorting
- aggregation
- `GROUP BY`
- `COUNT`
- `MIN`
- `MAX`
- `AVG`
- joins

### Common joins

- **INNER JOIN**
- **LEFT OUTER JOIN**
- **RIGHT OUTER JOIN**
- **FULL OUTER JOIN**

Joins are fundamental because they combine information stored in different tables.

---

## 6. Pandas as a SQL-like tool

Pandas follows many ideas similar to SQL.

Correspondence:

```text
SQL table  <->  Pandas DataFrame
```

Pandas is also influenced by functional programming through operations such as:

- `map()`
- `filter()`

### Advantages of Pandas

- lightweight
- fast for many local analyses
- native Python
- easy integration with Python functions
- integrates well with plotting libraries such as Matplotlib

### Limitations of Pandas

- tables generally need to fit into memory
- no database-style post-load indexing
- no transactions or journaling
- large and complicated joins may be slower than in database systems

A useful practical rule is:

> Use Pandas for flexible in-memory analysis; use a proper database when data size, indexing, transactions, or complex multi-user workflows matter.

---

## 7. Unix command line as a data-processing pipeline

The lecture also shows that declarative or compositional data processing is not limited to SQL.

Unix commands can be chained together:

```bash
cat users.txt \
| awk '$2 >= 18 && $2 <= 25' \
| join -1 1 -2 1 - url_visits.txt \
| cut -f 4 \
| sort \
| uniq -c \
| sort -k 1,1 -n -r \
| head -n 5
```

The general idea is:

> Build a complex analysis by composing many simple transformations.

This resembles the logic of SQL queries and Pandas pipelines.

---

## 8. Document model

The **document model** is appropriate when the world is naturally hierarchical.

Common formats:

- XML
- JSON

Example in JSON:

```json
{
  "id": 656,
  "firstname": "Chuck",
  "lastname": "Smith",
  "phones": [
    "(123) 555-0178",
    "(890) 555-0133"
  ],
  "address": {
    "street": "Rue de l'Ale 8",
    "city": "Lausanne",
    "zip": 1007,
    "country": "CH"
  }
}
```

The data structure is naturally a **tree**:

```text
contact
├── id
├── firstname
├── lastname
├── phones
│   ├── phone 1
│   └── phone 2
└── address
    ├── street
    ├── city
    ├── zip
    └── country
```

### Relational vs. document representation

Suppose a person can have multiple phone numbers.

In a document model, the phone numbers can simply be stored as an array.

In a relational model, they would typically be stored in a separate table:

```text
person
+-----+------------+
| id  | first_name |
+-----+------------+
| 656 | Chuck      |
+-----+------------+
```

```text
phone
+-----+----------------+
| id  | phone          |
+-----+----------------+
| 656 | (123) 555-0178 |
| 656 | (890) 555-0133 |
+-----+----------------+
```

The two models can represent the same information, but they organize it differently.

---

## 9. Processing XML and JSON

Since a document is structured like a tree, it can be processed using tree traversal:

- depth-first search
- breadth-first search

Specialized query tools can also be used:

- **XQuery** for XML
- **jq** for JSON

---

## 10. Network model

The **network model** is useful when the world consists of a complex network of entities and relationships.

Typical examples include:

- social networks
- knowledge graphs
- citation networks
- transportation networks

Conceptually, this is a graph:

\[
G = (V, E)
\]

where:

- \(V\) = set of nodes/entities
- \(E\) = set of edges/relationships

This notation was not explicitly developed in the lecture, but it is the standard mathematical representation of the network-model idea shown in the slides.

---

## 11. Binary data formats

Text formats such as CSV, JSON, or XML need to be **parsed**.

Parsing means converting strings into program-level data types such as:

- `int`
- `float`
- `bool`
- arrays
- lists

Parsing can be expensive.

Binary formats avoid repeatedly converting values to and from text and can support:

- nested structures
- schema enforcement
- compression
- efficient storage
- efficient loading

Examples mentioned in the lecture:

- Python `pickle`
- Java `Serializable`
- Protocol Buffers
- Avro
- Parquet

### Parquet

Parquet is **column-oriented**, which is especially useful in analytical workloads.

A major practical recommendation from the lecture is:

> For large datasets, consider converting raw data into an appropriate binary format early in the processing pipeline.

---

# Part 2 — Data sources

## 12. Data comes in many levels of structure

Data sources can be divided roughly into:

### Structured data

Data with a clear schema.

Examples:

- application databases

### Semi-structured data

Data with some self-describing structure.

Examples:

- server logs
- event logs
- CSV
- JSON
- XML

### Unstructured data

Data without an immediately usable tabular schema.

Examples:

- webpages
- free text
- images
- video

A real-world analysis may combine all three.

---

## 13. Wikipedia as a data source

Wikipedia contains a very large and rich collection of data.

Possible access methods include:

- REST APIs
- XML dumps
- SQL database dumps

Practical difficulties include:

- Unicode
- dataset size
- recency
- parsing wiki markup

A useful strategy is to:

1. search for existing tools/projects that already solve part of the problem
2. prefer more structured versions of the data when possible

---

## 14. Wikidata

Wikidata is described as a more database-like representation of Wikipedia knowledge.

For example, several language-specific names can all refer to the same entity:

```text
Suisse
Schweiz
Svizzera
Switzerland
    ↓
   Q39
```

Wikidata supports:

- API access
- full database dumps

Formats include:

- JSON → document model
- RDF → network model

This is a good example of how the same underlying information can be represented using different data models.

---

## 15. Web data and crawling

Large collections of webpages can be downloaded from sources such as **Common Crawl**.

For specific websites, a crawler or “spider” may be used.

Tools mentioned in the lecture include:

- Apache Nutch
- Storm
- Heritrix
- Scrapy
- `wget`

---

## 16. Useful HTML tools

### Requests

Python library for HTTP requests.

Typical use:

```python
import requests

response = requests.get(url)
html = response.text
```

### Scrapy

Framework for building web crawlers.

### Beautiful Soup

Python library for parsing and navigating HTML.

### Regular expressions

Regex can also be useful for extracting patterns from text, although HTML-specific parsers are generally better suited to structured HTML.

---

## 17. Structured information inside HTML

A webpage can look unstructured to a human reader while still containing machine-readable metadata.

The lecture uses **Schema.org microformats** as an example.

Conceptually:

```html
<div itemscope itemtype="http://schema.org/Movie">
    <h1 itemprop="name">Avatar</h1>
    ...
</div>
```

This embeds structured semantic information inside HTML.

---

## 18. Web services and REST APIs

Many websites discourage direct screen scraping and instead provide APIs.

The lecture presents **REST** as a common framework.

Basic idea:

1. a client requests a resource using a URL
2. the request is sent over HTTP
3. the server returns a representation of the resource

Common response formats include:

- JSON
- XML
- plain text

Example:

```bash
curl http://www.example.org/users/jane/
```

Possible response:

```json
{
  "user": {
    "name": "Jane",
    "gender": "female",
    "location": {
      "href": "http://www.example.org/us/ny/new_york",
      "text": "New York"
    }
  }
}
```

A client can follow linked resources by making additional HTTP requests.

An important implication is that the client must understand the returned resource format.

---

# Part 3 — Data wrangling

## 19. Why raw data is difficult

Real-world data comes in many forms:

- CSV
- PDF
- SQL dumps
- images
- logs
- HTML
- JSON
- etc.

Even within one format, files may differ because of:

- missing values represented differently
- extra header rows
- character-encoding problems
- inconsistent delimiters
- malformed values
- duplicates
- anomalies

Therefore:

> Raw data should not be analyzed blindly.

---

## 20. What is data wrangling?

**Data wrangling** is the process of turning raw data into usable data.

Main goals:

- extract data
- standardize data
- combine multiple data sources
- clean anomalies

A recommended strategy is to combine:

- automation
- interactive visualization
- human inspection

The lecture emphasizes that wrangling can take a very large fraction of an analysis project — often more time than the final statistical analysis itself.

---

## 21. Common data problems

Typical problems include:

- **missing data**
- **incorrect data**
- **inconsistent representations**

Examples of inconsistent representation:

```text
"Switzerland"
"CH"
"CHE"
"Swiss"
```

or a person's name appearing in several forms.

The lecture also emphasizes that many data problems require **human intervention**, such as:

- domain experts
- manual inspection
- crowdsourcing

There is always a trade-off:

> Cleaning data can improve quality, but excessive cleaning may remove meaningful information.

---

## 22. Diagnosing data problems

Simple statistics and visualizations are useful diagnostic tools.

They can reveal:

- outliers
- missing values
- impossible values
- suspicious clusters
- corrupted measurements
- unusual patterns

Different visual representations reveal different types of problems.

For large datasets, visualization becomes harder because of overplotting and scale.

A practical solution is:

> **Sample the data** before visualizing it.

---

## 23. Visualization depends on representation

The lecture uses a Facebook graph to demonstrate that the same network can reveal very different patterns depending on how it is visualized.

For example:

- a node-link graph can show connectivity
- an adjacency matrix can expose blocks and communities
- reordering rows and columns can reveal structure that is otherwise hidden

This illustrates a broader principle:

> The representation chosen for data inspection can strongly affect which patterns or problems become visible.

---

## 24. Visualization at scale

Large numbers of points can produce severe overplotting.

A dense scatter plot may hide the true distribution.

Alternatives include:

- aggregation
- binning
- density-based visualization
- sampling

The key lesson is:

> A plot of every raw point is not always the most informative visualization.

---

## 25. Missing data

Missing data should not automatically be replaced with zero.

The lecture illustrates several possible approaches:

1. mark the missing value as zero
2. interpolate and pretend the value was observed
3. omit the missing value
4. interpolate but explicitly indicate that it was missing

The correct choice depends on:

- the domain
- how the data was collected
- why the value is missing

The central rule is:

> **Knowledge about the domain and data-collection process should drive how missing values are handled.**

---

## 26. Questions to ask before analysis

Before starting the main analysis, ask:

### Missingness

- Do I have missing data?
- If values were missing, how would I know?

### Corruption

- Do I have corrupted data?
- Could errors have arisen from measurement problems?
- Could bugs in the processing pipeline have introduced errors?

### Representation

- Is the data in the right format for this analysis?
- Should I parse, reshape, normalize, or convert it?

### Iteration

Data preparation is not necessarily a one-time step.

You may need to return to wrangling after discovering problems during the analysis.

---

## 27. Data provenance

Whenever possible, obtain:

- documentation
- collection methodology
- source code
- processing scripts
- schema definitions
- metadata

Together, these help establish the **provenance** of the dataset.

Provenance helps answer questions such as:

- Where did this value come from?
- How was it generated?
- What transformations were applied?
- What does a missing value mean?
- Can this dataset be trusted for this analysis?

---

## 28. Parseability matters

Well-structured data is much easier to process automatically.

Poorly structured data may require:

- regular expressions
- custom parsers
- OCR
- manual annotation
- human validation

This can dominate the cost of a data-analysis pipeline.

Therefore, a good dataset is not only one that contains useful information, but also one whose structure and provenance make it possible to process reliably.

---

# Key takeaways

1. **Choose a data model that matches the structure of the problem.**
2. **Relational, document, flat, and network models emphasize different structures.**
3. **SQL is declarative: specify what you want, not the full algorithm for obtaining it.**
4. **Pandas provides SQL-like data manipulation inside Python, but is mainly intended for in-memory analysis.**
5. **JSON/XML are natural for hierarchical data; graph/network formats are natural for highly connected data.**
6. **Binary formats can reduce parsing overhead and improve large-scale processing.**
7. **Prefer APIs and structured sources when acquiring online data.**
8. **Real-world data is usually messy and must be inspected before analysis.**
9. **Visualization and basic statistics are important tools for diagnosing data-quality problems.**
10. **Missing values must be treated according to domain knowledge and the data-collection process.**
11. **Data wrangling is iterative and can take a substantial fraction of the total project time.**
12. **Dataset documentation and provenance are crucial for trustworthy analysis.**

---

# Compact mental model

```text
WORLD
  ↓
Choose a data model
  ↓
Acquire data from one or more sources
  ↓
Parse / load / convert
  ↓
Inspect and diagnose
  ↓
Clean / standardize / combine
  ↓
Analyze
  ↓
Discover a problem?
  └──────────────→ return to wrangling
```

The main lesson of the lecture can be summarized as:

> **Good analysis starts long before the statistical analysis itself: it starts with choosing appropriate representations, understanding where the data came from, and making the raw data trustworthy and usable.**
