---
title: QuantLib 金融计算——比较几种生成 Sobol 序列的方向数
date: 2021-12-04 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: Sobol 序列的方向数影响了所产生随机数的质量，这里比较了 QuantLib 中内置的方向数配置。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——比较几种生成 Sobol 序列的方向数

## 概述

Sobol 序列因方向数的选取而不同，下面比较一下 QuantLib 中 10 种方向数配置所产生的 Sobol 序列。

QuantLib 提供 10 种方向数配置，分别是：
* `Jaeckel`：理论支持的最大维度为 **32**，来源于文献 *Monte Carlo Methods in Finance（by Peter Jäckel）*；
* `SobolLevitan`：理论支持的最大维度为 **40**，来源于文献 *Algorithm 659: Implementing Sobol's quasirandom sequence generator*；
* `SobolLevitanLemieux`：理论支持的最大维度为 **360**，来源于文献 *RandQMC user's guide - A package for randomized quasi-Monte Carlo methods in C*；
* `JoeKuoD5`：理论支持的最大维度为 **2000**，来源于文献 *Constructing Sobol sequences with better two-dimensional projections*；
* `JoeKuoD6`：理论支持的最大维度为 **21201**，来源于文献 *Constructing Sobol sequences with better two-dimensional projections*；
* `JoeKuoD7`：理论支持的最大维度为 **1900**，来源于文献 *Constructing Sobol sequences with better two-dimensional projections*；
* `Kuo`：理论支持的最大维度为 **4926**，来源于文献 *Constructing Sobol sequences with better two-dimensional projections*；
* `Kuo2`：理论支持的最大维度为 **3947**，来源于文献 *Constructing Sobol sequences with better two-dimensional projections*；
* `Kuo3`：理论支持的最大维度为 **4587**，来源于文献 *Constructing Sobol sequences with better two-dimensional projections*；
* `Unit`：来源于文献 *Monte Carlo Methods in Finance（by Peter Jäckel）*。

如果序列的维度超过了理论支持的最大维度，剩余维度上的方向数由伪随机数（Mersenne twister）填充。

更多关于 `JoeKuoD5`、`JoeKuoD6`、`JoeKuoD7`、`Kuo`、`Kuo2` 和 `Kuo3` 的细节请查看：<https://web.maths.unsw.edu.au/~fkuo/sobol/>

## 数值实验

实验案例：算术平均亚式看涨期权。

随机过程和期权参数配置：
* $s$：100.0
* $q$：0.0
* $r$：0.09
* $v$：0.2
* 执行价：95.0
* 时间长度：365 天、30 天

实验路径数：1000、2000、3000、4000、5000、10000、20000、50000、100000

### 长期期权实验结果

* 不使用布朗桥

![](/img/QuantLib/sobol/long-no-bridge.png)

`Jaeckel`、`SobolLevitan` 和 `SobolLevitanLemieux` 三个低维度算法的表现明显好于其他。

`Unit` 的表现非常糟糕，尽管表现出收敛的态势，但 100000 路径模拟结果是 18.16，而准确值则是 9.997（文献【1】）。

![](/img/QuantLib/sobol/unit.png)

> 注：NAG，SciPy 和 Julia 中的 Sobol 序列使用了 `JoeKuoD6` 的方向数配置：
> * https://www.nag.com/numeric/cl/nagdoc_cl24/html/G05/g05ylc.html
> * https://docs.scipy.org/doc/scipy/reference/reference/generated/scipy.stats.qmc.Sobol.html
> * https://www.juliapackages.com/p/sobol
> 
> 但 `JoeKuoD6` 的表现居然不及三个需要随机初始化的低维算法。
> 
> ![](/img/meme/confuse.jpg)

* 使用布朗桥

![](/img/QuantLib/sobol/long-bridge.png)

使用布朗桥之后，几种配置没有明显差别，`Unit` 的表现也有很大改善。

![](/img/QuantLib/sobol/unit-bridge.png)

### 短期期权实验结果

* 不使用布朗桥

![](/img/QuantLib/sobol/short-no-bridge.png)

* 使用布朗桥

![](/img/QuantLib/sobol/short-bridge.png)

对于短期期权，无论是否使用布朗桥，各个方法没有明显差别。

## 结论

尽管其他知名软件包选择使用了 `JeoKuoD6` 的配置，但 QuantLib 当前的默认选项 `Jaeckel` 可能已然是最好的选择。（欢迎加入讨论：<https://github.com/lballabio/QuantLib/issues/1219>）

此外，布朗桥的使用真可谓“化腐朽为神奇”。

## 扩展阅读

* 几种方向数配置之间其他方面的比较请查看文献【2】
* [《Sobol 序列并行化的实践经验》](https://xuruilong100.github.io/posts/Sobol-%E5%BA%8F%E5%88%97%E5%B9%B6%E8%A1%8C%E5%8C%96%E7%9A%84%E5%AE%9E%E8%B7%B5%E7%BB%8F%E9%AA%8C/)

## 参考文献

1. Lo, Chien-Ling, Kenneth J. Palmer, and Min-Teh Yu. "Moment-matching approximations for Asian options." The Journal of Derivatives 21.4 (2014): 103-122.
2. Sobol', Ilya M., et al. "Construction and comparison of high‐dimensional Sobol'generators." Wilmott 2011.56 (2011): 64-79.
