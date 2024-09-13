---
title: QuantLib 金融计算——基本组件之 Currency 类
date: 2019-04-08 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 Currency 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `Currency` 类

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.15
```

## 概述

QuantLib 中描述货币基本信息的类是 `Currency` 及其派生类，`Currency` 的体系很庞杂，但层次结构很简单。整个类的继承结构分为两层：`Currency` 作为唯一的基类；一种货币对应一个派生类，例如表示美元的 `USDCurrency`，表示人民币的 `CNYCurrency` 以及表示日元的 `JPYCurrency`，等等。

## 构造函数

`Currency` 及其派生类的构造函数不接受参数，需要注意的是，基类 `Currency` 是可以实例化的。

## 成员函数

`Currency` 及其派生类的成员函数基本上作为类的检查器出现，直白地提供一些描述性的信息。

常用成员函数如下：

* `name()`：返回字符串，货币的名称；
* `code()`：返回字符串，货币的 ISO4217 代码，通常是三个大写英文字母；
* `numericCode()`：返回整数，货币的 ISO4217 代码对应的数字；
* `symbol()`：返回字符串，即现实世界中常用于表示该货币的一个符号，美元的话就是“$”，日元的话就是“￥”。需要注意的是，该函数返回的可能是 Unicode，在 python 中可能导致程序运行失败；
* `fractionSymbol()`：返回字符串，即现实世界中常用于表示该货币最小单位的一个符号，和 `symbol()` 一样，该函数返回的可能是 Unicode，在 python 中可能导致程序运行失败；
* `fractionsPerUnit()`：返回整数，一单位货币相对于该货币最小单位的倍数，通常是 100。
* `format()`：返回字符串，一个用于格式化打印结果的“格式化字符串”。
* `empty()`：返回布尔值，如果对象由派生类实例化，则返回 `True`；如果对象由 `Currency` 实例化，则返回 `False`，毕竟基类对象中货币信息是“空”的。
* `rounding()`：返回一个 `Rounding` 对象，即该货币舍入的规则，默认不进行舍入。

示例，

```python
import QuantLib as ql

usd = ql.USDCurrency()
cny = ql.CNYCurrency()

print('{0:<20}{1}'.format('USDCurrency', 'CNYCurrency'))

print('{0:<20}{1}'.format(usd.name(), cny.name()))
print('{0:<20}{1}'.format(usd.code(), cny.code()))
print('{0:<20}{1}'.format(usd.numericCode(), cny.numericCode()))
print('{0:<20}{1}'.format(usd.symbol(), cny.symbol()))
# print('{0:<20}{1}'.format(usd.fractionSymbol(), cny.fractionSymbol()))
print('{0:<20}{1}'.format(usd.fractionsPerUnit(), cny.fractionsPerUnit()))
print('{0:<20}{1}'.format(usd.format(), cny.format()))

c1 = ql.Currency()
c2 = ql.EURCurrency()
c3 = ql.USDCurrency()
c4 = c2

e1 = c1.empty()
e2 = c2.empty()
e3 = c3.empty()
e4 = c4.empty()

print(e1, e2, e3, e4)
```

```
USDCurrency         CNYCurrency
U.S. dollar         Chinese yuan
USD                 CNY
840                 156
$                   Y
100                 100
%3% %1$.2f          %3% %1$.2f
True False False False
```
