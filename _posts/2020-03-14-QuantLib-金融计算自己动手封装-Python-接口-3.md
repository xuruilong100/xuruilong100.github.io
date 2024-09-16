---
title: QuantLib 金融计算——自己动手封装 Python 接口（3）
date: 2020-03-14 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍如何为 QuantLib 封装 Python 接口。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——自己动手封装 Python 接口（3）

## 概述

承接[《自己动手封装 Python 接口（2）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%B0%81%E8%A3%85-Python-%E6%8E%A5%E5%8F%A3-2/)中留下的问题，即封装 [QuantLibEx](https://github.com/xuruilong100/QuantLibEx) 中的几个期限结构模型。

## 如何封装源代码？

与前一篇文章中的情况不同，要封装的程序不是已经编译好的库文件，而是 C++ 源代码。

SWIG 可以从源代码的层面封装 C++ 接口，一方面要提供头文件，告知 SWIG 类、函数等的声明；另一方面要提供源文件，让 SWIG 知道方法的实现，SWIG 会自动对源文件进行编译，并最终链接到生成的 Python 接口中。

## 实践

幸运的是 QuantLibEx 中几个 NS 型期限结构模型的构造函数没有引入新的类型，所以“最小功能集合”没有变。

要封装这几个模型，只需对 [`fittedbondcurve.i`](https://github.com/xuruilong100/QuantLibEx-SWIG/blob/master/fittedbondcurve.i) 和 [`setup.py`](https://github.com/xuruilong100/QuantLibEx-SWIG/blob/master/setup.py) 稍加修改。在 `fittedbondcurve.i` 中编写接口代码，在 `setup.py` 添加头文件路径和几个源文件就可以了。

### 六个 NS 型期限结构模型的参数估计

把[《利率曲线之构建曲线（5）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%94%B6%E7%9B%8A%E7%8E%87%E6%9B%B2%E7%BA%BF%E4%B9%8B%E6%9E%84%E5%BB%BA%E6%9B%B2%E7%BA%BF-5/)中的 C++ 代码翻译成 Python，验证封装后的接口是否可用。

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

l2Asv = qlx.Array(6, 0.5)
guessAsv = qlx.Array(6)
guessAsv[0] = 4 / 100.0
guessAsv[1] = 0.0
guessAsv[2] = 0.0
guessAsv[3] = 0.0
guessAsv[4] = 0.2
guessAsv[5] = 0.3

l2Bc = qlx.Array(5, 0.5)
guessBc = qlx.Array(5)
guessBc[0] = 4 / 100.0
guessBc[1] = 0.0
guessBc[2] = 0.0
guessBc[3] = 0.0
guessBc[4] = 0.2

l2Bl = qlx.Array(5, 0.5)
guessBl = qlx.Array(5)
guessBl[0] = 4 / 100.0
guessBl[1] = 0.0
guessBl[2] = 0.0
guessBl[3] = 0.5
guessBl[4] = 0.5

optMethod = qlx.LevenbergMarquardt()

# 拟合方法

nsf = qlx.NelsonSiegelFitting(
    weights, optMethod, l2Ns)
svf = qlx.SvenssonFitting(
    weights, optMethod, l2Sv)
asvf = qlx.AdjustedSvenssonFitting(
    weights, optMethod, l2Asv)
dlf = qlx.DieboldLiFitting(
    0.5, weights, optMethod)
bcf = qlx.BjorkChristensenFitting(
    weights, optMethod, l2Bc)
blf = qlx.BlissFitting(
    weights, optMethod, l2Bl)

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

tsAdjustedSvensson = qlx.FittedBondDiscountCurve(
    bondSettlementDate,
    instruments,
    dayCounter,
    asvf,
    accuracy,
    maxEvaluations,
    guessAsv)

tsDieboldLi = qlx.FittedBondDiscountCurve(
    bondSettlementDate,
    instruments,
    dayCounter,
    dlf,
    accuracy,
    maxEvaluations)

tsBjorkChristensen = qlx.FittedBondDiscountCurve(
    bondSettlementDate,
    instruments,
    dayCounter,
    bcf,
    accuracy,
    maxEvaluations,
    guessBc)

tsBliss = qlx.FittedBondDiscountCurve(
    bondSettlementDate,
    instruments,
    dayCounter,
    blf,
    accuracy,
    maxEvaluations,
    guessBl)

print("NelsonSiegel Results: \t\t", tsNelsonSiegel.fitResults().solution())
print("Svensson Results: \t\t\t", tsSvensson.fitResults().solution())
print("AdjustedSvensson Results: \t", tsAdjustedSvensson.fitResults().solution())
print("DieboldLi Results: \t\t\t", tsDieboldLi.fitResults().solution())
print("BjorkChristensen Results: \t", tsBjorkChristensen.fitResults().solution())
print("Bliss Results: \t\t\t\t", tsBliss.fitResults().solution())
```

```
NelsonSiegel Results:       [ 0.0500803; -0.0105414; -0.0303842; 0.456529 ]
Svensson Results:           [ 0.0431095; -0.00716036; -0.0340932; 0.0391339; 0.228995; 0.117208 ]
AdjustedSvensson Results:   [ 0.0506269; -0.0116339; 0.0029305; -0.0135686; 0.179066; 0.267767 ]
DieboldLi Results:          [ 0.0496643; -0.00879931; -0.0329267 ]
BjorkChristensen Results:   [ 0.0508039; -0.0555185; 0.0115282; 0.0415581; 0.227838 ]
Bliss Results:              [ 0.0500892; -0.0106013; -0.0315605; 0.513831; 0.456329 ]
```

所得结果和[《利率曲线之构建曲线（5）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%94%B6%E7%9B%8A%E7%8E%87%E6%9B%B2%E7%BA%BF%E4%B9%8B%E6%9E%84%E5%BB%BA%E6%9B%B2%E7%BA%BF-5/)中的完全一致。

![](/img/meme/good.jpg)
