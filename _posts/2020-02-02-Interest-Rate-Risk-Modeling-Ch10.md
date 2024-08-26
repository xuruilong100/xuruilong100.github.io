---
title: 主成分模型与 VaR 分析
date: 2020-02-02 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [读书笔记, 利率风险, 固收]
description: 《Interest Rate Risk Modeling》第十章知识思维导图
---

# 《Interest Rate Risk Modeling》第十章：主成分模型与 VaR 分析

![](/img/irrm/cover.jpg)

## 思维导图

![](/img/irrm/ch10.png)

## 一些想法

* NS 家族模型的参数有经济意义，同时参数变化的行为类似主成分，考虑基于 NS 模型参数的风险度量。
* 尝试用（多元）GARCH 滤波利率变化，对残差应用 PCA。

## 推导 PCD、PCC 和 KRD、KRC 的关系

利用主成分系数矩阵的正交性。

### PCD 和 KRD

$$
\begin{aligned}
PCD(i) &= -\frac{1}{P} \frac{\partial P}{\partial c^{\ast}_i}\\&= -\sqrt{\lambda_i} \frac{1}{P} \frac{\partial P}{\partial c_i}\\
&=-\sqrt{\lambda_i} \frac{1}{P} \frac{\partial P}{\partial c_i} \sum_{j=1}^k \mu_{ij}^2\\
&=-\sqrt{\lambda_i} \frac{1}{P} \sum_{j=1}^k \frac{\partial P}{\partial c_i} \mu_{ij}^2\\
&=-\sqrt{\lambda_i} \frac{1}{P} \sum_{j=1}^k \frac{\partial P}{\partial c_i} \frac{\partial c_i}{\partial y(t_j)} \mu_{ij}\\
&=- \sqrt{\lambda_i} \frac{1}{P} \sum_{j=1}^k \frac{\partial P}{\partial y(t_j)} \mu_{ij}\\
&=\sqrt{\lambda_i}\sum_{j=1}^k KRD(j) \mu_{ij}\\
&=\sum_{j=1}^k KRD(j) l_{ji}
\end{aligned}
$$

### PCC 和 KRC

$$
\begin{aligned}
PCC(i,j) &=  -\frac{1}{P} \frac{\partial^2 P}{\partial c^{\ast}_i \partial c^{\ast}_j}\\
&=-\sqrt{\lambda_i}\sqrt{\lambda_j}\frac{1}{P} \frac{\partial^2 P}{\partial c_i \partial c_j}\\
\end{aligned}
$$

其中

$$
\begin{aligned}
\frac{\partial^2 P}{\partial c_i \partial c_j}&=
\frac{\partial\left(\frac{\partial P}{\partial c_i}\right)}{\partial c_j}\\
&=\frac{\partial\left(\sum_{l=1}^k \frac{\partial P}{\partial y(t_l)} \mu_{il}\right)}{\partial c_j}\\
&=\sum_{l=1}^k \frac{\partial^2 P}{\partial y(t_l) \partial c_j} \mu_{il}\\
\end{aligned}
$$

又有

$$
\begin{aligned}
\frac{\partial^2 P}{\partial y(t_l) \partial c_j}&=
\frac{\partial^2 P}{\partial y(t_l) \partial c_j} \sum_{n=1}^k \mu_{jn}^2\\
&=\sum_{n=1}^k \frac{\partial^2 P}{\partial y(t_l) \partial c_j} \mu_{jn}^2\\
&=\sum_{n=1}^k \frac{\partial^2 P}{\partial y(t_l) \partial c_j} \frac{\partial c_j}{\partial y(t_n)} \mu_{jn}\\
&=\sum_{n=1}^k \frac{\partial^2 P}{\partial y(t_l) \partial y(t_n)} \mu_{jn}\\
\end{aligned}
$$

所以

$$
\begin{aligned}
\frac{\partial^2 P}{\partial c_i \partial c_j}&=
\sum_{l=1}^k \sum_{n=1}^k \frac{\partial^2 P}{\partial y(t_l) \partial y(t_n)} \mu_{jn} \mu_{il}
\end{aligned}
$$

最终

$$
\begin{aligned}
PCC(i,j) &=  -\sqrt{\lambda_i}\sqrt{\lambda_j}\frac{1}{P} \sum_{l=1}^k \sum_{n=1}^k \frac{\partial^2 P}{\partial y(t_l) \partial y(t_n)} \mu_{jn} \mu_{il}\\
&=\sum_{l=1}^k \sum_{n=1}^k KRC(l,n) l_{nj}l_{li}
\end{aligned}
$$
