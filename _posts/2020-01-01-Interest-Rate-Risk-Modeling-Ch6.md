---
title: 《Interest Rate Risk Modeling》第六章：用利率期货对冲
date: 2020-01-01 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [读书笔记, 利率风险, 固收]
description: 《Interest Rate Risk Modeling》第六章知识思维导图
---

# 用利率期货对冲

![](/img/irrm/cover.jpg)

## 思维导图

![](/img/irrm/ch6.png)

## 转换因子释疑

转换因子是特定假设下可交割券全价与标准券全价的比值，用该比值近似真实情况下可交割券全价与标准券全价的比例关系。

标准券有 6% 的票息率，每半年付息一次，发行日为期货合约到期月份的首个交割日。若发行日期限结构保持水平，利率为 6%，则标准券全价是 \$100，可交割券（到期月份要向下取整到 0 月）全价是，

$$
\frac{c \times 100}{2} \left(\frac{1}{0.03} - \frac{1}{0.03(1+0.03)^{2n}} \right) + \frac{100}{0.03(1+0.03)^{2n}}
$$

转换因子等于，

$$
CF_0 = \frac{c}{2} \left(\frac{1}{0.03} - \frac{1}{0.03(1+0.03)^{2n}} \right) + \frac{1}{0.03(1+0.03)^{2n}}
$$

其他情况类似。

因此，国债期货的机制可以理解为：以标准券为基准，为所有可交割券估价。若期货合约价格 $FP$，CTD 的转换因子是 $CF$，那么交割时多头相当于用净价 $FP \times CF$ 购买了 CTD。

## 利率期货久期向量的推导

### Eurodollar 期货的久期向量

未来 $s$ 年的 $t$ 年期远期利率 $f(s, s+t)$ 和瞬时远期利率 $f(s)$ 之间存在关系：

$$
f(s,s+t)t = \int_{s}^{s+t}f(x)dx\\
\Delta f(s,s+t)t = \int_{s}^{s+t}\Delta f(x)dx
$$

（连续复利）期货利率 $f^{\ast}$ 和远期利率 $f$ 之间存在“凸性修正”关系：

$$
f(s,s+t) = f^{\ast}(s,s+t) - \frac{1}{2} \sigma^2 s(s+t)
$$

所以

$$
\Delta f(s,s+t) = \Delta f^{\ast}(s,s+t)
$$

已知：

$$
CP = 1000000[1-(100-Q)/400]\\
q=100-Q\\
$$

那么

$$
\Delta CP = -2500 \times \Delta q
$$

如果（连续复利）期货利率由 $f^{\ast}(s,s+90/365)$ 变为 $f^{\ast \prime}(s,s+90/365)$（记 $\Delta f^\ast = f^{\ast \prime} - f^\ast$），那么

$$
\begin{aligned}
\Delta q &=q^{\prime} - q\\
&= (e^{f^{* \prime}(s,s+90/365) \times(90/365)} - e^{f^{*}(s,s+90/365)\times(90/365)})\times 400\\
&=e^{f^{*}(s,s+90/365)\times(90/365)}(e^{\Delta f^{*}(s,s+90/365)\times(90/365)} - 1)\times 400\\
\end{aligned}
$$

根据 $e^x - 1 \approx x$，

$$
\begin{aligned}
\Delta CP &= -2500 \times \Delta q\\
&=-1000000\times e^{f^{*}(s,s+90/365)\times(90/365)}(e^{\Delta f^{*}(s,s+90/365)\times(90/365)} - 1)\\
&\approx -1000000\times e^{f^{*}(s,s+90/365)\times(90/365)} \Delta f^{*}(s,s+90/365)\times(90/365)\\
&=-1000000\times ((100-Q)/400+1)\Delta f^{*}(s,s+90/365)\times(90/365)\\
\end{aligned}
$$

如果：

$$
\Delta y(t) = \Delta A_0 + \Delta A_1 t + \Delta A_2 t^2 + \Delta A_3 t^3 + \cdots\\
\Delta f(t) = \Delta A_0 + 2\Delta A_1 t + 3\Delta A_2 t^2 + 4\Delta A_3 t^3 + \cdots
$$

那么

$$
\begin{aligned}
&\Delta f^{*}(s,s+90/365)\times(90/365) \\
&= \Delta f(s,s+90/365)\times(90/365)\\
&=\int_{s}^{s+90/365} \Delta f(t)dt\\
&=\int_{s}^{s+90/365} \Delta A_0 + 2\Delta A_1 t + 3\Delta A_2 t^2 + 4\Delta A_3 t^3 + \cdots dx\\
&=\Delta A_0(90/365) + \Delta A_1\left[(s+90/365)^2-s^2 \right] + \Delta A_2\left[(s+90/365)^3-s^3 \right] + \Delta A_3\left[(s+90/365)^4-s^4 \right] + \cdots
\end{aligned}
$$

最终

$$
\frac{\Delta CP}{CP} = -D^f(1)\times \Delta A_0 -D^f(2)\times \Delta A_1 -D^f(3)\times \Delta A_2 + \cdots\\
\begin{aligned}
D^f(1) &= K(Q)\times(90/365)\\
D^f(2) &= K(Q)\times[(s+90/365)^2-s^2]\\
D^f(3) &= K(Q)\times[(s+90/365)^3-s^3]\\
\end{aligned}
\\
K(Q)=\left(1+\frac{100-Q}{400} \right) / \left(1-\frac{100-Q}{400} \right)=\frac{500-Q}{300+Q}
$$

