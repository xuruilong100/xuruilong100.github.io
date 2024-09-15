---
title: QuantLib 金融计算——案例之浮息债（挂钩 LPR）的价格、久期和凸性
date: 2020-09-21 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍用 QuantLib 分析浮息债（挂钩 LPR）的价格、久期和凸性。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——案例之浮息债（挂钩 LPR）的价格、久期和凸性

## 概述

作为利率风险系列的第三篇，本文将依据中债登公布的估值公式，介绍挂钩 LPR 的浮息债的价格、久期和凸性的计算方法，并依托 QuantLib 和 Python 展示相关的编程案例。

## 中债登的估值公式

**贷款市场报价利率**（Loan Prime Rate，简称 LPR），是由具有代表性的报价行，根据本行对最优质客户的贷款利率，以公开市场操作利率（主要指中期借贷便利利率）加点形成的方式报价，由中国人民银行授权全国银行间同业拆借中心计算并公布的基础性的贷款参考利率，各金融机构应主要参考 LPR 进行贷款定价。

目前，LPR 包括 1 年期和 5 年期以上两个品种。每月 20 日（遇节假日顺延）上午 9 时 30 分由人民银行授权全国银行间同业拆借中心发布。

最近一年来央行一直在大力推行 LPR，不但推出了挂钩 LPR 的浮息债，而且推出了 LPR 的利率互换和利率期权（未来将专门论述）。

对于挂钩 LPR 的浮息债，中债登使用如下估值公式：

$$
PV = \left(
\frac{(r+s)/f}{(1+(R+y)/f)^t} +
\sum_{i=1}^n \frac{(R+s)/f}{(1+(R+y)/f)^{t+i}} +
\frac{1}{(1+(R+y)/f)^{t+n}}
\right) \times 100
$$

其中：
* $PV$：债券全价
* $r$：当期债券基础利率
* $R$：估值日基础利率
* $s$：债券招标利差
* $f$：每年付息次数
* $y$：点差利率
* $R + y$：到期利率（YTM）
* $n$：剩余完整付息周期个数
* $t$：距离下一付息日的天数占当前付息周期长度的比例

这与国外教科书中的估值公式有很大不同，国外教科书中的公式通常要利用当前的期限结构推算远期利率，进而得到预期的未来现金流（浮动票息），再对现金流贴现。中债登的公式可以看做是使用了“水平”的期限结构，如果浮息债挂钩 Shibor3M 或 FR007，也许可以照搬教科书，因为这两种利率有对应的 IRS 在交易，且流动性较好，理论上可以推算出 Shibor 和 FR 的期限结构（或远期利率）。

## 浮息债的久期和凸性

依据中债登的估值公式，浮息债的价格受到两个可变参数的影响，分别是 $R$ 和 $y$。因此，浮息债分别就 $R$ 和 $y$ 衍生出两套久期和凸性。

### 利差久期和利差凸性

浮息债价格关于点差利率（$y$）的一阶敏感性叫做“利差久期”，二阶敏感性叫做“利差凸性”。由于 $y$ 只出现在贴现因子中，**浮息债的利差久期（凸性）和普通固息债的久期（凸性）别无二致**。

* 利差久期：

$$
D_y = -\frac{\mathrm{d} PV}{\mathrm{d}y} \frac{1}{PV}
$$

$$
D_y = \frac{1}{PV}\frac{1}{f}\frac{1}{1+(R+y)/f}
\left(
\frac{t(r+s)/f}{(1+(R+y)/f)^t} +
\sum_{i=1}^n \frac{(t+i)(R+s)/f}{(1+(R+y)/f)^{t+i}} +
\frac{t+n}{(1+(R+y)/f)^{t+n}}
\right) \times 100 \tag{1}
$$

* 利差凸性：

$$
C_y = \frac{\mathrm{d}^2 PV}{\mathrm{d}y^2} \frac{1}{PV}
$$

