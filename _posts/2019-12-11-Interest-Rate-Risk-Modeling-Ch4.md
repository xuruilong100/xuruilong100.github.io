---
title: M-absolute 和 M-square 风险度量
date: 2019-12-11 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [读书笔记, 利率风险, 固收]
description: 《Interest Rate Risk Modeling》第四章知识思维导图
---

# 《Interest Rate Risk Modeling》第四章：M-absolute 和 M-square 风险度量

![](/img/irrm/cover.jpg)

## 思维导图

> 从第四章开始比较难了
>
> $M^A$ 和 $M^2$ 控制了组合预期变化的下限

![](/img/irrm/ch4.png)

## 两个重要不等式的推导

首先有

$$
V_0 = \sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}
$$

令

$$
\begin{aligned}
V_H &= V_0 e^{\int_0^H f(s)ds}\\
    &= \sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds} e^{\int_0^H f(s)ds}\\
    &= \sum_{t=t_1}^{t_n} CF_t e^{\int_t^H f(s)ds}
\end{aligned}
$$

以及

$$
V_H^{\prime} = \sum_{t=t_1}^{t_n} CF_t e^{\int_t^H f^{\prime}(s)ds}
$$

那么

$$
\begin{aligned}
\frac{V_H^{\prime} - V_H}{V_H} &=
\frac{1}{V_0 e^{\int_0^H f(s)ds}}
\sum_{t=t_1}^{t_n} CF_t (e^{\int_t^H f^{\prime}(s)ds} - e^{\int_t^H f(s)ds})\\
&=\frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_t[e^{\int_t^H f(s)ds}(e^{\int_t^H \Delta f(s)ds}-1)]e^{-\int_0^H f(s)ds}\\
&=\frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_te^{-\int_0^t f(s)ds}(e^{\int_t^H \Delta f(s)ds}-1)
\end{aligned}
$$

记

$$
h(t) = \int_t^H \Delta f(s)ds
$$

### 关于 $M^A$ 的不等式

$\Delta f(t)$ 的边界分别是 $K_1$ 和 $K_2$，即 $K_1 \le \Delta f(t) \le K_2$

若 $H>t$ 时

$$
h(t) \ge K_1(H-t) = K_1 |t-H|
$$

若 $H \le t$ 时

$$
h(t) \ge -K_2(t-H) = -K_2|t-H|
$$

于是

$$
h(t) \ge \min(K_1, -K_2)|t-H|
$$

而

$$
\begin{aligned}
\min (K_1, -K_2) &= -\max(-K_1, K_2)\\
&\ge -\max(|K_1|, |K_2|)\\
&=-K_3
\end{aligned}
$$

则

$$
h(t) \ge -K_3|t-H|
$$

已知

$$
e^x - 1 \ge x
$$

那么

$$
\begin{aligned}
\frac{V_H^{\prime} - V_H}{V_H} & =\frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_te^{-\int_0^t f(s)ds}(e^{\int_t^H \Delta f(s)ds}-1)\\
& \ge \frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_te^{-\int_0^t f(s)ds}h(t)\\
& \ge \frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_te^{-\int_0^t f(s)ds} (-K_3|t-H|)\\
& = -K_3M^A
\end{aligned}
$$

### 关于 $M^2$ 的不等式

记

$$
\frac{d\Delta f(t)}{dt} = g(t) \le K_4
$$

那么

$$
\begin{aligned}
h(t) &= \int_t^H \Delta f(s)ds\\
& = t\Delta f(t)|_{t}^{H} - \int_t^H s g(s)ds\\
& = H\Delta f(H) - t\Delta f(t) - \int_t^H s g(s)ds\\
& = (H-t)\Delta f(H) + t\Delta f(H) - t\Delta f(t) - \int_t^H s g(s)ds\\
& = (H-t)\Delta f(H) + t\int_t^H g(s)ds - \int_t^H s g(s)ds\\
& = (H-t)\Delta f(H) + \int_t^H(t-s) g(s)ds\\
\end{aligned}
$$

