+++
+++

```python
values = [1, 2, 4, 7, 11]

# as list comprehension
only_small_values = [v for v in values if v < 5]

# using filter
only_small_values = list(filter(lambda v: v < 5, values))

# 0, 1, 2
indices = [i for i, _ in enumerate(values) if v < 5]
```
