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

## Checklist before submitting a notebook

- `Kernel → Restart & Run All` — it must run top-to-bottom without errors.
- Delete dead cells: leftover `mask.head(5)`, `df.plot.scatter?` help calls,
  commented-out experiments.
- Check `df.dtypes` after every load, and row counts after every merge.
- Every plot gets axes labels + a sentence saying what it shows and what follows
  from it.
- Don't mutate a source column when you only need a derived version of it.
