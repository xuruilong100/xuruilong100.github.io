---
title: QuantLib 金融计算——基本组件之 Schedule 类
date: 2018-04-17 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 Schedule 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `Schedule` 类

## 概述

`Schedule` 类用于构造一个特定的日期列表，例如债券的付息日列表，是 QuantLib 中固定收益类产品分析最常用到的组件。

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.10
```

## `Schedule` 对象的构造

`Schedule` 类对象的构造依赖于之前介绍的几个基本组件。

```python
Schedule(effectiveDate ,
         terminationDate ,
         tenor,
         calendar,
         convention,
         terminationDateConvention,
         rule,
         endOfMonth,
         firstDate = Date (),
         nextToLastDate = Date ())
```

这些变量的类型和解释如下：

* `effectiveDate`、`terminationDate`：日期，日历列表的起始、结束日期，例如债券的起息日和到期日。
* `tenor`：`Period` 对象，相邻两个日期的间隔，例如债券的付息频率（1 年或 6 个月）或利率互换的利息重置频率（3 个月）。
* `calendar`：日历表，生成日期所遵循的特定日历表。
* `convention`：整数，如何调整非工作日（除最后一个日期外），取值范围是 quantlib-python 的一些预留变量。
* `terminationDateConvention`：整数，如果最后一个日期是非工作日，该如何调整，取值范围是 quantlib-python 的一些预留变量。
* `rule`：`DateGeneration` 的成员，生成日期的规则。
* `endOfMonth`：如果起始日期在月末，是否要求其他日期也要安排在月末（除最后一个日期外）。
* `firstDate`, `nextToLastDate`（可选）：日期，专门为生成方法 `rule` 提供的起始、结束日期（不常用）。

## 作为“容器”的 `Schedule` 对象

`Schedule` 对象的行为和 Python 中的 `list` 非常相似，作为一种存储 `Date` 对象的序列容器存在。因此下面两个函数是可用的：

* `len(sch)`：返回 `Schedule` 对象 `sch` 内日期的个数。
* `[i]`：返回第 `i` 个日期。

作为序列容器，和 `list` 一样，`Schedule` 对象也是可迭代的。

假设想要获得 2017 年每月首个工作日的列表：

* 起始、结束日期分别是 2017-01-01 和 2017-12-01。
* 时间间隔是一个月。
* 日历表遵循中国银行间市场的规定
* 遇到非工作日就递延到下一工作日

例子 1：

```python
def testingSchedule1():
    effectiveDate = ql.Date(1, ql.January, 2017)
    terminationDate = ql.Date(1, ql.December, 2017)
    tenor = ql.Period(1, ql.Months)
    calendar = ql.China(ql.China.IB)
    convention = ql.Following
    terminationDateConvention = ql.Following
    rule = ql.DateGeneration.Forward
    endOfMonth = False

    mySched = ql.Schedule(
        effectiveDate,
        terminationDate,
        tenor,
        calendar,
        convention,
        terminationDateConvention,
        rule,
        endOfMonth)

    for i in range(len(mySched)):
        print(mySched[i])
    
    print('------')
    
    for i in mySched:
        print(i)
```

```
January 3rd, 2017
February 3rd, 2017
March 1st, 2017
April 1st, 2017
May 2nd, 2017
June 1st, 2017
July 3rd, 2017
August 1st, 2017
September 1st, 2017
October 9th, 2017
November 1st, 2017
December 1st, 2017
------
January 3rd, 2017
February 3rd, 2017
March 1st, 2017
April 1st, 2017
May 2nd, 2017
June 1st, 2017
July 3rd, 2017
August 1st, 2017
September 1st, 2017
October 9th, 2017
November 1st, 2017
December 1st, 2017
```

## 一些常用的成员函数

* `until(d)`：从日期列表中截取前半部分，并保证最后一个日期是 `d`。
* `isRegular(i)`：判断第 `i` 个区间是否完整。这个概念需要解释以下：如果一个 `Schedule` 对象含有 `n` 个日期，那么这个对象就含有 `n-1` 个区间。如果第 `i` 个区间的长度和事先规定的时间间隔一致，那么这个区间就是完整的（Regular）。

例子 2：

```python
def testingSchedule2():
    effectiveDate = ql.Date(1, ql.January, 2017)
    terminationDate = ql.Date(1, ql.December, 2017)
    tenor = ql.Period(1, ql.Months)
    calendar = ql.China(ql.China.IB)
    convention = ql.Following
    terminationDateConvention = ql.Following
    rule = ql.DateGeneration.Forward
    endOfMonth = False

    mySched = ql.Schedule(
        effectiveDate,
        terminationDate,
        tenor,
        calendar,
        convention,
        terminationDateConvention,
        rule,
        endOfMonth)

    mySched = mySched.until(ql.Date(15, ql.June, 2017))

    for i in mySched:
        print(i)

    print('------')

    for i in range(len(mySched) - 1):
        print('{}-th internal is regular? {}'.format(
            i + 1, mySched.isRegular(i + 1)))
```

```
January 3rd, 2017
February 3rd, 2017
March 1st, 2017
April 1st, 2017
May 2nd, 2017
June 1st, 2017
June 15th, 2017
------
1-th internal is regular? True
2-th internal is regular? True
3-th internal is regular? True
4-th internal is regular? True
5-th internal is regular? True
6-th internal is regular? False
```

最后一个区间的长度只有 15 天，所以是“不完整的”。
