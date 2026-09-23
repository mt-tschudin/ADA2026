# Recap — Week 01: Handling data (pandas basics)

Notes written 2026-09-23 after comparing my `Happiness.ipynb` against
`Happiness_solution.ipynb`. All my numbers matched the official solution; the
lessons below are about *idiom* and *safety*, not about wrong answers.

---

## 1. Reading a CSV with European decimal commas

```python
countries = pd.read_csv(COUNTRIES_DATASET, decimal=',')
```
Why: the raw file stores literacy as `"36,0"`. Without `decimal=','` pandas
reads the column as `object` (strings), and every later `.mean()` or comparison
silently breaks. Always check `df.dtypes` after loading.

Also: `pd.read_csv()` already returns a DataFrame — don't wrap it in
`pd.DataFrame(...)`.

---

## 2. Merging — build the join key, don't destroy the data

```python
# Good: temporary key column, dropped after the merge
happiness['country_name'] = happiness['country'].str.lower()
country_features = happiness.merge(countries, on="country_name").drop("country_name", axis=1)
```
Why: my version did `happiness['country'] = happiness['country'].str.lower()`,
which overwrote the original column. Every later output then showed `finland`
instead of `Finland`. Normalise into a *new* column when you only need the
lowercase form as a join key.

### Always verify the row count after a merge

```python
print(len(happiness), len(countries), len(country_features))   # 135 135 135
```
Why: `pd.merge` defaults to an **inner** join, which silently drops rows whose
key doesn't match. A typo or a stray accent in one country name loses that row
with no warning. This check is the single most valuable habit in this notebook.

Merge kinds: `how='inner'` (default, keep only matches), `'left'` (keep all of
the left frame), `'outer'` (keep everything, fill gaps with NaN).

---

## 3. Several statistics in one groupby

```python
average_by_region = country_features.groupby("world_region")['happiness_score'].agg(['mean', 'size'])
average_by_region.sort_values("mean", ascending=False)
```
Why: I did two separate `groupby` calls and merged the results — three cells for
what `.agg([...])` does in one. The resulting columns are literally named
`mean` and `size`, so that's what you sort on.

- `size` counts rows, including NaN.
- `count` counts non-NaN values only.
- Identical here, but not in general — pick deliberately.

Named output columns:
```python
.agg(avg_score=('happiness_score', 'mean'), n_countries=('happiness_score', 'size'))
```

---

## 4. Filtering and showing only relevant columns

```python
country_features[country_features.world_region == 'North America and ANZ'][['country', 'happiness_score']]
country_features.sort_values("happiness_score", ascending=False)[['country', 'happiness_score']].head(10)
```
Why: selecting the 2 columns that matter turns a 12-column wall of numbers into
a readable answer. Assigning a boolean mask to a variable first is fine, but the
inline form is shorter and there's no intermediate to keep straight.

`df['col']` vs `df.col`: both work. Bracket form is safer — it handles names
with spaces and names that collide with methods (`count`, `size`, `mean`).

---

## 5. Formatted printing

```python
for idx, row in country_features[country_features.literacy == 100].iterrows():
    print(f"{row.world_region} - {row.country} ({row.happiness_score})")
```
Why: when a task specifies `{region} - {country} ({score})`, the braces are
*placeholders*, not literal characters. And `print(a, " - ", b)` inserts an
extra space around each argument — use one f-string instead.

### Percent format spec

```python
percentage = len(country_features[country_features.literacy < 50]) / len(country_features)
print(f"Countries with literacy < 50%: {percentage:.2%}")   # 11.85%
```
Why: `:.2%` multiplies by 100 and appends `%` in one step. Writing
`{x*100:.2f}%` works but gives you two chances to forget the `*100`.

---

## 6. Vectorised arithmetic over columns

