---
title: Why robust regression?
author: Obryan Poyser
date: '2019-09-02'
slug: robust-regression
categories:
  - statistics
  - r-Language
tags:
  - time series analysis
  - robust statistics
image:
  caption: ''
  focal_point: ''
math: true
---

For every researcher there is **that** moment when we question ourself of how much our findings could hold if we relax one or more assumptions. Specially in economics where we breath and eat regressions. 

Robust statistics is a set of procedures that aim to provide near ideal estimates even if some of the assumptions in which the estimation method relies are not met. Particularly, a robust regression represents an alternative to least square estimations that is less susceptible to the existance of "data contamination", that is, influential points (outliers or leverage) that are unlikely to match a normal distribution. 

Under the existance of heavy-tailed error distributions a OLS overemphasize the weight of those outliers, specifically at $e_t\sim1/T$. Having said that, a OLS estimator has the form:

$$
y_t=\boldsymbol{x_i'}\beta+e_t
$$

with an objective function $L(e_t)=e_t^2$

$$
\hat{\beta}=argmin_\beta=\sum_{t=1}^T(y_t-\boldsymbol{x_t'}\beta-e_t)^2=\sum_{t=1}^T(y_t-\hat{y_t})^2=\sum_{t=1}^T\hat{e}_t^2
$$

Here is when the "robust" appear, the idea is to reweight the error terms that are impacting the estimator by assigning them a lower power. Among the most relevant robust estimators we have the M^[M stands for Maximum Likelihood]-estimation 



$$
argmin=\sum_t^T f(y_t-\boldsymbol{x_{tj}}\beta_j)
$$

