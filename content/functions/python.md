+++
+++

```python
def num_small_elements(v, threshold):
    n = 0
    for i in v:
        if i < threshold:
            n += 1
    return n


print(num_small_elements([1, 2, 4, 7, 11], 5))
```
