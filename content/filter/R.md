+++
+++

```r
values <- c(1, 2, 4, 7, NA, 11)

# will contain the NA value
values[values < 5]

# will not contain the NA value
subset(values, values < 5)

# indices where value < 5: 1, 2, 3
which(values < 5)
```
