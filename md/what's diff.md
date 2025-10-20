Great question 👍

In **pandas**, the method **`.diff()`** calculates the **difference between each value and the one before it** in a column or row.

---

### 🧩 Example

```python
import pandas as pd

s = pd.Series([10, 13, 15, 20])
print(s.diff())
```

**Output:**

```
0    NaN      # first value has no previous value
1    3.0      # 13 - 10
2    2.0      # 15 - 13
3    5.0      # 20 - 15
dtype: float64
```

---

### 💡 In your function

```python
point['x'].diff()
```

This computes the **difference between each x value and the previous x value** (after sorting).
Then we take `.abs()` to get absolute distances, since distance is always positive.

Example:

```python
point = pd.DataFrame({'x': [-1, 0, 2]})
point = point.sort_values(by='x')
print(point['x'].diff())
```

Output:

```
0    NaN
1    1.0   # 0 - (-1)
2    2.0   # 2 - 0
Name: x, dtype: float64
```

So `.diff()` helps us easily find **the distance between consecutive points** instead of manually looping or subtracting.
