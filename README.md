Functions useful for exploratory data analysis using random forests. Developed by [Zachary M. Jones](http://zmjones.com) and [Fridolin Linder](http://polisci.la.psu.edu/people/fjl128) in support of "[Exploratory Data Analysis Using Random Forests](https://github.com/zmjones/rfss/)."

This package allows you to easily calculate the partial dependence of an arbitrarily large set of explanatory variables on the outcome variable given a fitted random forest from the following packages (outcome variable types supported in parenthesis): [party](http://cran.r-project.org/web/packages/party/index.html) (multivariate, regression, and classification), [randomForestSRC](http://cran.r-project.org/web/packages/randomForestSRC/index.html) (regression, classification, and survival), and [randomForest](http://cran.r-project.org/web/packages/randomForest/index.html) (regression and classification).

`partial_dependence` can be run in parallel by registering the appropriate parallel backend using [doParallel](http://cran.r-project.org/web/packages/doParallel/index.html). Beyond the random forest and the set of variables for which to calculate the partial dependence, there are additional arguments which control the dimension of the prediction grid used (naturally, the more points used the longer execution will take) and whether or not points that were not observed in the data can be used in said grid (interpolation).

The `partial_dependence` method can also either return interactions (the partial dependence on unique combinations of a subset of the predictor space) or a list of bivariate partial dependence estimates.

For regression we provide (by default) confidence intervals using the bias-corrected infinitesimal jackknife ([Wager, Hastie, and Tibsharani, 2014](http://jmlr.org/papers/v15/wager14a.html)) using code adapted from [randomForestCI](https://github.com/swager/randomForestCI).

We also provide methods to extract and plot variable importance.

We are planning on adding methods to make interpreting proximity and maximal subtree matrices easier.

Pull requests, bug reports, feature requests, etc. are welcome!

## Contents

 - [Installation](#install)
 - [Partial Dependence](#partial_dependence)
    + [Classification](#classification)
    + [Regression](#regression)
	+ [Survival](#survival)
	+ [Multivariate](#multivariate)
 - [Variable Importance](#variable_importance)
 - [Variance Estimation](#variance_estimation)
 - [Proximity Matrix Visualization](#proximity)

## <a name="install">Installation</a>

It is not yet on CRAN, but you can install it from Github using [devtools](http://cran.r-project.org/web/packages/devtools/index.html). 

```{r}
library(devtools)
install_github("zmjones/edarf")
```

## <a name="partial_dependence">Partial Dependence</a>
### <a name="classification">Classification</a>

```{r}
library(edarf)

data(iris)
library(randomForest)
fit <- randomForest(Species ~ ., iris)
pd <- partial_dependence(fit, df = iris, var = "Petal.Width", type = "prob")
plot_pd(pd, geom = "line")
```
![](http://zmjones.com/static/images/iris_pd_line.png)

```{r}
plot_pd(pd, geom = "area")
```
![](http://zmjones.com/static/images/iris_pd_area.png)

```{r}
pd <- partial_dependence(fit, df = iris, var = "Petal.Width")
plot_pd(pd, geom = "bar")
```
![](http://zmjones.com/static/images/iris_pd_bar.png)

```{r}
pd_int <- partial_dependence(fit, df = iris, var = c("Petal.Width", "Sepal.Length"),
                             interaction = TRUE, type = "prob")
plot_pd(pd_int, geom = "line")
```
![](http://zmjones.com/static/images/iris_pd_int_line.png)

```{r}
plot_pd(pd_int, geom = "area")
```
![](http://zmjones.com/static/images/iris_pd_int_area.png)

```{r}
pd_int <- partial_dependence(fit, df = iris, var = c("Petal.Width", "Sepal.Length"), interaction = TRUE)
plot_pd(pd_int, geom = "bar")
```
![](http://zmjones.com/static/images/iris_pd_int_bar.png)

```{r}
pd_lst <- partial_dependence(fit, df = iris, var = c("Petal.Width", "Sepal.Length"),
                             interaction = FALSE, type = "prob")
plot_pd(pd_lst, geom = "line")
```
![](http://zmjones.com/static/images/iris_pd_lst_line.png)

### <a name="regression">Regression</a>

```{r}
data(swiss)
fit <- randomForest(Fertility ~ ., swiss, keep.inbag = TRUE)
pd <- partial_dependence(fit, df = swiss, var = "Education")
plot_pd(pd)
```
![](http://zmjones.com/static/images/swiss_pd_line.png)

```{r}
pd_int <- partial_dependence(fit, df = swiss, var = c("Education", "Catholic"), interaction = TRUE)
plot_pd(pd_int)
```
![](http://zmjones.com/static/images/swiss_pd_int_line.png)

```{r}
pd_lst <- partial_dependence(fit, df = swiss, var = c("Education", "Catholic"),
                             interaction = FALSE, ci = TRUE)
plot_pd(pd_lst)
```
![](http://zmjones.com/static/images/swiss_pd_lst_line.png)

### <a name="survival">Survival</a>

```{r}
library(randomForestSRC)
data(veteran)
fit <- rfsrc(Surv(time, status) ~ ., veteran)
pd <- partial_dependence(fit, var = "age")
plot_pd(pd)
```
![](http://zmjones.com/static/images/veteran_pd_line.png)

```{r}
pd_int <- partial_dependence(fit, var = c("age", "diagtime"))
plot_pd(pd_int)
```
![](http://zmjones.com/static/images/veteran_pd_int_line.png)

### <a name="multivariate">Multivariate</a>

```{r}
data(mtcars)
library(party)
fit <- cforest(hp + qsec ~ ., mtcars, controls = cforest_control(mtry = 2))
pd <- partial_dependence(fit, var = "mpg")
plot_pd(pd)
```
![](http://zmjones.com/static/images/mtcars_pd_line.png)

Multivariate two-way interaction plots not yet implemented. See [Issue #19](https://github.com/zmjones/edarf/issues/19).

## <a name="variable_importance">Variable Importance</a>

```{r}
fit <- randomForest(Species ~ ., iris, importance = TRUE)
imp <- variable_importance(fit, type = "accuracy", class_levels = TRUE)
plot_imp(imp)
```
![](http://zmjones.com/static/images/iris_imp_class.png)

```{r}
plot_imp(imp, geom = "bar", horizontal = FALSE)
```
![](http://zmjones.com/static/images/iris_imp_class_bar.png)

```{r}
plot_imp(imp, geom = "bar", facet = TRUE)
```
![](http://zmjones.com/static/images/iris_imp_class_bar_facet.png)

```{r}
imp <- variable_importance(fit, type = "accuracy", class_levels = FALSE)
plot_imp(imp, horizontal = FALSE)
```
![](http://zmjones.com/static/images/iris_imp.png)

## <a name="variance_estimation">Variance Estimation</a>

To use `var_est` with `randomForest` we need an updated (but not merged) version of [randomForest]([randomForest](http://github.com/swager/randomForest)) which is installed below with `devtools`.

```{r}
library(devtools)
install_github("swager/randomForest")

fit <- randomForest(hp ~ ., mtcars, keep.inbag = TRUE)
out <- var_est(fit, mtcars)
```

All the below is handled internally if variance estimates are requested, for `partial_dependence`, however, it is possible to use `var_est` with the fitted model alone as well (regression with `randomForest`, `cforest`, and `rfsrc`)

```{r}
colnames(out)[1] <- "hp"

cl <- qnorm(.05 / 2, lower.tail = FALSE)
se <- sqrt(out$variance)
out$low <- out$hp - cl * se
out$high <- out$hp + cl * se
out$actual_hp <- mtcars$hp

library(ggplot2)
ggplot(out, aes(actual_hp, hp)) +
    geom_point() +
        geom_errorbar(aes(ymax = high, ymin = low), size = .5, width = .5) +
            geom_abline(aes(intercept = 0, slope = 1), colour = "blue") +
                labs(x = "Observed Horsepower", y = "Predicted Horsepower") +
                    theme_bw()
```
![](http://zmjones.com/static/images/mtcars_pred.png)

## <a name="proximity">Proximity Matrix Visualization</a>

```{r}
fit <- randomForest(hp ~ ., mtcars, proximity = TRUE)
prox <- extract_proximity(fit)
plot_prox(prox, labels = row.names(mtcars))
```
![](http://zmjones.com/static/images/prox_mtcars.png)

```{r}
fit <- randomForest(Species ~ ., iris, proximity = TRUE)
prox <- extract_proximity(fit)
plot_prox(prox, color = iris$Species, color_label = "Species", size = 2)
```

![](http://zmjones.com/static/images/prox_iris.png)
