---
title: QuantLib 金融计算——原理之 Bootstrap
date: 2021-01-21 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍 Bootstrap 方法的数学原理和代码架构。
---

> 以下文字源自我对源代码的理解，如有不同意见，欢迎留言讨论或发邮件（`xuruilong100@163.com`）

# QuantLib 金融计算——原理之 Bootstrap

这次新开一个系列，不讲应用案例了，尝试介绍 QuantLib 某些核心功能背后的代码逻辑与数学原理。

## Bootstrap 的原理

通过合约报价推算期限结构的过程称为“bootstrap”，其思想和实践非常类似于理论证明中用到的“数学归纳法”，大体过程如下：
1. 首先将要用到的已知利率和金融工具根据期限升序排列；
2. 假设已经求得期限结构上的第 $n$ 个值——$TS_n$，对应于第 $n$ 个报价；
3. 令 $TS_{n+1}$ 是个待定参数 $x$，并给定一个初值；
4. 用已知的期限结构数据——$TS_1,\dots,TS_n,x$，对第 $n+1$ 个金融工具进行估值；
5. 调整 $x$，使得估值结果与报价达到一致；
6. 此时，$x$ 便是要求解的 $TS_{n+1}$；
7. 以此类推。

其中 $TS$ 可以是即期利率、远期利率、贴现因子三者中的任意一个，而可用的插值方法也有线性插值、样条插值、对数线性插值和常数插值等等。两个维度相互搭配可以产生非常多的组合，QuantLib 通过模板技术实现两个维度的自由搭配，具体选择哪种组合要视业务需要而定。

其他问题语境下的 bootstrap 计算原理类似。

## QuantLib 中的 bootstrap 计算（利率语境）

要在 QuantLib 中 bootstrap 出一条利率曲线所涉及的最核心的类有两个：
* 直接调用的类是 `PiecewiseYieldCurve` 模板类，它直接接受一组报价；
* 另外一个间接调用的类是 `IterativeBootstrap` 模板类，作为 `PiecewiseYieldCurve` 的默认模板参数。

`PiecewiseYieldCurve` 需要三个模板参数，前两个分别确定底层数据和插值方法的选择。比如说 `PiecewiseYieldCurve<Discount, LogLinear>` 表示程序内部 bootstrap 出一条贴现因子曲线，用对数线性法插值得到任意日期上的利率值。

`IterativeBootstrap` 是 `PiecewiseYieldCurve` 需要的第三个模板参数，并且是默认选项。它本身就是个模板类，而它的模板参数正是实例化后的 `PiecewiseYieldCurve`。C++ 模板就是这么神奇！

在整个 bootstrap 计算过程中，实际出苦力的类是 `IterativeBootstrap`，它负责从若干金融工具的报价数据中迭代地求解出利率曲线，而 `PiecewiseYieldCurve` 则是充当 Boss 的角色。

![](/img/meme/boss.jpg)

## 核心代码细节

现在来看看 QuantLib 具体怎么实现 bootstrap 的。

通过一些模板技巧，`PiecewiseYieldCurve` 其实是 `YieldTermStructure` 的派生类，所以它最核心的方法是 `discountImpl`（要了解这一点，请阅读[《构建 QuantLib》](https://leanpub.com/implementingquantlib-cn)）。

```c++
DiscountFactor PiecewiseYieldCurve<C,I,B>::discountImpl(Time t) const
{
    calculate();
    return base_curve::discountImpl(t);
}
```

在返回数据之前，`PiecewiseYieldCurve` 会先调用 `calculate`，bootstrap 的计算就发生在这里。

`PiecewiseYieldCurve` 的 `calculate` 方法直接复用其基类 `LazyObject` 的 `calculate` 方法。这里应用了“模板方法模式”，`calculate` 只起到传达命令的作用，实际的计算任务被重新委派回了 `PiecewiseYieldCurve` 的 `performCalculations` 方法。

```c++
void PiecewiseYieldCurve<C,I,B>::performCalculations() const
{
    // just delegate to the bootstrapper
    bootstrap_.calculate();
}
```

在 `performCalculations` 方法中 `IterativeBootstrap` 的一个实例 `bootstrap_` 最终执行所有计算。

尽管 `IterativeBootstrap` 的 `calculate` 方法代码很长，但毛教员教育我们分析问题要“抓大放小”，其实最核心的代码只有两行，

```c++
if (validData)
    solver_.solve(*errors_[i], accuracy, guess, min, max);
else
    firstSolver_.solve(*errors_[i], accuracy, guess, min, max);
```

`solver_` 和 `firstSolver_` 分别是 `Brent` 和 `FiniteDifferenceNewtonSafe` 对象，用于求解（非）线性函数的根，也就是求解利率。

```c++
errors_[i] = ext::shared_ptr<BootstrapError<Curve> >(
    new BootstrapError<Curve>(ts_, helper, i));
```

`errors_` 是 `BootstrapError` 对象，也是一个函数对象，它负责返回估值结果与实际报价之间的差距，充当待求解的（非）线性函数。

```c++
const ext::shared_ptr<typename Traits::helper>& helper = ts_->instruments_[j];
```

而 `helper` 就是某个传递给 `PiecewiseYieldCurve` 的报价对象，在利率语境下通常是 `FraRateHelper` 或 `SwapRateHelper` 等 `XXXRateHelper` 对象，这些 `XXXRateHelper` 类专门用于 bootstrap 计算。

上述代码便是 QuantLib 对 bootstrap 原理的实现。

## 开放问题：如何实现 FR007 互换的相关分析？

利率互换的分析分为两大类：
1. 对存续合约估计、计算敏感性等；
2. 根据最新合约的报价推算利率期限结构。

FR007 互换的现金流结构和普通的利率互换（例如 Shibor3M 互换）基本一致，每隔三个月交换现金流，但确定浮动端利率的方式有不同。在三个月的付息区间内 FR007 利率要每隔七天确定一次，付息区间内的若干 FR007 利率还需要经过某些计算才能最终确定浮动端利率。

根据上述代码的分析，要想最大限度的复用当前逻辑并最小程度的编写代码，需要在 `helper` 上做文章，可以考虑实现 `SwapRateHelper` 的兄弟类 `FR007SwapRateHelper`，以及配套的 `FR007Swap` 类（作为 `VanillaSwap` 的派生类或兄弟类）。

QuantLib 当前实现的 `ArithmeticAverageOIS` 和 `ArithmeticOISRateHelper` 也许是非常值得效仿的范例。

下一步将尝试按照上述思路实现 C++ 代码。