```python
illiterate_people = country_features.population * (100 - country_features.literacy) / 100
illiterate_fraction = illiterate_people.sum() / country_features.population.sum()

country_features["population_density"] = country_features['population'] / country_features['area']
```
Why: column-wise operations replace any loop over rows. Reach for `.iterrows()`
only for *printing*, never for computing — it's slow and reads badly.

Note this is a weighted aggregate: the illiterate fraction (20.33%) differs from
the unweighted mean literacy (81.85% → 18.15%) because big countries count more.
Know which one a question is asking for.

---

## 7. Scatter plot

```python
country_features.plot(x="happiness_score", y="healthy_life_expectancy", kind="scatter")

# better for a graded notebook:
country_features.plot.scatter(x='happiness_score', y='healthy_life_expectancy',
                              title='Healthy and happy?',
                              xlabel='Happiness Score', ylabel='Healthy Life Expectancy')
country_features[['happiness_score', 'healthy_life_expectancy']].corr()
```
Why: the solution plots a bare scatter, but homework criterion 2 asks for a
description of every plot — axes labels, title, and a correlation coefficient to
back the claim. Doing more than the solution here was the right call.

---

## 8. Pandas capability map

The point of this section is **recognition, not syntax**: you can only look up an
operation if you know it exists. Skim the left column until something matches the
shape of your problem, then look up the details.

Source: `Intro to Pandas I.ipynb` (= I) and `Intro to Pandas II.ipynb` (= II).

### Loading and inspecting — I

| Need | Tool |
|---|---|
| Read a table | `pd.read_csv` / `read_table` / `read_excel` |
| Non-standard separator | `sep='\s+'` (regex allowed) |
| Comma decimals, custom header, index column | `decimal=','`, `header=`, `names=`, `index_col=` |
| Skip bad rows / read only the first N | `skiprows=`, `nrows=` |
| Parse dates at load time | `parse_dates=['col']` |
| Check types, shape, summary | `.dtypes`, `.shape`, `.info()`, `.describe()`, `.head()` |
| Write results out | `.to_csv()`, `.to_excel()` |

### Selecting — I

| Need | Tool |
|---|---|
| By label | `df.loc[row_label, col_label]` |
| By position | `df.iloc[0, 2]` |
| Boolean filter | `df[df.literacy < 50]` |
| Combine conditions | `&`, `\|`, `~` — each condition in its own parentheses |
| Membership | `df[df.country.isin([...])]` |
| Several columns | `df[['country', 'happiness_score']]` |
| Multi-level row labels | hierarchical / MultiIndex |

### Combining tables — II

| Need | Tool |
|---|---|
| Join on a shared key (SQL-style) | `pd.merge(a, b, on='key')` or `a.merge(b, ...)` |
| Keep non-matching rows | `how='left'` / `'right'` / `'outer'` (default `'inner'`) |
| Key is an index on one side | `left_on='col', right_index=True` |
| Shared column names not used as keys | they get `_x` / `_y`; override with `suffixes=` |
| Stack rows or columns of same-shaped data | `pd.concat([a, b], axis=0 or 1)` |

### Reshaping, long ↔ wide — II

| Need | Tool |
|---|---|
| Columns → rows | `.stack()` |
| Rows → columns | `.unstack()` |
| Long → wide, one value per cell | `.pivot(index=, columns=, values=)` |
| Long → wide, aggregating duplicates | `.pivot_table(..., aggfunc=)` |
| Wide → long | `pd.melt(df, id_vars=, value_vars=)` |
| Frequency cross-tabulation of two factors | `pd.crosstab(a, b)` |

Long format = one observation per row (preferred for storage: new data is new
rows). Wide format = one subject per row, repeated measures across columns
(often needed for plotting or modelling).

### Grouping and aggregating — II

| Need | Tool |
|---|---|
| Split-apply-combine | `df.groupby('col')` |
| One statistic | `.mean()`, `.sum()`, `.size()`, `.count()`, `.max()` |
| Several at once | `.agg(['mean', 'size'])` |
| Named output columns | `.agg(avg=('score','mean'), n=('score','size'))` |
| Arbitrary per-group logic | `.apply(func)` |
| Group result broadcast back to original rows | `.transform(func)` |

