Thanks for the sample output — that helps. The mismatch you’re seeing is almost certainly due to **rounding**.
Pandas/NumPy use **banker’s rounding** (round-half-to-even), while LeetCode/PostgreSQL expect **round-half-up** (e.g., 66.665 → 66.67). Since you even imported `Decimal`, here’s a version that matches the SQL behavior exactly.

```python
import pandas as pd
from decimal import Decimal, ROUND_HALF_UP

def queries_stats(queries: pd.DataFrame) -> pd.DataFrame:
    df = queries.copy()

    # quality = AVG(rating / position) with float division
    df['ratio'] = df['rating'].astype(float) / df['position'].astype(float)

    # group and compute raw metrics
    out = (
        df.groupby('query_name', as_index=False)
          .agg(
              quality=('ratio', 'mean'),
              poor_query_percentage=('rating', lambda s: (s.lt(3).mean() * 100.0))
          )
    )

    # Use SQL-like ROUND(..., 2) behavior: HALF UP
    def round_half_up(x: float) -> float:
        return float(Decimal(str(x)).quantize(Decimal('0.01'), rounding=ROUND_HALF_UP))

    out['quality'] = out['quality'].map(round_half_up)
    out['poor_query_percentage'] = out['poor_query_percentage'].map(round_half_up)

    return out
```

### Why this fixes your case

* **quality**: computed as the mean of `rating / position` (float), then rounded **half up** to 2 decimals.
* **poor_query_percentage**: computed as `(count(rating < 3) / total) * 100`, then rounded **half up** to 2 decimals.
* Using `Decimal(..., ROUND_HALF_UP)` ensures values like **66.665 → 66.67**, matching your posted results (e.g., 66.67, 16.67, etc.), rather than sometimes getting 66.66 from banker’s rounding.

If you still see differences, check:

1. You’re not doing integer division anywhere (`/` not `//`, and cast to float as shown).
2. You group **by `query_name` only** (not including other columns).
3. The dataset doesn’t have unexpected NaNs in `rating`/`position` (would change means).
<br><br>
**Banker’s rounding** is a rounding rule also called:

* **round half to even**
* **banker’s rule**

It is the **default** rounding method in many programming languages, floating-point hardware, Pandas, NumPy, and IEEE-754 arithmetic.

---

# ✅ Short definition

When the number is exactly **halfway** between two digits (like 2.345 **5**), instead of always rounding **up**, banker’s rounding will round to the **nearest even** number.

---

# 🔥 Example (halfway cases)

### Banker’s rounding (round half to even):

```
2.345 **5** → 2.346  (next even digit = 6)
2.344 **5** → 2.344  (next even digit = 4)
```

Notice 4 is even, so it stays.

---

# ✅ More examples

| Value | Traditional (round half up) | Banker’s rounding (half to even)         |
| ----- | --------------------------- | ---------------------------------------- |
| 1.5   | 2                           | 2 (nearest even)                         |
| 2.5   | 3                           | 2 (nearest even) ← this surprises people |
| 3.5   | 4                           | 4                                        |
| 4.5   | 5                           | 4                                        |

---

# ✅ Why does this exist?

Because rounding **always up** on ties introduces statistical bias.

Imagine millions of `.5` values:

* always up → shifts numbers **upward** over time
* half to even → evens out upward & downward bias

That’s why financial systems historically used it.

---

# ✅ Used in:

* IEEE-754 floating point (hardware)
* Python `round()`
* NumPy
* Pandas
* Excel

So:

```python
round(2.5)  # → 2
round(3.5)  # → 4
```

Many people think this is a bug 😄

---

# ✅ When do we want **traditional** rounding?

In SQL (PostgreSQL), Java BigDecimal, finance, accounting, LeetCode output, etc., we use:

**round half up**

This always rounds `.5` upward.

---

# ✅ Summary table

| Method    | Rule              |
| --------- | ----------------- |
| Banker’s  | .5 → nearest even |
| Half up   | .5 → always up    |
| Half down | .5 → always down  |
| Floor     | always down       |
| Ceil      | always up         |

---

# ✅ Why your earlier answer was wrong

Pandas used banker’s rounding, so values like:

```
66.665 → 66.66  ❌ (banker’s)
66.665 → 66.67  ✅ (SQL)
```

caused mismatches.

---

If you want, I can show:

* how to reproduce the difference in Python,
* visual graphs of bias,
* why banks chose that method.

Just ask 😎
