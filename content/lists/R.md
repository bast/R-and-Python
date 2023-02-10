+++
+++

```r
experiment <- list(location="Svalbard", date="2021-03-23", num_measurements=23)

print(experiment)

# print one item
print(experiment$location)

# update one item
experiment$num_measurements <- 24

# insert a new item
experiment$temperature <- -5.0

# print the updated list
print(experiment)
```