### Transforming values — II

| Need | Tool |
|---|---|
| Recode categories via a dict | `.map({'Placebo': 0, ...})` |
| Replace specific values | `.replace(old, new)` |
| Categories → 0/1 indicator columns (design matrix) | `pd.get_dummies(df.col)` |
| Continuous → fixed bins | `pd.cut(x, bins)` |
| Continuous → equal-sized quantile bins | `pd.qcut(x, 4)` |
| Elementwise function on a Series | `.apply(func)` |
| Random subset / shuffle | `.sample(n=)`, `np.random.permutation` |

### Dates — II

| Need | Tool |
|---|---|
| Strings → datetime | `pd.to_datetime(series)`, `format=` if ambiguous |
| At import | `parse_dates=[...]` |
| Extract year/month/hour | `.dt.year`, `.dt.month`, `.dt.hour` |
| Missing timestamp | `NaT` (the datetime equivalent of `NaN`) |
| Difference between two times | subtract → `Timedelta` |

### Missing data and cleaning — I and II

| Need | Tool |
|---|---|
| Find gaps | `.isnull()`, `.notnull()`, `.isnull().sum()` |
| Drop | `.dropna()` |
| Fill | `.fillna(value)` |
| Flag values as missing at load | `na_values=[...]` |
| Find / drop repeated rows | `.duplicated()`, `.drop_duplicates(subset=)` |
| Remove rows or columns | `.drop(labels, axis=0 or 1)` |

---

## 9. Which one do I actually want?

The pairs that are easy to confuse. Worth re-reading right before the exam.

- **`merge` vs `concat`** — `merge` aligns rows by *key value* (SQL join, tables
  have different columns). `concat` glues blocks together by *position/index*
  (same-shaped data, just more of it).
- **`pivot` vs `pivot_table`** — `pivot` fails if an index/column pair has more
  than one value. `pivot_table` aggregates them with `aggfunc` (default `mean`).
  Duplicate keys → you want `pivot_table`.
- **`melt` vs `stack`** — same idea (wide → long). `melt` works on *columns* and
  returns a flat frame; `stack` works on the *index* and gives a MultiIndex.
- **`agg` vs `apply` vs `transform`** (on a groupby) — `agg` returns one row per
  group. `transform` returns the same shape as the input (for adding a
  group-mean column). `apply` is the general case, can return anything.
- **`map` vs `replace` vs `apply`** — `map` for a dict lookup over a Series,
  `replace` for swapping specific values in place, `apply` for an arbitrary
  function.
- **`loc` vs `iloc`** — labels vs integer positions. If the index *is* integers,
  these differ and the bug is silent.
- **`cut` vs `qcut`** — `cut` makes bins of equal *width* (unequal counts),
  `qcut` bins of equal *count* (unequal widths).
- **`size` vs `count`** — `size` includes NaN, `count` excludes them.
- **view vs copy** — indexing a DataFrame usually gives a *view*; writing to it
  triggers `SettingWithCopyWarning` and may not do what you meant. Use `.copy()`
  when you intend to modify an extracted piece.
- **Series alignment** — arithmetic between two Series aligns on *labels*, not
  position, and produces `NaN` where labels don't match. Different from NumPy
  arrays, and a classic source of silently wrong results.

---

## Checklist before submitting a notebook

- `Kernel → Restart & Run All` — it must run top-to-bottom without errors.
- Delete dead cells: leftover `mask.head(5)`, `df.plot.scatter?` help calls,
  commented-out experiments.
- Check `df.dtypes` after every load, and row counts after every merge.
- Every plot gets axes labels + a sentence saying what it shows and what follows
  from it.
- Don't mutate a source column when you only need a derived version of it.
