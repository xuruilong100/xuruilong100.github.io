---
title: QuantLib 金融计算——基本组件之 InterestRate 类
date: 2018-06-24 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 InterestRate 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `InterestRate` 类

## 概述

围绕利率展开的若干计算（如计算贴现因子）是固定收益分析中最基础的部分。同时，由于固定收益产品在付息频率、计息方式、天数计算规则等细节方面的多样性，这一块的计算显得更加复杂繁琐。QuantLib 将与利率有关的计算整合封装在 `InterestRate` 类，用户所作的只是按照规定配置特定的参数。

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.12
```

## `InterestRate` 对象的构造

`InterestRate` 对象的构造需要四个参数，

```python
InterestRate(r,
             dc,
             comp,
             freq)
```

这些变量的类型和解释如下：
* `r`，浮点数，利率大小；
* `dc`，`DayCounter` 对象，配置天数计算规则；
* `comp`，整数，配置计息方式，取值范围是 quantlib-python 的一些预留变量；
* `freq`，整数，配置付息频率，取值范围是 quantlib-python 的一些预留变量。

目前 quantlib-python 支持的计息方式有：
* `Simple`，$1 + r\tau$，单利
* `Compounded`，$(1 + r)^\tau$，复利
* `Continuous`，$e^{r\tau}$，连续复利

目前 quantlib-python 支持的计息方式有很多：
* `NoFrequency`，无付息；
* `Once`，付息一次，常见于零息债券；
* `Annual`，每年付息一次；
* `Semiannual`，每半年付息一次；
* `EveryFourthMonth`，每 4 个月年付息一次；
* `Quarterly`，每季度付息一次；
* `Bimonthly`，每两个月付息一次；
* `Monthly`，每月付息一次；
* `EveryFourthWeek`，每 4 周付息一次；
* `Biweekly`，每两周付息一次；
* `Weekly`，每周付息一次；
* `Daily`，每天付息一次。

## 一些常用的成员函数

下面是一些常用的成员函数：

* `rate()`：浮点数，返回利率的值；
* `dayCounter()`：`DayCounter` 对象，返回控制天数计算规则的成员变量；
* `compounding()`：整数，返回计息方式；
* `frequency()`：整数，返回付息频率。
* `discountFactor(d1, d2)`：浮点数，`d1` 和 `d2` 都是 `Date` 型对象（`d1` < `d2`），返回 `d1` 到 `d2` 的贴现因子大小；
* `compoundFactor(d1, d2)`：浮点数，`d1` 和 `d2` 都是 `Date` 型对象（`d1` < `d2`），返回 `d1` 到 `d2` 的付息因子大小；
* `equivalentRate(resultDC, comp, freq, d1, d2)`：`InterestRate` 对象，返回某个与当前对象等价的 `InterestRate` 对象，该对象的配置参数包括 `resultDC`、`comp`、`freq`：
    * `d1` 和 `d2` 都是 `Date` 型对象（`d1` < `d2`）
    * `resultDC`，`DayCounter` 对象，配置天数计算规则；
    * `comp`，整数，配置计息方式，取值范围是 quantlib-python 的一些预留变量；
    * `freq`，整数，配置付息频率，取值范围是 quantlib-python 的一些预留变量。 

某些情况下需要根据付息因子的大小逆算利率，`InterestRate` 类提供了函数 `impliedRate` 实现这一功能：
* `impliedRate(compound, resultDC, comp, freq, d1, d2)`：`InterestRate` 对象，返回逆算出的 `InterestRate` 对象，该对象的配置参数包括 `resultDC`、`comp`、`freq`：
    * `d1` 和 `d2` 都是 `Date` 型对象（`d1` < `d2`）
    * `resultDC`，`DayCounter` 对象，配置天数计算规则；
    * `comp`，整数，配置计息方式，取值范围是 quantlib-python 的一些预留变量；
    * `freq`，整数，配置付息频率，取值范围是 quantlib-python 的一些预留变量。 

例子1：

```python
def InterestRate1():
    dc = ql.ActualActual()
    myRate = ql.InterestRate(
        0.0341, dc, ql.Simple, ql.Annual)

    print('Rate:', myRate)

    d1 = ql.Date(10, ql.September, 2009)
    d2 = d1 + ql.Period(3, ql.Months)
    compFact = myRate.compoundFactor(d1, d2)

    print('Compound Factor: ', compFact)
    print('Discount Factor: ', myRate.discountFactor(d1, d2))
    print(
        'Equivalent Rate: ',
        myRate.equivalentRate(
            dc, ql.Continuous, ql.Semiannual, d1, d2))

    implRate = ql.InterestRate.impliedRate(
        compFact, dc, ql.Simple, ql.Annual, d1, d2)

    print('Implied Rate from Comp Fact : ', implRate)


InterestRate1()
```

```
Rate: 3.410000 % Actual/Actual (ISDA) simple compounding
Compound Factor:  1.0085016438356165
Discount Factor:  0.9915700248109837
Equivalent Rate:  3.395586 % Actual/Actual (ISDA) continuous compounding
Implied Rate from Comp Fact :  3.410000 % Actual/Actual (ISDA) simple compounding
```