$$
C_y = \frac{1}{PV}\frac{1}{f^2}\frac{1}{(1+(R+y)/f)^2}
\left(
\frac{t(t+1)(r+s)/f}{(1+(R+y)/f)^t} +
\sum_{i=1}^n \frac{(t+i)(t+i+1)(R+s)/f}{(1+(R+y)/f)^{t+i}} +
\frac{(t+n)(t+n+1)}{(1+(R+y)/f)^{t+n}}
\right) \times 100 \tag{2}
$$

### 利率久期和利率凸性

浮息债价格关于估值日基础利率（$R$）的一阶敏感性叫做“利率久期”，二阶敏感性叫做“利率凸性”。由于 $R$ 同时出现在贴现因子和现金流中，$R$ 变化的影响会被抵消掉一部分。因此，**浮息债的利率久期（凸性）较利差久期（凸性）通常要小很多**。

对浮息债而言，利率的市场风险主要体现在点差利率 $y$ 的变化上。（这一点和信用利差非常相似）

* 利率久期：

$$
D_{R} = -\frac{\mathrm{d} PV}{\mathrm{d}R} \frac{1}{PV}
$$

$$
\begin{aligned}
D_{R} =&\frac{1}{PV}\frac{1}{f}\frac{1}{1+(R+y)/f}
\left(
\frac{t(r+s)/f}{(1+(R+y)/f)^t} +
\sum_{i=1}^n \frac{(t+i)(R+s)/f}{(1+(R+y)/f)^{t+i}} +
\frac{t+n}{(1+(R+y)/f)^{t+n}}
\right) \times 100\\
&- \frac{1}{PV}\frac{1}{f}
\left(
\sum_{i=1}^n \frac{1}{(1+(R+y)/f)^{t+i}}
\right)\times 100\\
=&D_y - \frac{1}{PV}\frac{1}{f}
\left(
\sum_{i=1}^n \frac{1}{(1+(R+y)/f)^{t+i}}
\right)\times 100
\end{aligned} \tag{3}
$$

记

$$
\Sigma = \frac{1}{f}
\left(
\sum_{i=1}^n \frac{1}{(1+(R+y)/f)^{t+i}}
\right)\times 100
$$

利率久期等于利差久期减去 $\Sigma / PV$，可以推测利率久期要比利差久期小很多。

* 利率凸性

$$
C_{R} = \frac{\mathrm{d}^2 PV}{\mathrm{d}R^2} \frac{1}{PV}
$$

根据（3）可以知道

$$
\frac{\mathrm{d} PV}{\mathrm{d}R} = \frac{\mathrm{d} PV}{\mathrm{d}y} + \Sigma
$$

因此

$$
\frac{\mathrm{d}^2 PV}{\mathrm{d}R^2} = \frac{\mathrm{d}^2 PV}{\mathrm{d}y\mathrm{d}R} + \frac{\mathrm{d}\Sigma}{\mathrm{d}R}\\
\frac{\mathrm{d}^2 PV}{\mathrm{d}R\mathrm{d}y} = \frac{\mathrm{d}^2 PV}{\mathrm{d}y^2} + \frac{\mathrm{d}\Sigma}{\mathrm{d}y}
$$

进而

$$
\frac{\mathrm{d}^2 PV}{\mathrm{d}R^2} =  \frac{\mathrm{d}^2 PV}{\mathrm{d}y^2} + \frac{\mathrm{d}\Sigma}{\mathrm{d}y}  + \frac{\mathrm{d}\Sigma}{\mathrm{d}R}
$$

而对于 $\Sigma$ 来说，

$$
\frac{\mathrm{d}\Sigma}{\mathrm{d}y}  = \frac{\mathrm{d}\Sigma}{\mathrm{d}R}
$$

所以

