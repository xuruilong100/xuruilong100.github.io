---
title: QuantLib 金融计算——基本组件之 Money 类
date: 2019-05-28 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 QuantLib 基本组件 Money 类的使用。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——基本组件之 `Money` 类

载入 QuantLib：

```python
import QuantLib as ql

print(ql.__version__)
```

```
1.15
```

## 概述

若要在 QuantLib 中对货币进行代数计算，就要将 `Currency` 对象转变成为一个 `Money` 对象。

## 构造函数

`Money` 的构造函数有两种，都接受两个参数：

```python
Money(currency, value)
Money(value, currency)
```

* `currency`：一个 `Currency` 对象；
* `value`：一个浮点数，表示货币的数量。

`Money` 通常不显式构造，而是用通过 `Currency` 对象乘一个浮点数产生：

```python
cny = ql.CNYCurrency()

m = 123.4567 * cny
```

## 成员函数

常用成员函数如下：

* `currency()`：返回 `Currency` 对象，即货币；
* `value()`：返回浮点数，即货币量；
* `rounded()`：返回四舍五入后的 `Money` 对象。

此外 `Money` 类重载了运算符，以实现基本的代数计算。

示例，

```python
cny = ql.CNYCurrency()

m = 123.4567 * cny

print(m.value())
print(m.currency())
print(m.rounded())
print(m * 10)
print(m + m)
```

```
123.4567
Chinese yuan
Y 123.46
Y 1234.57
Y 246.91
```

结果会根据货币的类型自动四舍五入。
