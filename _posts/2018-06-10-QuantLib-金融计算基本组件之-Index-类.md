---
title: QuantLib 金融计算——基本组件之 Index 类
date: 2018-06-10 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 Index 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `Index` 类

## 概述

`Index` 类用于表示已知的指数或者利率，例如 Libor 或 Shibor。这些指数或利率的属性可能取决于若干个变量，如基础货币和期限。想象一下，一个交易员正在交易利率互换，浮动利率定为 3 个月的 Shibor，他肯定需要了解此利率以及基于此利率的互换合约的若干结算细节，通常来说这些细节是固定不变的。

这些属性因期限而不同，此外，还取决于相应的基础货币。幸运的是，用户不必为大多数常用指数或利率指定这些属性，因为它们在 QuantLib 中实现。

例如，用户可以通过 `Shibor` 类构造特定期限的 Shibor 利率。`Shibor` 类继承自 `IborIndex` 类（表示银行间市场利率的基类）， `IborIndex` 类又继承自 `InterestRateIndex` 类（表示指数和利率的基类）。这些类提供了如下几个常用函数：

* `name()`：指数或利率的名字
* `fixingCalendar()`：指数或利率使用的日历表
* `dayCounter()`：指数或利率使用的天数计算规则
* `currency()`：指数或利率对应的基础货币
* `tenor()`：指数或利率的期限
* `businessDayConvention()`：如何调整非工作日
* 

目前版本的 quantlib-python 中没有封装 `Shibor` 类，只能在 C++ 中调用。

```c++
#include <iostream>

#include <ql/indexes/ibor/shibor.hpp>
#include <ql/time/period.hpp>

using namespace std;
using namespace QuantLib;

void testingIndexes1();

int main()
{
    testingIndexes1();
    return 0;
}

void testingIndexes1()
{
    Period tensor(3, Months);
    Shibor index(tensor);

    cout << "Name :\t" << index.familyName() << endl;
    cout << "BDC :\t" << index.businessDayConvention() << endl;
    cout << "End of Month rule ?:\t" << index.endOfMonth() << endl;
    cout << "Tenor :\t" << index.tenor() << endl;
    cout << "Calendar :\t" << index.fixingCalendar() << endl;
    cout << "Day Counter :\t" << index.dayCounter() << endl;
    cout << "Currency :\t" << index.currency() << endl;
}
```

```
Name :	Shibor
BDC :	Modified Following
End of Month rule ?:	0
Tenor :	3M
Calendar :	China inter bank market
Day Counter :	Actual/360
Currency :	CNY
```

在 QuantLib 中有若干类的构造函数需要这样一个指数或利率对象作为输入参数，特别是和构造利率曲线有关的类，需要利率的确切属性。`Index` 类使得其他构造函数的定义更加紧凑。
