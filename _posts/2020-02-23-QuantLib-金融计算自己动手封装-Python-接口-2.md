---
title: QuantLib 金融计算——自己动手封装 Python 接口（2）
date: 2020-02-23 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍如何为 QuantLib 封装 Python 接口。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——自己动手封装 Python 接口（2）

## 概述

对于一项简单功能，通常只需要包装少数几个类就可以，正如[《自己动手封装 Python 接口（1）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%B0%81%E8%A3%85-Python-%E6%8E%A5%E5%8F%A3-1/)演示的那样。

下面，将演示如何包装 QuantLib 中的复杂功能，最终实现**从固息债交易数据中估计期限结构模型的参数**。

## 如何封装一项复杂功能？

经过一翻摸索后发现，要封装一项复杂功能，首先要找到**最小功能集合**，即这项功能直接或间接涉及的类和函数有哪些。然后，找到最小功能集合后再对涉及到的类或函数分别编写接口文件。最后，按照常规流程生成包装好的 Python 接口。

对于简单功能来说最小功能集合可能就是一两个类或函数。而对于复杂功能来说，寻找最小功能集合是一个递归的过程（A 用到 B，B 用到 C，...），最终可能找到很多类或函数需要包装。

### 寻找最小功能集合的策略

寻找最小功能集合有一些经验性的方法，以“从固息债交易数据中估计期限结构模型的参数”这项功能为例：
1. 找到核心功能类，即 `FittedBondDiscountCurve`，最小功能集合要包含这个类、它的基类以及基类的基类，等等；
2. 找到构造 `FittedBondDiscountCurve` 对象时涉及到一系列的类，例如 `Calendar` 和 `FittingMethod` 等，这些类、它们的基类以及基类的基类也要包含在最小功能集合中；
3. 找到 `FittedBondDiscountCurve` 成员函数涉及到一系列的类，这些类、它们的基类以及基类的基类也要包含在最小功能集合中；
4. 把第 2 和第 3 步递归地进行下去，直到最小功能集合中的类和函数不再增加。

需要注意的是，到现在为止最小功能集合中出现的类有的可以发挥实际作用，例如 `Date`；而有的只是充当接口的基类，例如 `FittingMethod`，对于这些情况，要把它们能够发挥实际作用的派生类包含进最小功能集合。

## 实践

QuantLib-SWIG 从 1.16 开始修改了智能指针的包装方式，为了和最新版本保持一致，这里以 QuantLib 1.17 的 SWIG 接口文件为基础做适当修改，删去一些冗余代码，用以包装 QuantLib 1.15 的接口。

官方发布的接口文件中 `FittingMethod` 的构造函数不能接受 `OptimizationMethod` 对象，也不能进行 $L^2$ 正则化约束。在本次自定义的接口文件中扩展了构造函数的接口，克服上述局限。

