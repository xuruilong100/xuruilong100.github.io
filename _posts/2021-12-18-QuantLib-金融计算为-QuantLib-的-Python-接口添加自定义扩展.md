---
title: QuantLib 金融计算——为 QuantLib 的 Python 接口添加自定义扩展
date: 2021-12-18 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 以 FR007 互换为例，演示如何为 QuantLib 的 Python 接口添加自定义扩展。
---

# 为 QuantLib 的 Python 接口添加自定义扩展——以 FR007 互换为例

现在将 [QuantLibEx](https://github.com/xuruilong100/QuantLibEx) 中的自定义扩展和 QuantLib 源代码合并封装成一个 Python 接口。

首先，要为自定义扩展添加 swig 接口文件，详见[这里](https://github.com/xuruilong100/QuantLibPythonExamples/tree/main/SWIGpy/ql/qlex)。

其次，要修改 `setup.py` 文件，让 python 知道需要编译那些 `.cpp` 文件，详见[这里](https://github.com/xuruilong100/QuantLibPythonExamples/blob/main/SWIGpy/setup.py)。

剩下的工作和往常一样：
1. 生成 `.cpp` 文件：`swig3.0 -c++ -python -outdir QuantLib -o QuantLib/ql_wrap.cpp quantlib.i`
2. 编译 .cpp 文件：`CC=clang CXX=clang++ python3 setup.py build`
3. 安装 Python 包装：`python3 setup.py install`

作为测试，将 [QuantLibEx](https://github.com/xuruilong100/QuantLibEx) 中的例子 `ChinaFixingRepoSwapCurve` 翻译成 python 代码。

> 扩展阅读：[《C++ 代码改写成 Python 程序的一些经验》](https://www.cnblogs.com/xuruilong100/p/13281006.html)

```python
import QuantLib as ql
import seaborn as sns

calendar = ql.China(ql.China.IB)
today = ql.Date(21, ql.January, 2020)
ql.Settings.instance().evaluationDate = today

delayDays = 1

settlementDate = calendar.advance(
    today, delayDays, ql.Days)
# must be a business day
settlementDate = calendar.adjust(settlementDate)

print("Today: ", today)
print("Settlement date: ", settlementDate)

termStrcDayCounter = ql.Actual365Fixed()

dy7 = ql.Period(7, ql.Days)
mn1 = ql.Period(1, ql.Months)
mn3 = ql.Period(3, ql.Months)
mn6 = ql.Period(6, ql.Months)
mn9 = ql.Period(9, ql.Months)
yr1 = ql.Period(1, ql.Years)
yr2 = ql.Period(2, ql.Years)
yr3 = ql.Period(3, ql.Years)
yr4 = ql.Period(4, ql.Years)
yr5 = ql.Period(5, ql.Years)
yr7 = ql.Period(7, ql.Years)
yr10 = ql.Period(10, ql.Years)

d7Rate = ql.SimpleQuote(2.5900 / 100.0)
s1mRate = ql.SimpleQuote(2.5848 / 100.0)
s3mRate = ql.SimpleQuote(2.5713 / 100.0)
s6mRate = ql.SimpleQuote(2.5788 / 100.0)
s9mRate = ql.SimpleQuote(2.5925 / 100.0)
s1yRate = ql.SimpleQuote(2.6033 / 100.0)
s2yRate = ql.SimpleQuote(2.6665 / 100.0)
s3yRate = ql.SimpleQuote(2.7415 / 100.0)
s4yRate = ql.SimpleQuote(2.8288 / 100.0)
s5yRate = ql.SimpleQuote(2.9130 / 100.0)
s7yRate = ql.SimpleQuote(3.0466 / 100.0)
s10yRate = ql.SimpleQuote(3.1763 / 100.0)

d7RateHandle = ql.QuoteHandle(d7Rate)
s1mRateHandle = ql.QuoteHandle(s1mRate)
s3mRateHandle = ql.QuoteHandle(s3mRate)
s6mRateHandle = ql.QuoteHandle(s6mRate)
s9mRateHandle = ql.QuoteHandle(s9mRate)
s1yRateHandle = ql.QuoteHandle(s1yRate)
s2yRateHandle = ql.QuoteHandle(s2yRate)
s3yRateHandle = ql.QuoteHandle(s3yRate)
s4yRateHandle = ql.QuoteHandle(s4yRate)
s5yRateHandle = ql.QuoteHandle(s5yRate)
s7yRateHandle = ql.QuoteHandle(s7yRate)
s10yRateHandle = ql.QuoteHandle(s10yRate)

depositDayCounter = ql.Actual365Fixed()

d7 = ql.DepositRateHelper(
    d7RateHandle, dy7, 0, calendar,
    ql.Unadjusted, False, depositDayCounter)

fixedLegFreq = ql.Quarterly
fixedLegConv = ql.ModifiedFollowing
fixedLegDayCounter = ql.Actual365Fixed()

chinaFixingRepo = ql.ChinaFixingRepo(dy7, delayDays)

s1m = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, mn1,
    s1mRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s3m = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, mn3,
    s3mRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s6m = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, mn6,
    s6mRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s9m = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, mn9,
    s9mRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s1y = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, yr1,
    s1yRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s2y = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, yr2,
    s2yRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s3y = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, yr3,
    s3yRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s4y = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, yr4,
    s4yRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s5y = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, yr5,
    s5yRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s7y = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, yr7,
    s7yRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)
s10y = ql.ChinaFixingRepoSwapRateHelper(
    delayDays, yr10,
    s10yRateHandle, chinaFixingRepo, ql.YieldTermStructureHandle(), 0,
    fixedLegConv, fixedLegFreq, calendar)

instruments = ql.RateHelperVector()

instruments.push_back(d7)
instruments.push_back(s1m)
instruments.push_back(s3m)
instruments.push_back(s6m)
instruments.push_back(s9m)
instruments.push_back(s1y)
instruments.push_back(s2y)
instruments.push_back(s3y)
instruments.push_back(s4y)
instruments.push_back(s5y)
instruments.push_back(s7y)
instruments.push_back(s10y)

termStrc = ql.PiecewiseBackwardFlatForward(
    today,
    instruments,
    termStrcDayCounter)

curveNodeDate = calendar.adjust(settlementDate + dy7)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + mn1)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + mn3)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + mn6)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + mn9)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + yr1)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + yr2)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + yr3)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + yr4)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + yr5)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + yr7)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

curveNodeDate = calendar.adjust(settlementDate + yr10)
print(curveNodeDate - today, '{0:.8f}'.format(termStrc.discount(curveNodeDate)),
      '{0:.7f}'.format(termStrc.zeroRate(curveNodeDate, termStrcDayCounter, ql.Continuous).rate() * 100.0))

sns.lineplot(
    x=[13, 34, 92, 183, 275, 367, 734, 1098, 1462, 1828, 2558, 3654],
    y=[2.5884391, 2.5819802, 2.5633687, 2.570667, 2.5842461, 2.5950071,
       2.6584605, 2.7347369, 2.8246057, 2.9122383, 3.0531303, 3.1927605])
```

结果和之前的完全一样。

![](/img/meme/ok.gif)

```
Today:  January 21st, 2020
Settlement date:  January 22nd, 2020
13 0.99907852 2.5884391
34 0.99759776 2.5819802
92 0.99355973 2.5633687
183 0.98719415 2.5706670
275 0.98071798 2.5842461
367 0.97424520 2.5950071
734 0.94794334 2.6584605
1098 0.92102612 2.7347369
1462 0.89302652 2.8246057
1828 0.86428623 2.9122383
2558 0.80737256 3.0531303
3654 0.72642071 3.1927605
```

![](/img/QuantLib/fr007-curve.png)
