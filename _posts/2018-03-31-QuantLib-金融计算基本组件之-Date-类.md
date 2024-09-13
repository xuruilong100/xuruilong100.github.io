---
title: QuantLib 金融计算——基本组件之 Date 类
date: 2018-03-31 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 Date 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `Date` 类

QuantLib 将金融领域的日期对象抽象为 `Date` 类，并提供了丰富的计算函数。需要注意的是，quantlib-python 中的 `Date` 类并不同于 python 自身包含的 `datetime` 类，也没有继承关系。

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.10
```

## `Date` 对象的构造

`Date` 对象的构造方式有两种，分别是 

* **`Date(serialNumber)`**，其中 `serialNumber` 是一个整数，例如 24214，并且 1 对应 1899-12-31。这种用法和 Excel 中一样。（需要注意的是，在较新版本的 quantlib-python 中，`serialNumber` 的取值范围被限定在 367～109574，相应的日期范围是 1901-01-01 ～ 2199-12-31。）
* **`Date(d, m, y)`**，其中 `d` 和 `y` 是整数；`m` 是 quantlib-python 中预留的特殊对象，专门用来表示月份:
  * 一月：`January`（等于 1）
  * ...
  * 十二月：`December`（等于 12）

`Date` 对象可以和整数做运算，用来向前或向后移动特定天数；`Date` 对象也可以和 `Period` 对象做运算，用来向前或向后移动特定的时间间隔。

`Period` 对象的构造：
* **`Period(n, units)`**，其中 `n` 是时间间隔的个数；`units` 的取值范围是  quantlib-python 预留的四个特殊对象：`Days`、`Weeks`、`Months`、`Years`。

例子 1：

```python
def DateTesting1():

    myDate = ql.Date(12, ql.August, 2009)
    print(myDate)

    myDate = myDate + 1
    print(myDate)

    myDate = myDate + ql.Period(12, ql.Days)
    print(myDate)

    myDate = myDate - ql.Period(2, ql.Months)
    print(myDate)

    myDate = myDate - 1
    print(myDate)

    myDate = myDate + ql.Period(10, ql.Weeks)
    print(myDate)
```

```
August 12th, 2009
August 13th, 2009
August 25th, 2009
June 25th, 2009
June 24th, 2009
September 2nd, 2009
```

## 一些常用的成员函数

`Date` 类常用的成员函数有：

* `weekday()`：整数，返回星期对应的数字：
  * 星期日：1
  * ...
  * 星期六：7
* `dayOfMonth()`：整数，返回日期是所在月份的第几天
* `dayOfYear()`：整数，返回日期是所在年份的第几天
* `month()`：整数，返回日期对应的月份
* `year()`：整数，返回日期对应的年份
* `serialNumber()`整数，返回日期对应的天数（从 1899-12-31 开始）

例子 2：

```python
def DateTesting2():
    myDate = ql.Date(12, ql.August, 2017)

    print('Original Date :', myDate)
    print('Weekday :', myDate.weekday())
    print('Day of Month :', myDate.dayOfMonth())
    print('Day of Year :', myDate.dayOfYear())
    print('Month :', myDate.month())
    print('Year :', myDate.year())

    serialNum = myDate.serialNumber()

    print('Serial Number :', serialNum)
```

```
Original Date : August 12th, 2017
Weekday : 7
Day of Month : 12
Day of Year : 224
Month : 8
Year : 2017
Serial Number : 42959
```

## 一些常用的静态函数

`Date` 类也提供了一些有用的静态函数，例如用来判断是否闰年或者是否是月末。一些常用的静态函数如下：

* `Date.todaysDate()`：`Date` 对象，返回系统当前的日期
* `Date.minDate()`：`Date` 对象，返回 QuantLib 可表示的最小日期
* `Date.maxDate()`：`Date` 对象，返回 QuantLib 可表示的最大日期
* `Date.isLeap(y)`：布尔值，判断 `y` 是否闰年
* `Date.endOfMonth(d)`：`Date` 对象，返回日期 `d` 所在月份月末对应的日期
* `Date.isEndOfMonth(d)`：布尔值，判断 `d` 是否月末
* `Date.nextWeekday(d, w)`：`Date` 对象，返回日期 `d` 之后首个星期 `w` 对应的日期（例如 2018-03-12 之后第一个星期五）
* `Date.nthWeekday(n, w, m, y)`：`Date` 对象，返回所给月份 `m` 和年份 `y` 中的第 `n` 个星期 `w` 对应的日期（例如 2010 年七月的第三个星期三）

例子 3：

```python
def DateTesting3():
    print('Today :', ql.Date.todaysDate())
    print('Min Date :', ql.Date.minDate())
    print('Max Date :', ql.Date.maxDate())
    print('Is Leap :', ql.Date.isLeap(2011))
    print('End of Month :',
          ql.Date.endOfMonth(ql.Date(4, ql.August, 2009)))
    print('Is Month End :',
          ql.Date.isEndOfMonth(ql.Date(29, ql.September, 2009)))
    print('Is Month End :',
          ql.Date.isEndOfMonth(ql.Date(30, ql.September, 2009)))
    print('Next WD :',
          ql.Date.nextWeekday(ql.Date(1, ql.September, 2009), ql.Friday))
    print('n-th WD :',
          ql.Date.nthWeekday(3, ql.Wednesday, ql.September, 2009))
```

```
Today : March 30th, 2018
Min Date : January 1st, 1901
Max Date : December 31st, 2199
Is Leap : False
End of Month : August 31st, 2009
Is Month End : False
Is Month End : True
Next WD : September 4th, 2009
n-th WD : September 16th, 2009
```

## 为估值计算配置日期

有时候为了给金融产品定价，需要将估值计算发生的日期配置成特定日期。该金融产品可能依赖于其他产品，其他产品又在新的日期做定价。为了方便日期配置，QuantLib 提供了一个全局变量用来配置估值日期。`Settings.instance().evaluationDate` 返回的就是当前的估值日期，这一日期是可配置的。

例子 4：

```python
def DateTesting4():
    d = ql.Settings.instance().evaluationDate
    print('Eval Date :', d)

    ql.Settings.instance().evaluationDate = ql.Date(5, ql.January, 1995)
    d = ql.Settings.instance().evaluationDate
    print('New Eval Date :', d)
```

```
Eval Date : March 30th, 2018
New Eval Date : January 5th, 1995
```
