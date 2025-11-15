## The difference: `axis` in `nunique()` vs `dropna()`

These two functions use `axis` differently:

### 1. `df.nunique(axis=1)` — aggregation function

- `axis=1` means: aggregate along columns (horizontally)
- Result: one value per row
- You're collapsing columns → get a result per row

```
df.nunique(axis=1)
# Moves ACROSS columns (A→B→C) for each row
# Returns: Series with one value per row
```

### 2. `coffee.dropna(subset=['Units Sold'], axis=0)` — filtering function

- `axis=0` means: drop along the row axis (drop entire rows)
- Result: rows are removed
- You're removing rows → fewer rows remain

```
coffee.dropna(subset=['Units Sold'], axis=0)
# Checks rows, drops rows that have NaN
# Returns: DataFrame with fewer rows
```

## Why the difference?

**For aggregation functions** (like `nunique`, `sum`, `mean`):
- `axis=1` → aggregate across columns → result per row
- `axis=0` → aggregate down rows → result per column

**For filtering/dropping functions** (like `dropna`, `drop`):
- `axis=0` → drop rows
- `axis=1` → drop columns

## Summary

| Function Type | `axis=0` | `axis=1` |
|--------------|----------|----------|
| **Aggregation** (`nunique`, `sum`, `mean`) | Aggregate down rows → result per **column** | Aggregate across columns → result per **row** |
| **Dropping** (`dropna`, `drop`) | Drop **rows** | Drop **columns** |

**In your examples:**
- `nunique(axis=1)` → aggregate across → per row
- `dropna(axis=0)` → drop rows

They serve different purposes, so `axis` has different meanings.