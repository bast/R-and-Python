+++
template = "chapter-freestyle.html"
+++

## Profiling memory and CPU time in R

[Profvis](https://rstudio.github.io/profvis/) is one of the nicest ways in R to
get a line-by-line profile both for CPU time and memory.

We will need to add the following lines to our code:
```r
library(profvis)

# with this we make sure we can browse the source
# code in the generated HTML page
options(keep.source=TRUE)

p = profvis({

# here is the code you wish to profile ...

})

htmlwidgets::saveWidget(p, "profile.html")
```

Here is an actual example where we compare two ways of approximating pi using
the Monte-Carlo method. One version uses a for-loop, the other is vectorized:
```r
library(profvis)

# with this we make sure we can browse the source
# code in the generated HTML page
options(keep.source=TRUE)

p = profvis({

n <- 1000000

count <- 0

for (i in 1:n) {
  # generate random coordinates between 0 and 1
  x <- runif(1, min = 0, max = 1)
  y <- runif(1, min = 0, max = 1)

  # check if the point is within the unit circle
  if (x^2 + y^2 <= 1) {
    count <- count + 1
  }
}

# calculate the value of pi
approx_pi <- 4 * count / n
print(approx_pi)


# and now let us try with a vectorized version
# --------------------------------------------

# generate a vector of random coordinates between 0 and 1
x <- runif(n, min = 0, max = 1)
y <- runif(n, min = 0, max = 1)

# count how many points are within the unit circle
count <- sum(x^2 + y^2 <= 1)

# calculate the value of pi
approx_pi <- 4 * count / n
print(approx_pi)

})

htmlwidgets::saveWidget(p, "profile.html")
```

The above example will generate a file called "profile.html" which you can open
and browse in your browser. An example is below here:

{{ iframe(src="https://bast.github.io/R-and-Python/profile.html", width="800px", height="1400px") }}

If you want to try this out in a container, then this is the
Singularity/Apptainer definition file which I have used to test this:
```
Bootstrap: docker
From: rstudio/r-base:focal

%post
    R_VERSION=4.3.0
    OS_IDENTIFIER=ubuntu-2004

    wget https://cdn.posit.co/r/${OS_IDENTIFIER}/pkgs/r-${R_VERSION}_1_amd64.deb && \
    apt-get update -qq && \
    DEBIAN_FRONTEND=noninteractive apt-get install -f -y ./r-${R_VERSION}_1_amd64.deb && \
    ln -s /opt/R/${R_VERSION}/bin/R /usr/bin/R && \
    ln -s /opt/R/${R_VERSION}/bin/Rscript /usr/bin/Rscript && \
    ln -s /opt/R/${R_VERSION}/lib/R /usr/lib/R && \
    rm r-${R_VERSION}_1_amd64.deb && \
    rm -rf /var/lib/apt/lists/*

    R --quiet -e "install.packages('profvis', repos='http://cran.us.r-project.org')"
```


## Line-by-line CPU profiling in Python

This example requires the [line_profiler](https://github.com/pyutils/line_profiler) package.

As an example we will compute approximate pi using the Monte-Carlo method
(`example.py`):
```python
import random


@profile
def main():
    n = 1000000

    count = 0

    for _ in range(n):
        # generate random coordinates between 0 and 1
        x = random.random()
        y = random.random()

        # check if the point is within the unit circle
        if x**2 + y**2 <= 1.0:
            count += 1

    # calculate the approximate value of pi
    print(4.0 * count / n)

    coordinates = [(random.random(), random.random()) for _ in range(n)]

    inside_unit_circle = list(
        filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates)
    )
    count = len(inside_unit_circle)
    print(4.0 * count / n)

    count = sum(1 for _ in filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates))
    print(4.0 * count / n)


main()
```

Let us profile the code:
```console
$ kernprof -l -v example.py

3.141016
3.14008
3.14008
Wrote profile results to example.py.lprof
Timer unit: 1e-06 s

Total time: 1.75691 s
File: example.py
Function: main at line 4

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     4                                           @profile
     5                                           def main():
     6         1          0.5      0.5      0.0      n = 1000000
     7
     8         1          0.2      0.2      0.0      count = 0
     9
    10   1000000     143593.2      0.1      8.2      for _ in range(n):
    11                                                   # generate random coordinates between 0 and 1
    12   1000000     188331.9      0.2     10.7          x = random.random()
    13   1000000     180575.0      0.2     10.3          y = random.random()
    14
    15                                                   # check if the point is within the unit circle
    16    785254     222825.2      0.3     12.7          if x**2 + y**2 <= 1.0:
    17    785254     120810.1      0.2      6.9              count += 1
    18
    19                                               # calculate the approximate value of pi
    20         1         30.4     30.4      0.0      print(4.0 * count / n)
    21
    22         1     239801.0 239801.0     13.6      coordinates = [(random.random(), random.random()) for _ in range(n)]
    23
    24         1     304028.4 304028.4     17.3      inside_unit_circle = list(
    25         1          3.5      3.5      0.0          filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates)
    26                                               )
    27         1          1.5      1.5      0.0      count = len(inside_unit_circle)
    28         1         25.9     25.9      0.0      print(4.0 * count / n)
    29
    30         1     356858.8 356858.8     20.3      count = sum(1 for _ in filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates))
    31         1         25.9     25.9      0.0      print(4.0 * count / n)
```


## Line-by-line memory profiling in Python

This example requires the [memory_profiler](https://github.com/pythonprofilers/memory_profiler) package.

The only change compared to the CPU profiling example (above) is that we add the line `from
memory_profiler import profile` on top:
```python
from memory_profiler import profile
import random


@profile
def main():
    n = 1000000

    count = 0

    for _ in range(n):
        # generate random coordinates between 0 and 1
        x = random.random()
        y = random.random()

        # check if the point is within the unit circle
        if x**2 + y**2 <= 1.0:
            count += 1

    # calculate the approximate value of pi
    print(4.0 * count / n)

    coordinates = [(random.random(), random.random()) for _ in range(n)]

    inside_unit_circle = list(
        filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates)
    )
    count = len(inside_unit_circle)
    print(4.0 * count / n)

    count = sum(1 for _ in filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates))
    print(4.0 * count / n)


main()
```

And here is the result (the runtime for the above is considerably longer than
for profiling the runtime hotspots above):
```console
$ python example.py

3.14176
3.142728
3.142728
Filename: /home/bast/tmp/memory-profile/mem-example.py

Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     5     17.3 MiB     17.3 MiB           1   @profile
     6                                         def main():
     7     17.3 MiB      0.0 MiB           1       n = 1000000
     8
     9     17.3 MiB      0.0 MiB           1       count = 0
    10
    11     17.3 MiB      0.0 MiB     1000001       for _ in range(n):
    12                                                 # generate random coordinates between 0 and 1
    13     17.3 MiB      0.0 MiB     1000000           x = random.random()
    14     17.3 MiB      0.0 MiB     1000000           y = random.random()
    15
    16                                                 # check if the point is within the unit circle
    17     17.3 MiB      0.0 MiB     1000000           if x**2 + y**2 <= 1.0:
    18     17.3 MiB      0.0 MiB      785440               count += 1
    19
    20                                             # calculate the approximate value of pi
    21     17.3 MiB      0.0 MiB           1       print(4.0 * count / n)
    22
    23    148.8 MiB    131.6 MiB     1000003       coordinates = [(random.random(), random.random()) for _ in range(n)]
    24
    25    154.8 MiB      0.0 MiB           2       inside_unit_circle = list(
    26    154.8 MiB      6.0 MiB     2000001           filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates)
    27                                             )
    28    154.8 MiB      0.0 MiB           1       count = len(inside_unit_circle)
    29    154.8 MiB      0.0 MiB           1       print(4.0 * count / n)
    30
    31    154.8 MiB      0.0 MiB     3571367       count = sum(1 for _ in filter(lambda c: c[0] ** 2 + c[1] ** 2 <= 1.0, coordinates))
    32    154.8 MiB      0.0 MiB           1       print(4.0 * count / n)
```
