nestfs: an R package for cross-validated (nested) forward selection.
======

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/nestfs)](https://cran.r-project.org/package=nestfs)
[![CRAN\_Downloads\_Badge](https://cranlogs.r-pkg.org/badges/nestfs)](https://cran.r-project.org/package=nestfs)

This package provides an implementation of forward selection based on linear
and logistic regression which adopts cross-validation as a core component of
the selection procedure.

Forward selection is an inherently slow approach, as for each variable a
model needs to be fitted. In our implementation, this issue is further
aggravated by the fact that an inner cross-validation happens at each
iteration, with the aim of guiding the selection towards variables that
have better generalization properties.

The code is parallelized over the inner folds, thanks to the `foreach`
package. User time therefore depends on the number of available cores, but
there is no advantage in using more cores than inner folds. A parallel
backend must be registered before starting, otherwise operations will run
sequentially with a warning reported. This can be done through a call to
`registerDoParallel()`, for example.

The main advantage of forward selection is that it provides an immediately
interpretable model, and the panel of variables obtained is in some sense
the least redundant one, particularly if the number of variables to choose
from is not too large (in our experience, up to about 30-40 variables).

However, when the number of variables is much larger than that, forward
selection, besides being unbearably slow, may be more subject to overfitting,
which is in the nature of its greedy-like design.

A precompiled package is
[available on CRAN](https://cran.r-project.org/package=nestfs).

## Usage

First load the package and register a parallel cluster, setting the number of
cores to use. If you are lucky enough to work on a large multicore machine,
best performance is achieved by registering as many cores as the number of inner
folds being used (the default is 30).

```r
library(nestfs)
library(doParallel)
registerDoParallel(10)
```

To run forward selection from a baseline model that contains only age and sex,
the following is enough:

```r
data(diabetes)
fs.res <- forward.selection(X.diab, Y.diab, ~ age + sex, family=gaussian())
summary(fs.res)
```

By default, selection happens over all variables present in the data.frame
that are not part of the initial model. This can be controlled through the
`choose.from` option.

It is possible to promote sparser selection by requesting a larger improvement
in log-likelihood (option `min.llk.diff`, by default set to 0), or reducing the
number of iterations (option `max.iters`, by default set to 15).

To obtain a cross-validated measure of performance of the selection process,
nested forward selection should be run:

```r
cv.folds <- create.folds(10, nrow(X.diab), seed=1)
nestfs.res <- nested.forward.selection(X.diab, Y.diab, ~ age + sex,
                                       family=gaussian(), folds=cv.folds)
summary(nestfs.res)
nested.performance(nestfs.res)
```