### 国债期货的久期向量

记：

* $T$ = CTD 的剩余期限
* $C$ = CTD 的票息现金流（非年化）
* $F$ = CTD 的面额
* $CF$ = CTD 的转换因子
* $CP$ = CTD 的全价
* $\tau$ = 期货到期日与期货到期后债券首个付息日之间的距离
* $s$ = 期货到期日
* $n$ = 截止到期货到期日发生的付息次数
* $y(t)$：瞬时即期期限结构
* 默认付息两次（美式规则）

那么，国债期货的价格是：

$$
\begin{aligned}
FP &=\frac{1}{CF} (CP - AI)\\
&= \frac{1}{CF}
\left(
\sum_{t=0}^{2(T-s-\tau)}\frac{C}{e^{(s +\tau + t\times 0.5)\times y(s +\tau + t\times 0.5)}} +
\frac{F}{e^{T\times y(T)}}
\right)e^{s \times y(s)} -
\frac{C}{CF}\times \frac{0.5-\tau}{0.5}
\end{aligned}
$$

如果 $y$ 变化到 $y^{\prime}$（记 $\Delta y = y^{\prime}-y$），那么

$$
\begin{aligned}
\Delta FP &= FP^{\prime} - FP\\
&= \frac{1}{CF}
\left(
\sum_{t=0}^{2(T-s-\tau)}\frac{C}{e^{(s +\tau + t\times 0.5)\times y^{\prime}(s +\tau + t\times 0.5)}} +
\frac{F}{e^{T\times y^{\prime}(T)}}
\right)e^{s \times y^{\prime}(s)} \\
&\ \ \ \ -\frac{1}{CF}
\left(
\sum_{t=0}^{2(T-s-\tau)}\frac{C}{e^{(s +\tau + t\times 0.5)\times y(s +\tau + t\times 0.5)}} +
\frac{F}{e^{T\times y(T)}}
\right)e^{s \times y(s)}\\
&=\frac{1}{CF}\left(
\sum_{t=0}^{2(T-s-\tau)}\frac{C}{e^{(s +\tau + t\times 0.5)\times y(s +\tau + t\times 0.5)}}\left(e^{s\times \Delta y(s) - (s +\tau + t\times 0.5)\times\Delta y(s +\tau + t\times 0.5)} -1\right) +
\frac{F}{e^{T\times y(T)}}\left(e^{s\times \Delta y(s) - T\times\Delta y(T)} -1\right)
\right)e^{s \times y(s)}
\end{aligned}
$$

根据 $e^x - 1 \approx x$，

$$
\begin{aligned}
\Delta FP &\approx \\
&\frac{1}{CF}
\sum_{t=0}^{2(T-s-\tau)}\frac{C e^{s \times y(s)}}{e^{(s +\tau + t\times 0.5)\times y(s +\tau + t\times 0.5)}}
[{s\times \Delta y(s) - (s +\tau + t\times 0.5)\times\Delta y(s +\tau + t\times 0.5)} ] \\
&+ \frac{F e^{s \times y(s)}}{{T\times y(T)}}\left[{s\times \Delta y(s) - T\times\Delta y(T)} \right]
\end{aligned}
$$

如果：

$$
\Delta y(t) = \Delta A_0 + \Delta A_1 t + \Delta A_2 t^2 + \Delta A_3 t^3 + \cdots
$$

那么

$$
\begin{aligned}
&\Delta FP \approx \\
&\frac{1}{CF}
\sum_{t=0}^{2(T-s-\tau)}\frac{C e^{s \times y(s)}}{e^{(s +\tau + t\times 0.5)\times y(s +\tau + t\times 0.5)}}
\times\\
&\left\{s\times (\Delta A_0 + \Delta A_1 s + \Delta A_2 s^2 + \Delta A_3 s^3 +\cdots) - (s +\tau + t\times 0.5)\times \left[\Delta A_0 + \Delta A_1 (s +\tau + t\times 0.5) + \Delta A_2 (s +\tau + t\times 0.5)^2 + \Delta A_3 (s +\tau + t\times 0.5)^3 +\cdots \right] \right\} \\
&+\frac{F e^{s \times y(s)}}{{T\times y(T)}}\left[s\times (\Delta A_0 + \Delta A_1 s + \Delta A_2 s^2 + \Delta A_3 s^3+\cdots) - T\times (\Delta A_0 + \Delta A_1 T + \Delta A_2 T^2 + \Delta A_3 T^3+\cdots) \right]
\end{aligned}
$$

最终

$$
\begin{aligned}
\frac{\Delta FP}{FP} &\approx -D(1)\times \Delta A_0 -D(2)\times \Delta A_1 -D(3)\times \Delta A_2 - \cdots - D(M)\times \Delta A_{M-1} -\cdots\\
D(m)&= \frac{e^{s \times y(s)}}{CF \times FP}
\left(
\sum_{t=0}^{2(T-s-\tau)}\frac{C\left((s+\tau+t\times 0.5)^m - s^m \right)}{e^{(s +\tau + t\times 0.5)\times y(s +\tau + t\times 0.5)}} +
\frac{F(T^m - s^m)}{e^{T\times y(T)}}
\right)\\
m&=1,2,3,\dots,M
\end{aligned}
$$

## CME 上的 Eurodollar 教程

[Introduction to Eurodollars](https://www.cmegroup.com/education/courses/introduction-to-eurodollars.html)