---
title: 基于 LIBOR 模型用互换和利率期权进行对冲
date: 2020-01-29 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [读书笔记, 利率风险, 固收] # TAG names should always be lowercase
description: 《Interest Rate Risk Modeling》第八章知识思维导图
---

# 《Interest Rate Risk Modeling》第八章：基于 LIBOR 模型用互换和利率期权进行对冲

![](/img/irrm/cover.jpg)

## 思维导图

![](/img/irrm/ch8.png)

## 推导浮息债在重置日（reset date）的价格

记首个重置日 $T_0=0$ 观察到的即期期限结构是 $Y(t)$，对应零息债券的价格是，

$$
P(T_0,T_i) = e^{-Y(T_i)T_i}，i=1,\dots,n
$$

根据 LIBOR 远期利率的定义，

$$
\begin{aligned}
1 + \tau L(T_0,T_i,T_{i+1}) &= \frac{P(T_0,T_{i})}{P(T_0,T_{i+1})}\\
\tau L(T_0,T_i,T_{i+1}) &= \frac{P(T_0,T_{i}) - P(T_0,T_{i+1})}{P(T_0,T_{i+1})}
\end{aligned}
$$

面额是 $F$ 的浮息债在 $T_0$ 的预期现金流如下：

$$
\begin{aligned}
T_1&: CF_1 = F \times \tau \times L(T_0, T_0, T_1)\\
T_2&: CF_2 = F \times \tau \times L(T_0, T_1, T_2)\\
\vdots \\
T_n&: CF_n = F \times \tau \times L(T_0, T_{n-1}, T_n) + F\\
\end{aligned}
$$

这些现金流的贴现值是：

$$
\begin{aligned}
P &= \sum_{i=1}^n CF_i \times P(T_0,T_i)\\
&=\sum_{i=1}^n F \times \tau \times L(T_0, T_{i-1}, T_i) \times P(T_0,T_i) + F\times P(T_0,T_n)\\
&=\sum_{i=1}^n F \times \frac{P(T_0,T_{i-1}) - P(T_0,T_{i})}{P(T_0,T_{i})} \times P(T_0,T_i) + F\times P(T_0,T_n)\\
&=F\times P(T_0,T_0)\\
&=F
\end{aligned}
$$
