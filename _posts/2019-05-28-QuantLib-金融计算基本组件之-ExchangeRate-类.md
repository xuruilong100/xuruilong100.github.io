---
title: QuantLib 金融计算——基本组件之 ExchangeRate 类
date: 2019-05-28 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 ExchangeRate 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `ExchangeRate` 类

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.15
```

## 概述

QuantLib 中描述货币之间汇率信息的类是 `ExchangeRate`，`Currency` 体系内的每两种货币都可以生成出一个 `ExchangeRate` 对象。

## 构造函数

`ExchangeRate` 的构造函数非常固定，接受三个参数：

```python
ExchangeRate(source,
             target,
             rate)
```

* `source`：一个 `Currency` 对象，表示源货币；
* `target`：一个 `Currency` 对象，表示目标货币；
* `rate`：一个浮点数，表示“`source` 对 `target`”的汇率。

## 成员函数

常用成员函数如下：

* `source()`：返回 `Currency` 对象，即源货币；
* `target()`：返回 `Currency` 对象，即目标货币；
* `rate()`：返回浮点数，即汇率；
* `type()`：返回内置的整数常量，
  * `ExchangeRate.Direct`：等于 0，表示该汇率是通过构造函数直接构造的；
  * `ExchangeRate.Derived`：等于 1，表示该汇率是通过其他汇率对象简间接构造的；
* `exchange(amount)`：`amount` 是一个 `Money` 对象，该函数将 `amount` 转换成等价值的其他货币；
* `chain(r1, r2)`：`r1` 和 `r2` 是 `ExchangeRate` 对象，所涉及的货币必须构成一个三角关系，该函数将返回一个 `ExchangeRate` 对象，补全三角关系中缺失的一边。

示例，

```python
import QuantLib as ql
usd = ql.USDCurrency()
cny = ql.CNYCurrency()

usdTocny = ql.ExchangeRate(usd, cny, 6.85)

m_usd = 1.32 * usd
m_cny = 5.32 * cny

print(
    'Converting from USD: ', m_usd, ' = ',
    usdTocny.exchange(m_usd))
print(
    'Converting from CNY: ', m_cny, ' = ',
    usdTocny.exchange(m_cny))

print(usdTocny.source())
print(usdTocny.target())
print(usdTocny.rate())

eur = ql.EURCurrency()

cnyToeur = ql.ExchangeRate(eur, cny, 7.73)

usdToeur = ql.ExchangeRate.chain(usdTocny, cnyToeur)

m_eur = 1000.0 * eur

print(
    'Converting from EUR: ', m_eur, ' = ',
    usdToeur.exchange(m_eur))

print(usdTocny.type() == ql.ExchangeRate.Direct)
print(usdToeur.type() == ql.ExchangeRate.Derived)
```

```
Converting from USD:  $ 1.32  =  Y 9.04
Converting from CNY:  Y 5.32  =  $ 0.78
U.S. dollar
Chinese yuan
6.85
Converting from EUR:  EUR 1000.00  =  $ 1128.47
True
True
```

结果会根据货币的类型自动四舍五入。
