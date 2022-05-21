---
title: Optimal lag length in VAR models
author: Obryan Poyser
date: '2019-03-19'
slug: visual-selection-of-optimal-lag-length-in-var-models
categories:
  - Statistics
tags:
  - VAR
  - Time series analysis
type: post
description: ""
---

Recently, I have been playing with vector autoregressive models. During the model specification and "sanity checks" one has to choose model order, that is, how many LHS lags introduce in the multi-equation model.

The most common approach for lag order selection is to inspect among different information criteria and choose the model that minimizes these indicators. There are several Information Criterion alternatives, and they vary on the weight they put on prediction error and parameters. For instance, Schwarz-Bayes (SC or BIC) over penalized big models (several estimated parameters) in comparison to Akaike (AIC). Therefore, there is always on researchers' hand to choose the order according to the different IC options. But there is little "issue", different IC, have unequal units, therefore, they are not directly comparable, this is actually not a huge deal, but I just find out it is a good idea to normalize the outputs for each lag order as a way to have comparable units.

$$
IC_{i}^{norm}=\frac{IC_i-min(IC_i)}{max(IC_i)-min(IC_i)}
$$
```{r, echo=FALSE, message=FALSE, warning=FALSE}
require(vars)
require(xts)
require(tidyverse)
data=readr::read_rds("data/ic.rds")
```

Using `VARselect` function from the `vars` package brings the following output

```{r}
vars::VARselect(data)$criteria
```

Then again, it is not a big deal searching through the columns and finding the minimum rowwise. For STATA users this is more intuitive, since it is organized columnwise like:

```{r}
vars::VARselect(data)$criteria %>% t()
```

But in the end, they lack an easier comparable point, then, why not normalizing them? First, let's create a simple function for normalized data:

```{r}
normF <- function(x){
    (x-min(x, na.rm = T))/(max(x, na.rm = T)-min(x, na.rm = T))
}
```

Then, start from a over-amplified (high frequency data has its pros) selection of orders, and some "tidying" we can create the following graph:

```{r, fig.align='center', warning=FALSE}
VARselect(data, lag.max = 40, type = "both")$criteria %>%
    t() %>%
    as_tibble() %>%
    tibble::rownames_to_column(var = "lag") %>%
    set_names(nm = c("lag", "AIC", "HQ", "SC", "FPE")) %>%
    mutate_at(.vars = vars(-lag), ~normF(.)) %>%
    mutate(lag=as.numeric(lag)) %>%
    gather(key = "IC", value = "value", -lag) %>%
    group_by(IC) %>%
    mutate(diff=tsibble::difference(value)) %>%
    gather(key = "key", value = "value", -lag, -IC) %>%
    ggplot(aes(lag, value, col=IC))+
    geom_line() +
    facet_wrap(~key, ncol = 1, scales = "free")

```

From the graph below we can easily conclude that indeed, Schwarz-Bayes Criterion (SC) penalized big models, same as Hannan-Quinn Criterion, being 2 and 6 lags respectively, whereas Akaike IC and Akaike’s Final Prediction Error Criterion (FPE) coincide in 10 lags.

Another way to help in the decision is to select the number of lags in which the sequential difference stabilizes. Which actually happens around 8 lags, which could be a more parsimonious model.
