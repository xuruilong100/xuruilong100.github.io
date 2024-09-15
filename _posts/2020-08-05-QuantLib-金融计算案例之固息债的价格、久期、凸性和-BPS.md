---
title: QuantLib 金融计算——案例之固息债的价格、久期、凸性和 BPS
date: 2020-08-05 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍在 QuantLib 中计算固息债的价格、久期、凸性和 BPS。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——案例之固息债的价格、久期、凸性和 BPS

## 概述

从本篇开始计划开启一个系列，以《Interest Rate Risk Modeling》为蓝本，介绍有关利率风险的计算案例，内容涉及从简单的久期、凸性到主成分久期和久期向量模型等高阶的度量指标。

## 计算久期和凸性

固息债的久期、凸性和 BPS 是最常见的利率风险度量指标，下面将以 200205 为例，计算 2020-07-28 这一天的价格，以及久期、凸性和 BPS。

首先从[中国货币网](https://www.chinamoney.com.cn/chinese/)查询债券的基本信息，用以配置 `FixedRateBond` 对象。

* 债券起息日：2020-03-10
* 到期兑付日：2030-03-10
* 债券期限：10 年
* 面值(元)：100.00
* 计息基准：A/A
* 息票类型：附息式固定利率
* 付息频率：年
* 票面利率（%）：3.0700
* 结算方式：T+1

```python
import QuantLib as ql
import prettytable as pt

today = ql.Date(28, ql.July, 2020)
ql.Settings.instance().evaluationDate = today

settlementDays = 1
faceAmount = 100.0
```

`settlementDays = 1` 表示 T+1 结算，而估值日期就是 2020-07-28 这一天。

```python
effectiveDate = ql.Date(10, ql.March, 2020)
terminationDate = ql.Date(10, ql.March, 2030)
tenor = ql.Period(1, ql.Years)
calendar = ql.China(ql.China.IB)
convention = ql.Unadjusted
terminationDateConvention = convention
rule = ql.DateGeneration.Backward
endOfMonth = False

schedule = ql.Schedule(
    effectiveDate,
    terminationDate,
    tenor,
    calendar,
    convention,
    terminationDateConvention,
    rule,
    endOfMonth)

# for s in schedule:
#     print(s)

coupons = ql.DoubleVector(1)
coupons[0] = 3.07 / 100.0
accrualDayCounter = ql.ActualActual(
    ql.ActualActual.Bond, schedule)
paymentConvention = ql.Unadjusted

bond = ql.FixedRateBond(
    settlementDays,
    faceAmount,
    schedule,
    coupons,
    accrualDayCounter,
    paymentConvention)
```

需要注意的是，日历采用中国的银行间市场，遇到假期不调整。

如果像下面一样，采用基于期限结构的定价引擎，在构造 `ActualActual` 对象时要附加上债券现金流支付的日期表（`Schedule` 对象），否则在计算贴现因子的时候可能产生偏差，具体的讨论请查看 StackExchange 上的讨论：<https://quant.stackexchange.com/questions/12707/pricing-a-fixedratebond-in-quantlib-yield-vs-termstructure>

在[上海清算所](https://www.shclearing.com/)查询估值、价格和久期等数据，作为比较基准。

由于使用的是估值，也就是“到期利率”，这隐含要求于一个“水平”（flat）的期限结构，所以使用 `FlatForward` 类。对于水平的期限结构而言，远期利率、即期利率和到期利率三者相等。

`DiscountingBondEngine` 是最常见的债券定价引擎，主要用于现金流的贴现计算。

```python
bondYield = 3.4124 / 100.0

compounding = ql.Compounded
frequency = ql.Annual

termStructure = ql.YieldTermStructureHandle(
    ql.FlatForward(
        settlementDays,
        calendar,
        bondYield,
        accrualDayCounter,
        compounding,
        frequency))

engine = ql.DiscountingBondEngine(termStructure)
bond.setPricingEngine(engine)
```

价格信息可以通过 `FixedRateBond` 的成员函数获得，而久期等指标的计算在 `BondFunctions` 的内部函数中实现（`BondFunctions` 的内部函数也可以依据到期利率计算价格信息）。

```python
cleanPrice = bond.cleanPrice()
dirtyPrice = bond.dirtyPrice()
accruedAmount = bond.accruedAmount()

duration = ql.BondFunctions.duration(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency)

convexity = ql.BondFunctions.convexity(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency)

bps = ql.BondFunctions.basisPointValue(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency)

tab = pt.PrettyTable(['item', 'QuantLib', 'ShClearing'])
tab.add_row(['clean price', cleanPrice, 97.2211])
tab.add_row(['dirty price', dirtyPrice, 98.4071])
tab.add_row(['accrued amount', accruedAmount, 1.1859])
tab.add_row(['duration', duration, 8.0771])
tab.add_row(['convexity', convexity, 79.2206])
tab.add_row(['bps', abs(bps), 0.0795])

tab.float_format = '.4'

print(tab)
```

```
+----------------+----------+------------+
|      item      | QuantLib | ShClearing |
+----------------+----------+------------+
|  clean price   | 97.2212  |  97.2211   |
|  dirty price   | 98.4071  |  98.4071   |
| accrued amount |  1.1859  |   1.1859   |
|    duration    |  8.0771  |   8.0771   |
|   convexity    | 79.2206  |  79.2206   |
|      bps       |  0.0795  |   0.0795   |
+----------------+----------+------------+
```

最终结果和上海清算所公布的几乎一致。

![](/img/meme/ok.gif)

## 三种久期

`BondFunctions` 的 `duration` 函数可以计算三种久期，分别是简单久期（Simple）、麦考利久期（Macaulay）和修正久期（Modified），只需配置久期类型参数即可，默认计算的是修正久期。

程序实现上，麦考利久期的计算依赖于修正久期。

所谓简单久期，即现金流的期限关于现金流贴现值的加权平均。如果计息方式是复利，简单久期等于麦考利久期。不过，如果是连续复利，计算麦考利久期将会报错，简单久期依然可以计算出来，更有普适性。连续复利的情况下，简单久期等于修正久期。

```python
durationSimple = ql.BondFunctions.duration(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency,
    ql.Duration.Simple)

durationModified = ql.BondFunctions.duration(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency,
    ql.Duration.Modified)

durationMacaulay = ql.BondFunctions.duration(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency,
    ql.Duration.Macaulay)

tabDuration = pt.PrettyTable(['type', 'value'])
tabDuration.add_row(['Simple', durationSimple])
tabDuration.add_row(['Modified', durationModified])
tabDuration.add_row(['Macaulay', durationMacaulay])

print(tabDuration)
```

```
+----------+-------------------+
|   type   |       value       |
+----------+-------------------+
|  Simple  | 8.352745733674992 |
| Modified | 8.077122021802985 |
| Macaulay | 8.352745733674992 |
+----------+-------------------+
```