若 $H>t$ 时

$$
\begin{aligned}
\int_t^H (t-s) g(s)ds & \ge \int_t^H (t-s) K_4ds\\
&=-K_4(t-H)^2/2
\end{aligned}
$$

若 $H \le t$ 时

$$
\begin{aligned}
\int_t^H (t-s) g(s)ds & = -\int_H^t (t-s) g(s)ds\\
& \ge -\int_H^t (t-s) K_4ds\\
& = -K_4(t-H)^2/2
\end{aligned}
$$

那么

$$
h(t) \ge (H-t)\Delta f(H) -K_4(t-H)^2/2
$$

已知

$$
e^x - 1 \ge x
$$

那么

$$
\begin{aligned}
\frac{V_H^{\prime} - V_H}{V_H} & =\frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_te^{-\int_0^t f(s)ds}(e^{\int_t^H \Delta f(s)ds}-1)\\
& \ge \frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_te^{-\int_0^t f(s)ds}h(t)\\
& \ge \frac{1}{V_0}\sum_{t=t_1}^{t_n} CF_te^{-\int_0^t f(s)ds} \left((H-t)\Delta f(H) -K_4(t-H)^2/2\right)\\
& = (H-D)\Delta f(H) -K_4M^2/2
\end{aligned}
$$

## 凸性效应（CE）和风险效应（RE）的推导

$$
\begin{aligned}
R(H) &= \frac{V_H^{\prime} - V_0}{V_0}\\
&=\frac{\sum_{t=t_1}^{t_n} CF_t e^{\int_t^H f^{\prime}(s)ds}}{\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}} - 1\\
&=\frac{\sum_{t=t_1}^{t_n} CF_t e^{\int_0^H f^{\prime}(s)ds - \int_0^t f^{\prime}(s)ds}}{\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}} - 1\\
&=\frac{e^{\int_0^H f^{\prime}(s)ds}\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}e^{-\int_0^t \Delta f(s)ds}}{\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}} - 1\\
&=\frac{e^{\int_0^H f(s) + \Delta f(s)ds}\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}e^{-\int_0^t \Delta f(s)ds}}{\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}} - 1\\
&=\frac{e^{\int_0^H f(s)}\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}e^{\int_t^H \Delta f(s)ds}}{\sum_{t=t_1}^{t_n} CF_t e^{-\int_0^t f(s)ds}} - 1\\
\end{aligned}
$$

令

$$
R_F(H) = e^{\int_0^H f(s)ds} - 1
$$

记

$$
k(t) = e^{\int_t^H \Delta f(s)ds}
$$

对 $k(t)$ 在 $H$ 做 Taylor 展开

$$
\begin{aligned}
k(t) &= e^{\int_t^H \Delta f(s)ds}\\
&= e^{-\int_H^t \Delta f(s)ds}\\
&= k(H) + (t-H)k'(H) + \frac{1}{2}(t-H)^2k''(H) + \varepsilon\\
&= 1 + (t-H)(-\Delta f(H)) + \frac{1}{2}(t-H)^2(\Delta f(H)^2 - \frac{d(\Delta f(t))}{dt}|_{t=H}) + \varepsilon\\
&= 1 + (t-H)(-\Delta f(H)) + \frac{1}{2}(t-H)^2(\Delta f(H)^2 - g(H)) + \varepsilon\\
\end{aligned}
$$

代入得到

$$
\begin{aligned}
R(H) &= R_F(H) + \gamma_1 (D-H) + \gamma_2 M^2 + \varepsilon\\
\gamma_1 &= -\Delta f(H)(1+R_F(H))\\
\gamma_2 &= \frac{1}{2}(1+R_F(H))(\Delta f(H)^2 - g(H))\\
\gamma_2 &= CE - RE\\
CE &= \frac{1}{2}(1+R_F(H))\Delta f(H)^2\\
RE &= \frac{1}{2}(1+R_F(H))g(H)
\end{aligned}
$$