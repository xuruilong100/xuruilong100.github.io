---
title: QuantLib 金融计算——一个使用 ActualActual 时需要注意的陷阱
date: 2020-11-15 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: ActAct 是一个非常特殊的天数计算规则，这里记录一个它的使用陷阱。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——一个使用 `ActualActual` 时需要注意的陷阱

`ActualActual` 是分析债券时最常用的 day counter（天数计算规则），根据 StackExchange 上的讨论（<https://quant.stackexchange.com/questions/12707/pricing-a-fixedratebond-in-quantlib-yield-vs-termstructure>），在使用 `ActualActual` 时最好附加上债券现金流支付的日期表（`Schedule` 对象），否则在计算贴现因子的时候可能产生偏差。

然而，这种操作隐藏了一个陷阱，下面用一个案例来解释。

```python
import QuantLib as ql

print(ql.__version__)

today = ql.Date(10, ql.November, 2020)
ql.Settings.instance().evaluationDate = today

effectiveDate = ql.Date(21, ql.May, 2019)
terminationDate = ql.Date(21, ql.May, 2029)
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

scheduleEx = ql.Schedule(
    effectiveDate,
    ql.Date(21, ql.May, 2031),
    tenor,
    calendar,
    convention,
    terminationDateConvention,
    rule,
    endOfMonth)

dayCounter = ql.ActualActual(
    ql.ActualActual.Bond, schedule)

print('results 1:')
print(dayCounter.yearFraction(today, today + ql.Period(8, ql.Years)))
print(dayCounter.yearFraction(today, today + ql.Period(9, ql.Years)))
print(dayCounter.yearFraction(today, today + ql.Period(10, ql.Years)))

dayCounterEx = ql.ActualActual(
    ql.ActualActual.Bond, scheduleEx)

print('results 2:')
print(dayCounterEx.yearFraction(today, today + ql.Period(8, ql.Years)))
print(dayCounterEx.yearFraction(today, today + ql.Period(9, ql.Years)))
print(dayCounterEx.yearFraction(today, today + ql.Period(10, ql.Years)))

'''
1.20
results 1:
8.0
8.526027397260274
8.526027397260274
results 2:
8.0
9.0
10.0
'''
```

如果计算涉及到的日期超过了日期表的范围，即 `Schedule` 前两个参数确定的范围，日期计算可能产生与预期不一致的错误。

一个简便有效的解决方法是为债券对象和 `ActualActual` 分别提供各自的 `Schedule` 对象，`ActualActual` 的 `Schedule` 对象范围更大一点，以便包括所有可能涉及的日期，就像案例中的做法一样。
