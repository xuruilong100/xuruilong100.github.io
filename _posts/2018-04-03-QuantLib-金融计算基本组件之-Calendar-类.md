---
title: QuantLib 金融计算——基本组件之 Calendar 类
date: 2018-04-03 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 Calendar 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `Calendar` 类

针对相应国家编制一套日历表用来推算假期、工作日和周末，这对于金融实务来说是一件基础又非常重要的事。

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.10
```

## `Calendar` 对象的构造

在 QuantLib 中可以很轻松的构造特定国家的日历表。例如，通过 `myCal = UnitedKingdom()` 构造英国的日历表，其他国家诸如美国、日本、加拿大等等也可以用类似的方式构造。在有些国家，不同的市场遵循不同的日历表，例如在中国，银行间市场和交易所市场遵循的日历表是不一样的（交易所市场在周六周日一定不开放，不管是否调休）。对于这一问题，可以通过配置特定参数将日历表细化到具体的市场上，例如中国银行间市场的日历表可以通过 `myCal = China(China.IB)` 构造。`China.IB` 和 `China.SSE` 是 quantlib-python 的预留特殊变量，分别表示中国的银行间市场和交易所市场。

## 一些常用的成员函数

下面是一些常用的成员函数：

* `isBusinessDay(d)`：布尔值，判断 `d` 是不是工作日。
* `isHoliday(d)`：布尔值，判断 `d` 是不是假期。
* `isWeekend(w)`：布尔值，判断 `w` 是不是周末（在有些国家，周末没有安排在周六周日）。
* `isEndOfMonth(d)`：布尔值，判断 `d` 是不是月末最后一个工作日。
* `endOfMonth(d)`：日期，返回 `d` 所在月的最后一个工作日。

例子 1：

```python
def CalendarTesting1():
    chinaCal = ql.China(ql.China.IB)
    saudiArabCal = ql.SaudiArabia()
    nyEve = ql.Date(3, ql.April, 2017)

    print('Is BD :', chinaCal.isBusinessDay(nyEve))
    print('Is Holiday :', chinaCal.isHoliday(nyEve))
    print('Is Weekend in SA :', saudiArabCal.isWeekend(ql.Friday))
    print('Is Weekend in CN :', chinaCal.isWeekend(ql.Friday))
    print('Is Last BD :',
          chinaCal.isEndOfMonth(ql.Date(5, ql.April, 2018)))
    print('Last BD :', chinaCal.endOfMonth(nyEve))
```

```
Is BD : False
Is Holiday : True
Is Weekend in SA : True
Is Weekend in CN : False
Is Last BD : False
Last BD : April 28th, 2017
```

注意，在沙特阿拉伯周五周六是“周末”，这和中国不一样。

### 自定义假期列表

QuantLib 对中国市场的支持比较有限，目前的版本假期列表仅仅维护到 2004-2017 年，要想正确推算其他年份的日历表，用户需要自行配置假期。QuantLib 中的 `Calendar` 对象可以方便的实现自定义假期，通常仅仅需要借助下列两个函数：

* `addHoliday(d)`：添加 `d` 为假期。
* `removeHoliday(d)`：从假期表中移除 `d` 。

将 2018 年清明节放假调休的安排配置到 `Calendar` 对象中。

例子 2：

```python
def CalendarTesting2():
    chinaCal = ql.China(ql.China.IB)

    d1 = ql.Date(5, ql.April, 2018)
    d2 = ql.Date(6, ql.April, 2018)
    d3 = ql.Date(8, ql.April, 2018)

    print('Is Business Day : ', chinaCal.isBusinessDay(d1))
    print('Is Business Day : ', chinaCal.isBusinessDay(d2))
    print('Is Business Day : ', chinaCal.isBusinessDay(d3))

    chinaCal.addHoliday(d1)
    chinaCal.addHoliday(d2)
    chinaCal.removeHoliday(d3)

    print('Is Business Day : ', chinaCal.isBusinessDay(d1))
    print('Is Business Day : ', chinaCal.isBusinessDay(d2))
    print('Is Business Day : ', chinaCal.isBusinessDay(d3))
```

```
Is Business Day :  True
Is Business Day :  True
Is Business Day :  False
Is Business Day :  False
Is Business Day :  False
Is Business Day :  True
```

### 工作日修正

将某个日期修正为工作日是一项必要的工作，QuantLib 中支持如下**工作日转换**模式：

* `Following`：将日期修正为之后出现的第一个工作日。
* `ModifiedFollowing`：将日期修正为之后出现的第一个工作日，除非这个工作日出现在次月；如果修正后的工作日出现在次月，就将日期修正为之前出现的最近一个工作日，保证原始日期和修正后的日期处在同一个月。
* `Preceding`：将日期修正为之前出现的最近一个工作日。
* `ModifiedPreceding`：将日期修正为之前出现的最近一个工作日，除非这个工作日出现在上一个月；如果修正后的工作日出现在上一个月，就将日期修正为之后出现的第一个工作日，保证原始日期和修正后的日期处在同一个月。
* `Unadjusted`：不作调整。

`Calendar` 对象通过下列两个函数实现修正日期的功能：

* `adjust(d, convention)`：日期，按照转换模式 `convention` 修正 `d`。
* `advance(d, period, convention, endOfMonth)`：日期，将日期 `date` 向后推移时间间隔 `period` 后再按照转换模式 `convention` 修正；参数 `endOfMonth` 表示，如果 `d` 是月末的话，推移修正后的日期也要是在月末。

最后，可以通过下面的函数计算两个日期间的工作日个数：

* `businessDaysBetween(from, to, includeFirst, includeLast)`：计算日期 `from` 和 `to` 之间的工作日个数（是否包括首尾日期）。

例子 3：

```python
def CalendarTesting3():
    chinaCal = ql.China(ql.China.IB)

    firstDate = ql.Date(31, ql.January, 2018)
    secondDate = ql.Date(1, ql.April, 2018)

    print('Date 2 Adj :', chinaCal.adjust(secondDate, ql.Preceding))
    print('Date 2 Adj :', chinaCal.adjust(secondDate, ql.ModifiedPreceding))

    mat = ql.Period(2, ql.Months)

    print('Date 1 Month Adv :',
          chinaCal.advance(firstDate, mat, ql.Following, False))
    print('Date 1 Month Adv :',
          chinaCal.advance(firstDate, mat, ql.ModifiedFollowing, False))

    print('Business Days Between :',
          chinaCal.businessDaysBetween(
              ql.Date(5, ql.March, 2018), ql.Date(30, ql.March, 2018),
              True, True))
```

```
Date 2 Adj : March 30th, 2018
Date 2 Adj : April 2nd, 2018
Date 1 Month Adv : April 2nd, 2018
Date 1 Month Adv : March 30th, 2018
Business Days Between : 20
```
