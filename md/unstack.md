### Explanation of `unstack()` in your code

You used:

```python
action_counts = ads.groupby(['ad_id', 'action']).size().unstack(level='action', fill_value=0)
```

Let’s break down what `unstack()` does here.

---

### 1. What we have before `unstack()`

After this groupby:

```python
ads.groupby(['ad_id', 'action']).size()
```

We get a **Series** where the *index* has **two levels** (a MultiIndex):

```
ad_id  action 
1      Clicked    2
       Ignored    1
       Viewed     1
2      Clicked    1
       Viewed     2
3      Viewed     1
       Clicked    1
5      Ignored    1
dtype: int64
```

This means:
**"For each ad_id and action, count how many rows exist."**

---

### 2. What does `unstack(level='action')` do?

`unstack()` **converts one of the index levels into columns.**

Specifically:

* `ad_id` stays as the index (rows)
* `action` values become column names (`Clicked`, `Viewed`, `Ignored`)
* The counts (values) stay as the table data

Resulting DataFrame:

| ad_id | Clicked | Viewed | Ignored |
| ----- | ------- | ------ | ------- |
| 1     | 2       | 1      | 1       |
| 2     | 1       | 2      | 0       |
| 3     | 1       | 1      | 0       |
| 5     | 0       | 0      | 1       |

The argument `fill_value=0` tells Pandas:

> If an ad has no rows for a specific action (e.g., ad 5 never had "Clicked"), put `0` instead of NaN.

---

### Visual analogy

Before `unstack()`:

```
(ad_id, action) → count

(1, Clicked) → 2
(1, Viewed)  → 1
(1, Ignored) → 1
...
```

After `unstack()`:

```
          Clicked  Viewed  Ignored
ad_id
1              2       1        1
2              1       2        0
3              1       1        0
5              0       0        1
```

---

### Why use `unstack()` here?

Because CTR calculation requires aggregation per ad:

```
CTR = Clicks / (Clicks + Views)
```

With this structure (each action as a column), it's easy to write:

```python
total_interactions = action_counts['Clicked'] + action_counts['Viewed']
```

---

### TL;DR

`unstack()` transforms grouped data from long format to wide format:

**From:**

| ad_id | action | count |
| ----- | ------ | ----- |

**To:**

| ad_id | Clicked | Viewed | Ignored |

---

If you'd like, I can also show a simpler version using `pivot_table` or `crosstab`.
