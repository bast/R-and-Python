+++
+++

```r
num_small_elements <- function(v, threshold) {
  n <- 0
  for (i in v) {
    if (is.numeric(i) && i < threshold) {
      n <- n + 1
    }
  }
  n
}


result <- num_small_elements(c(1, 2, 4, 7, 11), 5)

cat("there are", result, "small elements\n")
```

Default arguments seem to work the same way.
