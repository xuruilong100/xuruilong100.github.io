---
title: QuantLib 金融计算——基本组件之 DayCounter 类
date: 2018-04-07 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 DayCounter 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `DayCounter` 类

“天数计算规则”（Day Count Convention）对金融产品的估值至关重要，尤其是对固定收益类的产品。QuantLib 提供了下列常见的规则：

* `Actual360`：Actual / 360
* `Actual365Fixed`：Actual / 365（Fixed）
* `ActualActual`：Actual / Actual
* `Business252`：Business / 252
* `Thirty360`：30 / 360

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.10
```

## `DayCounter` 对象的构造

`DayCounter` 对象的构造非常简便，例如，要构造 **Actual / 360** 规则的对象，只需 `myCounter = Actual360()` 即可。有些规则还允许针对具体的产品和市场配置相应的参数，例如针对美国国债市场使用的 **Act / Act ISMA** 规则可以这样构造：`myCounter = ActualActual(ActualActual.ISMA)`，其中 `ActualActual.ISMA` 是 quantlib-python 预留的特殊变量。

## 一些常用的成员函数

常用的成员函数有两个：

* `dayCount(d1, d2)`：计算 `d1`，`d2` 之间的天数。
* `yearFraction(d1, d2)`：将 `d1`，`d2` 之间的天数年化。

例子 1：

```python
def DayCounterTesting1():
    dc = ql.Thirty360()

    d1 = ql.Date(1, ql.March, 2018)
    d2 = d1 + ql.Period(2, ql.Months)

    print('Days Between d1 / d2:', dc.dayCount(d1, d2))
    print('Year Fraction d1 / d2:', dc.yearFraction(d1, d2))
    print('Actual Days Between d1 / d2:', d2 - d1)
    print('d1:', d1)
    print('d2:', d2)
```

```
Days Between d1 / d2: 60
Year Fraction d1 / d2: 0.16666666666666666
Actual Days Between d1 / d2: 61
d1: March 1st, 2018
d2: May 1st, 2018
```

2018-03-01 向后推移**两个月**后是 2018-05-01，两者的实际距离是 61 天。但在 **Thirty / 360** 的规则下，每年有 360 天，每月有 30 天，所以两者的距离是 60 天，60 天年化后的结果是 60 / 360 = 1 / 6。
