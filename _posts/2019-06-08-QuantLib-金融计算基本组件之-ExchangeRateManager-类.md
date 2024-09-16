---
title: QuantLib 金融计算——基本组件之 ExchangeRateManager 类
date: 2019-06-08 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 ExchangeRateManager 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `ExchangeRateManager` 类

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.15
```

## 概述

QuantLib 中管理货币之间汇率信息的类是 `ExchangeRateManager`，配合 `Money` 类中的相应配置可以实现在货币的代数计算中自动转换汇率。

## `Money` 类中的汇率转换配置

`Money` 类中的汇率转换配置通过静态函数 `setConversionType` 实现，可用的配置由三个内置整数表示，分别为：

| 配置类型                 | 含义                                    |
| ------------------------ | --------------------------------------- |
| `NoConversion`           | 不进行转换                              |
| `BaseCurrencyConversion` | 统一转换成一种基准货币（base currency） |
| `AutomatedConversion`    | 转换成计算表达式中出现的第一种货币      |

如果使用 `BaseCurrencyConversion`，还需要在 `Money` 类调用静态函数 `setBaseCurrency` 配置基准货币。

## `ExchangeRateManager`

`ExchangeRateManager` 是一个单体（Singleton），一般不直接显式实例化，需要通过调用静态函数 `instance` 获得唯一的一个 `ExchangeRateManager` 实例。

```python
ExchangeRateManager.instance()
```

## 函数

`ExchangeRateManager` 成员函数有三个。

* `add(ex, start_date, end_date)`：向 `ExchangeRateManager` 实例添加一个 `ExchangeRate` 对象 `ex`，该汇率的有效期起始时间是 `start_date`（默认值是 `Date.minDate()`），该汇率的有效期结束时间是 `end_date`（默认值是 `Date.maxDate()`）。
* `lookup(source, target, date, type)`：返回一个汇率对象，源货币为 `source`，目标货币是 `target`，日期在 `date`（默认为当前日期），类型为 `type`（默认值是 `ExchangeRate.Derived`）。`ExchangeRateManager` 实例将首先在所有记录的汇率中寻找想要的汇率；如果找不到，则试图根据汇率串联起来的路径返回最短路径上推算出的汇率（这种情况下 `type` 必须是 `ExchangeRate.Derived`）。
* `clear()`：清空记录的汇率。

示例，

```python
ql.Money.setConversionType(ql.Money.AutomatedConversion)

usd = ql.USDCurrency()
cny = ql.CNYCurrency()
eur = ql.EURCurrency()

usdXcny = ql.ExchangeRate(usd, cny, 6.912)
usdXeur = ql.ExchangeRate(usd, eur, 0.834)

ql.ExchangeRateManager.instance().add(usdXcny)
ql.ExchangeRateManager.instance().add(
    usdXeur,
    ql.Date(1, ql.May, 2019),
    ql.Date(3, ql.May, 2019))

m_eur = 100 * eur
m_cny = 150 * cny

ql.Settings.instance().evaluationDate = ql.Date(2, ql.May, 2019)

print(m_eur, " + ", m_cny, " = ", m_eur + m_cny)

ql.Settings.instance().evaluationDate = ql.Date(4, ql.May, 2019)

print(m_eur, " + ", m_cny, " = ", m_eur + m_cny)
```

```
EUR 100.00  +  Y 150.00  =  EUR 118.10
RuntimeError: no conversion available from CNY to EUR for May 4th, 2019
```

结果会根据货币的类型自动四舍五入。注意：欧元与人民币之间的汇率通过美元间接获得，当估值日期（`evaluationDate`）那天没有可用汇率时，系统会报错。