接口文件请见 [QuantLibEx-SWIG](https://github.com/xuruilong100/QuantLibEx-SWIG)。

### 估计期限结构参数

把[《收益率曲线之构建曲线（5）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%94%B6%E7%9B%8A%E7%8E%87%E6%9B%B2%E7%BA%BF%E4%B9%8B%E6%9E%84%E5%BB%BA%E6%9B%B2%E7%BA%BF-5/)中的 C++ 代码翻译成 Python，验证封装后的接口是否可用。

```python
import QuantLibEx as qlx

print(qlx.__version__)

bondNum = 16

cleanPrice = [100.4941, 103.5572, 104.4135, 105.0056, 99.8335, 101.25, 102.3832, 97.0053,
              99.5164, 101.2435, 104.0539, 101.15, 96.1395, 91.1123, 122.0027, 92.4369]
priceHandle = [qlx.QuoteHandle(qlx.SimpleQuote(p)) for p in cleanPrice]
issueYear = [1999, 1999, 2001, 2002, 2003, 1999, 2004, 2005,
             2006, 2007, 2003, 2008, 2005, 2006, 1997, 2007]
issueMonth = [qlx.February, qlx.October, qlx.January, qlx.January, qlx.May, qlx.January, qlx.January, qlx.April,
              qlx.April, qlx.September, qlx.January, qlx.January, qlx.January, qlx.January, qlx.July, qlx.January]
issueDay = [22, 22, 4, 9, 20, 15, 15, 26, 21, 17, 15, 8, 14, 11, 10, 12]

maturityYear = [2009, 2010, 2011, 2012, 2013, 2014, 2014, 2015,
                2016, 2017, 2018, 2019, 2020, 2021, 2027, 2037]

maturityMonth = [qlx.July, qlx.January, qlx.January, qlx.July, qlx.October, qlx.January, qlx.July, qlx.July,
                 qlx.September, qlx.September, qlx.January, qlx.March, qlx.July, qlx.September, qlx.July, qlx.March]

maturityDay = [15, 15, 4, 15, 20, 15, 15, 15,
               15, 15, 15, 15, 15, 15, 15, 15]

issueDate = []
maturityDate = []
for i in range(bondNum):
    issueDate.append(
        qlx.Date(issueDay[i], issueMonth[i], issueYear[i]))
    maturityDate.append(
        qlx.Date(maturityDay[i], maturityMonth[i], maturityYear[i]))

couponRate = [
    0.04, 0.055, 0.0525, 0.05, 0.038, 0.04125, 0.043, 0.035,
    0.04, 0.043, 0.0465, 0.0435, 0.039, 0.035, 0.0625, 0.0415]

# 配置 helper

frequency = qlx.Annual
dayCounter = qlx.Actual365Fixed(qlx.Actual365Fixed.Standard)
paymentConv = qlx.Unadjusted
terminationDateConv = qlx.Unadjusted
convention = qlx.Unadjusted
redemption = 100.0
faceAmount = 100.0
calendar = qlx.Australia()

today = calendar.adjust(qlx.Date(30, qlx.January, 2008))
qlx.Settings.instance().evaluationDate = today

bondSettlementDays = 0
bondSettlementDate = calendar.advance(
    today,
    qlx.Period(bondSettlementDays, qlx.Days))

instruments = []
maturity = []

for i in range(bondNum):
    bondCoupon = [couponRate[i]]

    schedule = qlx.Schedule(
        issueDate[i],
        maturityDate[i],
        qlx.Period(frequency),
        calendar,
        convention,
        terminationDateConv,
        qlx.DateGeneration.Backward,
        False)

    helper = qlx.FixedRateBondHelper(
        priceHandle[i],
        bondSettlementDays,
        faceAmount,
        schedule,
        bondCoupon,
        dayCounter,
        paymentConv,
        redemption)

    maturity.append(dayCounter.yearFraction(
        bondSettlementDate, helper.maturityDate()))

    instruments.append(helper)

accuracy = 1.0e-6
maxEvaluations = 5000
weights = qlx.Array()

# 正则化条件

l2Ns = qlx.Array(4, 0.5)
guessNs = qlx.Array(4)
guessNs[0] = 4 / 100.0
guessNs[1] = 0.0
guessNs[2] = 0.0
guessNs[3] = 0.5

l2Sv = qlx.Array(6, 0.5)
guessSv = qlx.Array(6)
guessSv[0] = 4 / 100.0
guessSv[1] = 0.0
guessSv[2] = 0.0
guessSv[3] = 0.0
guessSv[4] = 0.2
guessSv[5] = 0.15

optMethod = qlx.LevenbergMarquardt()

# 拟合方法

nsf = qlx.NelsonSiegelFitting(
    weights, optMethod, l2Ns)
svf = qlx.SvenssonFitting(
    weights, optMethod, l2Sv)

tsNelsonSiegel = qlx.FittedBondDiscountCurve(
    bondSettlementDate,
    instruments,
    dayCounter,
    nsf,
    accuracy,
    maxEvaluations,
    guessNs,
    1.0)

tsSvensson = qlx.FittedBondDiscountCurve(
    bondSettlementDate,
    instruments,
    dayCounter,
    svf,
    accuracy,
    maxEvaluations,
    guessSv)

print("NelsonSiegel Results: \t", tsNelsonSiegel.fitResults().solution())
print("Svensson Results: \t\t", tsSvensson.fitResults().solution())
```

```
NelsonSiegel Results: 	[ 0.0500803; -0.0105414; -0.0303842; 0.456529 ]
Svensson Results: 		[ 0.0431095; -0.00716036; -0.0340932; 0.0391339; 0.228995; 0.117208 ]
```

所得结果和[《收益率曲线之构建曲线（5）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%94%B6%E7%9B%8A%E7%8E%87%E6%9B%B2%E7%BA%BF%E4%B9%8B%E6%9E%84%E5%BB%BA%E6%9B%B2%E7%BA%BF-5/#%E6%8B%9F%E5%90%88%E7%BB%93%E6%9E%9C)中的完全一致。

![](/img/meme/good.jpg)

### 修改官方接口文件

如果已经安装了 1.16 以后的 QuantLib，只要对官方接口文件稍加修改再重新包装 Python 接口，就可以扩展 `FittingMethod` 的构造函数，使其能接受 `OptimizationMethod` 对象，并能进行正则化。

以 `NelsonSiegelFitting` 为例，需要在 `fittedbondcurve.i` 文件中用

```c++
class NelsonSiegelFitting : public FittingMethod {
  public:
    NelsonSiegelFitting(
        const Array& weights = Array(),
        boost::shared_ptr< OptimizationMethod > optimizationMethod = boost::shared_ptr< OptimizationMethod >(),
        const Array &l2 = Array());
};
```

替换

```c++
class NelsonSiegelFitting : public FittingMethod {
  public:
    NelsonSiegelFitting(const Array& weights = Array());
};
```

## 下一步的计划

1. 包装 [QuantLibEx](https://github.com/xuruilong100/QuantLibEx) 中的几个期限结构模型；
2. scipy 的优化算法引擎要相较于 QuantLib 自身提供的要更丰富，尝试使 `FittingMethod` 能接受 scipy 的算法。