$$
\frac{\mathrm{d}^2 PV}{\mathrm{d}R^2} =  \frac{\mathrm{d}^2 PV}{\mathrm{d}y^2} + 2\frac{\mathrm{d}\Sigma}{\mathrm{d}y}
$$

最终

$$
C_{R} = C_y - 2\frac{1}{PV}\frac{1}{f^2}
\left(
\sum_{i=1}^n \frac{t+i}{(1+(R+y)/f)^{t+i+1}}
\right)\times 100
$$

利率凸性等于利差凸性加 $2\frac{1}{PV}\frac{\mathrm{d}\Sigma}{\mathrm{d}y}$，可以推测利率凸性要比利差凸性小很多。

## 计算案例

下面将以 200218 为例，计算 2020-09-15 这一天的价格、久期和凸性。

### 价格与现金流

首先从[中国货币网](https://www.chinamoney.com.cn/chinese/)查询债券的基本信息，用以配置 `FloatingRateBond` 对象。

- 债券起息日：2020-06-09
- 到期兑付日：2025-06-09
- 债券期限：5 年
- 面值(元)：100.00
- 计息基准：A/A
- 息票类型：附息式浮动利率
- 付息频率：季
- 票面利率（%）：3.1（当前水平）
- 基准利率（%）：3.85（当前水平）
- 基准利差（%）：-0.75
- 基准利率名：LPR1Y
- 利率杠杆：1
- 提前确定利率的天数：1（没有查到该项目，不过此项不影响估值计算）
- 结算方式：T+0（与中债估值的规则保持一致）


```python
import QuantLib as ql
import prettytable as pt
from datetime import date

today = ql.Date(15, ql.September, 2020)
ql.Settings.instance().evaluationDate = today
evalueDate = ql.Settings.instance().evaluationDate

settlementDays = 0
faceAmount = 100.0

effectiveDate = ql.Date(9, ql.June, 2020)
terminationDate = ql.Date(9, ql.June, 2025)
tenor = ql.Period(ql.Quarterly)
calendar = ql.China(ql.China.IB)
convention = ql.Unadjusted
terminationDateConvention = convention
rule = ql.DateGeneration.Backward
endOfMonth = False

schedule = ql.Schedule(
    effectiveDate,
    terminationDate,
    tenor,
    calendar,
    convention,
    terminationDateConvention,
    rule,
    endOfMonth)

nextLpr = 3.85 / 100.0
nextLprQuote = ql.SimpleQuote(nextLpr)
nextLprHandle = ql.QuoteHandle(nextLprQuote)
fixedLpr = 3.85 / 100.0
```

需要注意的是，日历采用中国的银行间市场，遇到假期不调整。

目前，QuantLib 中没有挂钩 LPR 的浮息债的直接实现，但是鉴于中债登估值的公式比较简单，可以用 QuantLib 中的一些组件“模拟”出中债登的估值方法。
* 首先，要把 LPR1Y “想象”成一种类似 Shibor3M 的短期利率。此时的 $r$ 就是最新的 LPR1Y 利率；
* 浮动票息由一个水平的期限结构推算出来，对应利率是 $R$，也就是到期利率和点差利率的差（实际上就等于最新的 LPR1Y 利率）；
* 贴现因子也由一个水平的期限结构推算出来，对应利率是 $R+y$，也就是到期利率。

```python
compounding = ql.Compounded
frequency = ql.Quarterly
accrualDayCounter = ql.ActualActual(ql.ActualActual.Bond, schedule)
cfDayCounter = ql.ActualActual(ql.ActualActual.Bond)
paymentConvention = ql.Unadjusted
fixingDays = 1
gearings = ql.DoubleVector(1, 1.0)
benchmarkSpread = ql.DoubleVector(1, -0.75 / 100.0)

cfLprTermStructure = ql.YieldTermStructureHandle(
    ql.FlatForward(
        settlementDays,
        calendar,
        nextLprHandle,
        cfDayCounter,
        compounding,
        frequency))

lprTermStructure = ql.YieldTermStructureHandle(
    ql.FlatForward(
        settlementDays,
        calendar,
        nextLprHandle,
        accrualDayCounter,
        compounding,
        frequency))

lpr3m = ql.IborIndex(
    'LPR1Y',
    ql.Period(3, ql.Months),
    settlementDays,
    ql.CNYCurrency(),
    calendar,
    convention,
    endOfMonth,
    cfDayCounter,
    cfLprTermStructure)

lpr3m.addFixing(ql.Date(8, ql.June, 2020), fixedLpr)
lpr3m.addFixing(ql.Date(8, ql.September, 2020), fixedLpr)

bond = ql.FloatingRateBond(
    settlementDays,
    faceAmount,
    schedule,
    lpr3m,
    accrualDayCounter,
    convention,
    fixingDays,
    gearings,
    benchmarkSpread)

bondYield = 3.7179 / 100.0
basisSpread = bondYield - nextLpr
basisSpreadQuote = ql.SimpleQuote(basisSpread)
basisSpreadHandle = ql.QuoteHandle(basisSpreadQuote)

zeroSpreadedTermStructure = ql.YieldTermStructureHandle(
    ql.ZeroSpreadedTermStructure(
        lprTermStructure,
        basisSpreadHandle,
        compounding,
        frequency,
        accrualDayCounter))

engine = ql.DiscountingBondEngine(zeroSpreadedTermStructure)
bond.setPricingEngine(engine)
```

有三点注意事项：
* 推算票息和贴现因子的期限结构使用了各自的 day counter，原因出在 `IborIndex` 上，它和前面的 `Schedule` 在有关时间的计算上可能产生不一致（不算严重的 bug，算是个 flaw），具体的原因请阅读以下两个链接的内容（[链接 1](https://github.com/lballabio/QuantLib-SWIG/issues/305)、[链接 2](https://quant.stackexchange.com/questions/12707/pricing-a-fixedratebond-in-quantlib-yield-vs-termstructure)）
* 由于是对存续债券估值，需要为期限结构添加“历史浮动利率”——历史上 fixing date 上的 LPR1Y 数据。尽管只有最近一次 fixing 的 LPR1Y 利率会参与估值，但用户还是要添加更早期 fixing date 的利率，否则会报错，幸运的是更早期的历史利率不参与估值，可以随便用个数来填充。（[《案例之普通利率互换分析（1）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%A1%88%E4%BE%8B%E4%B9%8B%E6%99%AE%E9%80%9A%E5%88%A9%E7%8E%87%E4%BA%92%E6%8D%A2%E5%88%86%E6%9E%90-1/)也出现了这个情况，可以作为参考阅读）
* 计算贴现因子用到了 `ZeroSpreadedTermStructure`，这里的利差就是点差利率 $y$。

打印出债券的现金流。

```python
cfTab = pt.PrettyTable(['Date', 'Amount'])

for c in bond.cashflows():
    dt = date(c.date().year(), c.date().month(), c.date().dayOfMonth())
    cfTab.add_row([dt, c.amount()])

cfTab.float_format = '.4'

print(cfTab)

'''
+------------+----------+
|    Date    |  Amount  |
+------------+----------+
| 2020-09-09 |  0.7750  |
| 2020-12-09 |  0.7750  |
| 2021-03-09 |  0.7750  |
| 2021-06-09 |  0.7750  |
| 2021-09-09 |  0.7750  |
| 2021-12-09 |  0.7750  |
| 2022-03-09 |  0.7750  |
| 2022-06-09 |  0.7750  |
| 2022-09-09 |  0.7750  |
| 2022-12-09 |  0.7750  |
| 2023-03-09 |  0.7750  |
| 2023-06-09 |  0.7750  |
| 2023-09-09 |  0.7750  |
| 2023-12-09 |  0.7750  |
| 2024-03-09 |  0.7750  |
| 2024-06-09 |  0.7750  |
| 2024-09-09 |  0.7750  |
| 2024-12-09 |  0.7750  |
| 2025-03-09 |  0.7750  |
| 2025-06-09 |  0.7750  |
| 2025-06-09 | 100.0000 |
+------------+----------+
'''
```

如果 day counter 使用不当，现金流可能与 0.7750 存在肉眼难以发觉的细微差距（但对估值依然可以产生可观的影响），特别是 2023-09-09 这一天，示例详见[链接 1](https://github.com/lballabio/QuantLib-SWIG/issues/305)。

测试一下有关价格的计算。

```python
cleanPrice = bond.cleanPrice()
dirtyPrice = bond.dirtyPrice()
accruedAmount = bond.accruedAmount()

tab = pt.PrettyTable(['item', 'value'])
tab.add_row(['clean price', cleanPrice])
tab.add_row(['dirty price', dirtyPrice])
tab.add_row(['accrued amount', accruedAmount])

tab.float_format = '.4'

print(tab)

'''
+----------------+---------+
|      item      |  value  |
+----------------+---------+
|  clean price   | 97.3292 |
|  dirty price   | 97.3803 |
| accrued amount |  0.0511 |
+----------------+---------+
'''
```

### 久期与凸性

之前的公式推导显示，浮息债的利差久期（凸性）就是通常意义上的久期（凸性），而利率久期（凸性）则可以在利差久期（凸性）的基础上添加一个附加项得到。因此，可以针对 LPR 浮息债创建一个 `BondFunctions` 的派生类，把利率久期（凸性）的计算放到派生类里面，同时还可以复用 `BondFunctions` 的函数。

```python
class LprBondFunctions(ql.BondFunctions):
    def __init__(self):
        ql.BondFunctions.__init__(self)

    @staticmethod
    def yieldDuration(bond: ql.FloatingRateBond,
                      bondYield: float,
                      dayCounter: ql.DayCounter,
                      compounding,
                      frequency):
        evalueDate = ql.Settings.instance().evaluationDate
        notOccurred = [
            cf for cf in bond.cashflows() if cf.date() > evalueDate]
        dur = ql.BondFunctions.duration(
            bond,
            bondYield,
            dayCounter,
            compounding,
            frequency,
            ql.Duration.Modified)
        p = bond.dirtyPrice()
        y = ql.InterestRate(
            bondYield,
            dayCounter,
            compounding,
            frequency)
        f = y.frequency()
        sigma = 0.0

        # 如果 len(notOccurred) <= 2，这意味着
        # 当前处于最后一个付息周期
        if len(notOccurred) > 2:
            # 跳过第一个和最后一个日期，因为在最后一个日期，
            # 本金与票息是两个独立的现金流
            for i in range(1, len(notOccurred) - 1):
                df = y.discountFactor(
                    evalueDate,
                    notOccurred[i].date())
                sigma += df

        dur -= sigma / p / f * 100.0

        return dur

    @staticmethod
    def yieldConvexity(bond: ql.FloatingRateBond,
                       bondYield: float,
                       dayCounter: ql.DayCounter,
                       compounding,
                       frequency):
        evalueDate = ql.Settings.instance().evaluationDate
        notOccurred = [
            cf for cf in bond.cashflows() if cf.date() > evalueDate]
        conv = ql.BondFunctions.convexity(
            bond,
            bondYield,
            dayCounter,
            compounding,
            frequency)
        p = bond.dirtyPrice()
        y = ql.InterestRate(
            bondYield,
            dayCounter,
            compounding,
            frequency)
        f = y.frequency()
        dSigma = 0.0

        # 如果 len(notOccurred) <= 2，这意味着
        # 当前处于最后一个付息周期
        if len(notOccurred) > 2:
            # 跳过第一个和最后一个日期，因为在最后一个日期，
            # 本金与票息是两个独立的现金流
            for i in range(1, len(notOccurred) - 1):
                t = f * dayCounter.yearFraction(
                    evalueDate,
                    notOccurred[i].date())
                df = y.discountFactor(
                    evalueDate,
                    notOccurred[i].date())
                dSigma += t * df

        dSigma /= 1 + bondYield / f
        conv -= 2.0 * dSigma / p / f ** 2 * 100.0

        return conv
```

用解析公式和数值法分别计算久期和凸性，相互验证。这里依然用到了 `Quote` 类的奇妙特性。

```Python
compTab = pt.PrettyTable()
compTab.add_column(
    '项目',
    ['利差久期', '利差凸性', '利率久期', '利率凸性'])

spreadDuration = ql.BondFunctions.duration(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency,
    ql.Duration.Modified)

spreadConvexity = ql.BondFunctions.convexity(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency)

yieldDuration = LprBondFunctions.yieldDuration(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency)

yieldConvexity = LprBondFunctions.yieldConvexity(
    bond,
    bondYield,
    accrualDayCounter,
    compounding,
    frequency)

compTab.add_column(
    '解析结果',
    [spreadDuration, spreadConvexity, yieldDuration, yieldConvexity])

bp = 0.01 / 100.0

nextLprQuote.setValue(nextLpr + bp)
dp1 = bond.dirtyPrice()
nextLprQuote.setValue(nextLpr - bp)
dp2 = bond.dirtyPrice()
nextLprQuote.setValue(nextLpr)

yieldDuration = -(dp1 - dp2) / (2.0 * dirtyPrice * bp)
yieldConvexity = (dp1 + dp2 - 2.0 * dirtyPrice) / (dirtyPrice * bp ** 2)

basisSpreadQuote.setValue(basisSpread + bp)
dp1 = bond.dirtyPrice()
basisSpreadQuote.setValue(basisSpread - bp)
dp2 = bond.dirtyPrice()
basisSpreadQuote.setValue(basisSpread)

spreadDuration = -(dp1 - dp2) / (2.0 * dirtyPrice * bp)
spreadConvexity = (dp1 + dp2 - 2.0 * dirtyPrice) / (dirtyPrice * bp ** 2)

compTab.add_column(
    '数值结果',
    [spreadDuration, spreadConvexity, yieldDuration, yieldConvexity])

compTab.float_format = '.8'
print(compTab)

'''
+----------+-------------+-------------+
|   项目   |   解析结果  |   数值结果  |
+----------+-------------+-------------+
| 利差久期 |  4.37254790 |  4.37254808 |
| 利差凸性 | 21.08466334 | 21.08466386 |
| 利率久期 |  0.17188881 |  0.17188881 |
| 利率凸性 | -0.11051093 | -0.11051092 |
+----------+-------------+-------------+
'''
```

和中债登的结果比较一下。

| 项目          | 值      |
| ------------- | ------- |
| 估价收益率(%) | 3.7179  |
| 估价利差凸性  | 21.0847 |
| 估价全价      | 97.3803 |
| 点差收益率(%) | -0.1321 |
| 估价利差久期  | 4.3725  |
| 估价利率久期  | 0.1719  |
| 估价利差凸性  | 21.0847 |
| 估价利率凸性  | 空      |

![](/img/meme/ok.gif)

## 下一步

上述实践虽然成功，但实属非常规的做法，正规的做法是在 C++ 源代码层面上为 `FloatingRateBond`、`PricingEngine` 和 `BondFunctions` 分别创建派生类用于挂钩 LPR 浮息债的计算，下一步将尝试这种做法。

## 参考文献

* 《浮动利率债券收益率计算与风险分析》
* 《浮动利率债券久期和凸性的研究》
* 《浮动利率债券的基准利率选择及定价》
* 《中债价格指标产品久期基本计算方法》
* 《浮动利率债券定价的理论与实践》
