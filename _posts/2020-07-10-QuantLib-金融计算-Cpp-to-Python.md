---
title: QuantLib 金融计算——C++ 代码改写成 Python 程序的一些经验
date: 2020-07-10 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 总结 QuantLib C++ 代码改写成 Python 程序的经验。
---

# QuantLib 金融计算——C++ 代码改写成 Python 程序的一些经验

## 概述

Python 在科学计算、数据分析和可视化等方面已经形成了非常强大的生态。而且，作为一门时尚的脚本语言，易学易用。因此，对于量化分析和风险管理的从业者来说，将某些 QuantLib 的历史代码转换成 Python 程序是一件值得尝试的工作。

Python 本身的面向对象机制非常完善，借助 SWIG 的包装，由 C++ 代码转换而成的 Python 程序基本上可以完整地保留原本的类架构。对于用户来说，应用层面的历史代码几乎可以平行的进行移植，只需稍加修改即可。

本文将以 QuantLib 官方网站上的 [EquityOption.cpp](https://www.quantlib.org/reference/_equity_option_8cpp-example.html) 为例，展示如何将应用层面的 C++ 代码转换成 Python 程序，并总结出一般的转换方法和注意事项。

## 将 C++ 代码改写成 Python 程序

下面，我将逐句把 C++ 代码改写成 Python 程序。

**C++ 代码：**

```c++
#include <ql/qldefines.hpp>
#ifdef BOOST_MSVC
#  include <ql/auto_link.hpp>
#endif
#include <ql/instruments/vanillaoption.hpp>
#include <ql/pricingengines/vanilla/binomialengine.hpp>
#include <ql/pricingengines/vanilla/analyticeuropeanengine.hpp>
#include <ql/pricingengines/vanilla/analytichestonengine.hpp>
#include <ql/pricingengines/vanilla/baroneadesiwhaleyengine.hpp>
#include <ql/pricingengines/vanilla/bjerksundstenslandengine.hpp>
#include <ql/pricingengines/vanilla/batesengine.hpp>
#include <ql/pricingengines/vanilla/integralengine.hpp>
#include <ql/pricingengines/vanilla/fdblackscholesvanillaengine.hpp>
#include <ql/pricingengines/vanilla/mceuropeanengine.hpp>
#include <ql/pricingengines/vanilla/mcamericanengine.hpp>
#include <ql/time/calendars/target.hpp>
#include <ql/utilities/dataformatters.hpp>

#include <iostream>
#include <iomanip>
 
using namespace QuantLib;
 
#if defined(QL_ENABLE_SESSIONS)
namespace QuantLib {
    Integer sessionId() { return 0; }
}
#endif
```

**Python 代码：**

```python
import QuantLib as ql
import prettytable as pt
```

首先，引入必要的模块，对 C++ 来说是一组头文件。Python 的优势显而易见。

---

**C++ 代码：**

```c++
// set up dates
Calendar calendar = TARGET();
Date todaysDate(15, May, 1998);
Date settlementDate(17, May, 1998);
Settings::instance().evaluationDate() = todaysDate;
```

**Python 代码：**

```python
# set up dates
calendar = ql.TARGET()
todaysDate = ql.Date(15, ql.May, 1998)
settlementDate = ql.Date(17, ql.May, 1998)
ql.Settings.instance().evaluationDate = todaysDate
```

C++ 中对象的声明有两种常见的方式：
1. `BaseClass object = Class(...)`，其中 `Class` 可以是 `BaseClass` 本身，或者其派生类。示例中的 `TARGET` 正是 `Calendar` 的派生类；
2. `Class object(...)`。

Python 中无需声明对象类型，而是以赋值的形式创建一个对象，所以对于上述两类格式的代码，统一改写成 `object = Class(...)`。

> 经验 1：对象声明语句 `BaseClass object = Class(...)` 和 `Class object(...)` 统一改写成 `object = Class(...)`。

`Settings` 是 QuantLib 中的一个“单体模式”的实现，通常用来为整个程序设置统一的估值日期，几乎每个应用程序中都会出现。通过调用 `Settings` 的静态方法 `instance()`，用户可以修改单体实例的某些属性，其中 `evaluationDate()` 方法可以把存储估值日期的成员变量地址暴露出来，让用户进行设置。

不过，Python 中的类没有 `::` 运算符，类的方法也不能暴露成员变量的地址。所以，原本的静态方法一律通过 `.` 运算符调用，同时 `evaluationDate()` 方法被重定义为类的 `property`，这就是为什么 Python 语句中 `evaluationDate` 后面没有 `()`。注意，`instance()` 后面的 `()` 不能丢。

> 经验 2：用来对 `Settings::instance()` 进行配置的成员函数，例如 `evaluationDate()`，在 Python 中以类的 `property` 形式出现，不过名称不变。

---

**C++ 代码：**

```c++
// our options
Option::Type type(Option::Put);
Real underlying = 36;
Real strike = 40;
Spread dividendYield = 0.00;
Rate riskFreeRate = 0.06;
Volatility volatility = 0.20;
Date maturity(17, May, 1999);
DayCounter dayCounter = Actual365Fixed();
```

**Python 代码：**

```python
# our options
optType = ql.Option.Put
underlying = 36.0
strike = 40.0
dividendYield = 0.00
riskFreeRate = 0.06
volatility = 0.20
maturity = ql.Date(17, ql.May, 1999)
dayCounter = ql.Actual365Fixed()
```

C++ 中类内部枚举类型的对象声明和类对象声明相似，采用 `Class::Enum object(Class::element)` 的形式。枚举元素本质上是一些整数常量。

SWIG 在包装 QuantLib 的 Python 接口时会把 C++ 类内部的枚举类型转换成 Python 类中的公有属性，其值依然是一些整数值。所以，枚举类型对象的声明就直接改写成赋值语句。因此，`Class::Enum object(Class::element)` 语句统一改写成 `object = Class.element`。

示例中的 `Type` 是 `Option` 类内部的一个枚举型，而 `Put` 是 `Type` 中的一个元素，另一个是 `Call`。因为 `type` 是 Python 的关键字，改写时一定要重命名。

> 经验 3：对于类中的枚举类型，`Class::Enum object(Class::element)` 语句统一改写成 `object = Class.element`。

对于基本类型（整数、浮点数、字符、字符串）来说，改写非常容易。由于 Python 无需声明类型，`Type object = value` 语句统一改写成赋值语句——`object = value`。

> 经验 4：对于基本类型，`Type object = value` 语句统一改写成 `object = value`。

---

**C++ 代码：**

```c++
std::cout << "Option type = " << type << std::endl;
std::cout << "Maturity = " << maturity << std::endl;
std::cout << "Underlying price = " << underlying << std::endl;
std::cout << "Strike = " << strike << std::endl;
std::cout << "Risk-free interest rate = " << io::rate(riskFreeRate) << std::endl;
std::cout << "Dividend yield = " << io::rate(dividendYield) << std::endl;
std::cout << "Volatility = " << io::volatility(volatility) << std::endl;
std::cout << std::endl;
std::string method;
std::cout << std::endl ;

// write column headings
Size widths[] = { 35, 14, 14, 14 };
std::cout << std::setw(widths[0]) << std::left << "Method"
          << std::setw(widths[1]) << std::left << "European"
          << std::setw(widths[2]) << std::left << "Bermudan"
          << std::setw(widths[3]) << std::left << "American"
          << std::endl;
```

**Python 代码：**

```python
print('Option type =', optType)
print('Maturity =', maturity)
print('Underlying price =', underlying)
print('Strike =', strike)
print('Risk-free interest rate =', '{0:%}'.format(riskFreeRate))
print('Dividend yield =', '{0:%}'.format(dividendYield))
print('Volatility =', '{0:%}'.format(volatility))
print()

# show table

tab = pt.PrettyTable(['Method', 'European', 'Bermudan', 'American'])
```

字符串输出部分没什么好说的，我使用了 `prettytable` 包来美化输出结果。

---

**C++ 代码：**

```c++
std::vector<Date> exerciseDates;
for (Integer i = 1; i <= 4; i++)
    exerciseDates.push_back(settlementDate + 3 * i * Months);
```

**Python 代码：**

```python
exerciseDates = ql.DateVector()
for i in range(1, 5):
    exerciseDates.push_back(settlementDate + ql.Period(3 * i, ql.Months))
```

Python 本身没有“模板”的概念，因此 SWIG 只能对模板的实例化进行包装（模板的实例化就是一个具体的类），进而得到一些 Python 类。对于某些常用类型，例如 `Date`，QuantLib 的 Python 接口包装了对应的 `std::vector` 模板的实例化，包装后得到的 Python 类有一致的命名格式——`ClassVector`，对于 `std::vector<Date>` 而言就是 `DateVector`。

因为模板的实例化实际上就是一个具体的类，因此，这部分代码的改写方法遵循**经验 1**。

和 C++ 完全不同，Python 不是一个“强类型”的语言，在改写涉及**隐式转换**的代码时要格外注意。`Months` 是 QuantLib 中的枚举类型 `TimeUnit` 的元素，SWIG 在包装枚举类型时会将元素转换成 Python 中的整数，丢失了 `TimeUnit` 的类型信息。由于 Python 不是强类型的，被包装的枚举类型会丢失类型信息，因此，`3 * i * Months` 在 C++ 中可以顺利地隐式转换成一个 `Period` 对象——`Period(3 * i, Months)`，但是，在 Python 中 `3 * i * Months` 只会被当做三个整数相乘。此时，`3 * i * Months` 必须改写成显式声明的格式——`ql.Period(3 * i, ql.Months)`。

> 经验 5：隐式转换成 `Period` 对象的代码在改写时要改成显式声明的格式，这类代码通常与枚举类型 `TimeUnit` 有关。

---

**C++ 代码：**

```c++
ext::shared_ptr<Exercise> europeanExercise(
    new EuropeanExercise(maturity));

ext::shared_ptr<Exercise> bermudanExercise(
    new BermudanExercise(exerciseDates));

ext::shared_ptr<Exercise> americanExercise(
    new AmericanExercise(settlementDate, maturity));
```

**Python 代码：**

```python
europeanExercise = ql.EuropeanExercise(maturity)
bermudanExercise = ql.BermudanExercise(exerciseDates)
americanExercise = ql.AmericanExercise(settlementDate, maturity)
```

C++ 中声明智能指针的最常见方式是：`shared_ptr<BaseClass> object(new Class(...))`（`shared_ptr` 也是最常用的智能指针类模板），其中 `Class` 可以是 `BaseClass` 本身，或者其派生类。示例中的 `EuropeanExercise` 正是 `Exercise` 的派生类。这类代码在 Python 中统一改写成声明对象的形式——`object = Class(...)`，因为智能指针通常被视为一个对象。

> 经验 6：对于智能指针，`shared_ptr<BaseClass> object(new Class(...))` 统一改写成 `object = Class(...)`。

---

**C++ 代码：**

```c++
Handle<Quote> underlyingH(
    ext::shared_ptr<Quote>(new SimpleQuote(underlying)));
```

**Python 代码：**

```python
underlyingH = ql.QuoteHandle(ql.SimpleQuote(underlying))
```

`Quote` 类和 `Handle` 模板是 QuantLib 中最常用到的两个类（模板），它们通常充当“观察者模式”中被观察的一方，一般被当做参数来配置更复杂类的实例。`Quote` 类接受一个浮点数做参数，而 `Handle` 模板接受一个智能指针。当用户修改 `Quote` 实例的值，或 `Handle` 实例指向的指针之后，那些接受过这些实例的复杂类对象会接到通知，并自动触发相关计算。这个机制非常赞！

关于 `Quote` 的具体使用案例，详情可以参考[《`Quote` 带来的便利》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%A1%88%E4%BE%8B%E4%B9%8B%E6%99%AE%E9%80%9A%E6%AC%A7%E5%BC%8F%E6%9C%9F%E6%9D%83%E5%88%86%E6%9E%90/#quote-%E5%B8%A6%E6%9D%A5%E7%9A%84%E4%BE%BF%E5%88%A9)。

QuantLib 的 Python 接口已经包装了 `Handle` 模板的一些实例化，例如 `QuoteHandle` 和下面将要看到的 `YieldTermStructureHandle`，这些类有一致的命名格式——`ClassHandle`。

还是那句话，C++ 模板的实例化实际上就是一个具体的类，因此，这部分代码的改写方法遵循**经验 1** 和**经验 6**。

---

**C++ 代码：**

```c++
// bootstrap the yield/dividend/vol curves
Handle<YieldTermStructure> flatTermStructure(
    ext::shared_ptr<YieldTermStructure>(
        new FlatForward(settlementDate, riskFreeRate, dayCounter)));
Handle<YieldTermStructure> flatDividendTS(
    ext::shared_ptr<YieldTermStructure>(
        new FlatForward(settlementDate, dividendYield, dayCounter)));
Handle<BlackVolTermStructure> flatVolTS(
    ext::shared_ptr<BlackVolTermStructure>(
        new BlackConstantVol(
            settlementDate, calendar, volatility, dayCounter)));
ext::shared_ptr<StrikedTypePayoff> payoff(
    new PlainVanillaPayoff(type, strike));
ext::shared_ptr<BlackScholesMertonProcess> bsmProcess(
    new BlackScholesMertonProcess(
        underlyingH, flatDividendTS, flatTermStructure, flatVolTS));

// options
VanillaOption europeanOption(payoff, europeanExercise);
VanillaOption bermudanOption(payoff, bermudanExercise);
VanillaOption americanOption(payoff, americanExercise);

// Analytic formulas:

// Black-Scholes for European
method = "Black-Scholes";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new AnalyticEuropeanEngine(bsmProcess)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << "N/A"
          << std::endl;

// semi-analytic Heston for European
method = "Heston semi-analytic";
ext::shared_ptr<HestonProcess> hestonProcess(
    new HestonProcess(
        flatTermStructure, flatDividendTS, underlyingH,
        volatility * volatility, 1.0, volatility * volatility, 0.001, 0.0));
ext::shared_ptr<HestonModel> hestonModel(
    new HestonModel(hestonProcess));
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new AnalyticHestonEngine(hestonModel)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << "N/A"
          << std::endl;

// semi-analytic Bates for European
method = "Bates semi-analytic";
ext::shared_ptr<BatesProcess> batesProcess(
    new BatesProcess(
        flatTermStructure, flatDividendTS, underlyingH,
        volatility * volatility, 1.0, volatility * volatility,
        0.001, 0.0, 1e-14, 1e-14, 1e-14));
ext::shared_ptr<BatesModel> batesModel(
    new BatesModel(batesProcess));
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(new BatesEngine(batesModel)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << "N/A"
          << std::endl;
          
// Barone-Adesi and Whaley approximation for American
method = "Barone-Adesi/Whaley";
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BaroneAdesiWhaleyApproximationEngine(bsmProcess)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << "N/A"
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Bjerksund and Stensland approximation for American
method = "Bjerksund/Stensland";
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BjerksundStenslandApproximationEngine(bsmProcess)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << "N/A"
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Integral
method = "Integral";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new IntegralEngine(bsmProcess)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << "N/A"
          << std::endl;

// Finite differences
Size timeSteps = 801;
method = "Finite differences";
ext::shared_ptr<PricingEngine> fdengine =
    ext::make_shared<FdBlackScholesVanillaEngine>(
        bsmProcess, timeSteps, timeSteps - 1);
europeanOption.setPricingEngine(fdengine);
bermudanOption.setPricingEngine(fdengine);
americanOption.setPricingEngine(fdengine);
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;
```

**Python 代码：**

```python
# bootstrap the yield/dividend/vol curves
flatTermStructure = ql.YieldTermStructureHandle(
    ql.FlatForward(settlementDate, riskFreeRate, dayCounter))
flatDividendTS = ql.YieldTermStructureHandle(
    ql.FlatForward(settlementDate, dividendYield, dayCounter))
flatVolTS = ql.BlackVolTermStructureHandle(
    ql.BlackConstantVol(
        settlementDate, calendar, volatility, dayCounter))
payoff = ql.PlainVanillaPayoff(optType, strike)
bsmProcess = ql.BlackScholesMertonProcess(
    underlyingH, flatDividendTS, flatTermStructure, flatVolTS)

# options
europeanOption = ql.VanillaOption(payoff, europeanExercise)
bermudanOption = ql.VanillaOption(payoff, bermudanExercise)
americanOption = ql.VanillaOption(payoff, americanExercise)

# Analytic formulas:

# Black-Scholes for European
method = 'Black-Scholes'
europeanOption.setPricingEngine(
    ql.AnalyticEuropeanEngine(bsmProcess))
tab.add_row([method, europeanOption.NPV(), 'N/A', 'N/A'])

# semi-analytic Heston for European
method = 'Heston semi-analytic'
hestonProcess = ql.HestonProcess(
    flatTermStructure, flatDividendTS, underlyingH,
    volatility * volatility, 1.0, volatility * volatility, 0.001, 0.0)
hestonModel = ql.HestonModel(hestonProcess)
europeanOption.setPricingEngine(
    ql.AnalyticHestonEngine(hestonModel))
tab.add_row([method, europeanOption.NPV(), 'N/A', 'N/A'])

# semi-analytic Bates for European
method = 'Bates semi-analytic'
batesProcess = ql.BatesProcess(
    flatTermStructure, flatDividendTS, underlyingH,
    volatility * volatility, 1.0, volatility * volatility,
    0.001, 0.0, 1e-14, 1e-14, 1e-14)
batesModel = ql.BatesModel(batesProcess)
europeanOption.setPricingEngine(
    ql.BatesEngine(batesModel))
tab.add_row([method, europeanOption.NPV(), 'N/A', 'N/A'])

# Barone-Adesi and Whaley approximation for American
method = 'Barone-Adesi/Whaley'
americanOption.setPricingEngine(
    ql.BaroneAdesiWhaleyEngine(bsmProcess))
tab.add_row([method, 'N/A', 'N/A', americanOption.NPV()])

# Bjerksund and Stensland approximation for American
method = 'Bjerksund/Stensland'
americanOption.setPricingEngine(
    ql.BjerksundStenslandEngine(bsmProcess))
tab.add_row([method, 'N/A', 'N/A', americanOption.NPV()])

# Integral
method = 'Integral'
europeanOption.setPricingEngine(
    ql.IntegralEngine(bsmProcess))
tab.add_row([method, europeanOption.NPV(), 'N/A', 'N/A'])

# Finite differences
timeSteps = 801
method = 'Finite differences'
fdengine = ql.FdBlackScholesVanillaEngine(bsmProcess, timeSteps, timeSteps - 1)
europeanOption.setPricingEngine(fdengine)
bermudanOption.setPricingEngine(fdengine)
americanOption.setPricingEngine(fdengine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])
```

这部分代码的改写没什么新意，需要注意的是，某些**非模板类**在被包装时会被重命名，例如 `BaroneAdesiWhaleyApproximationEngine` 被重命名为 `BaroneAdesiWhaleyEngine`。如果用户根据前面的 6 条经验找不到 Python 接口中的对应物，那么，要改写的 C++ 代码可能遇到了重命名的情况。这时，用户需要到 QuantLib-SWIG 的[接口文件](https://github.com/lballabio/QuantLib-SWIG/tree/master/SWIG)中查找 C++ 类（结构体）或函数，看看有没有被重命名。继续前面的例子，SWIG 代码 `%rename(BaroneAdesiWhaleyEngine) BaroneAdesiWhaleyApproximationEngine;` 表明 `BaroneAdesiWhaleyApproximationEngine` 被重命名为 `BaroneAdesiWhaleyEngine`。

> 经验 7：疑似遇到重命名的情况（常见于名字特别长的类），到 QuantLib-SWIG 的[接口文件](https://github.com/lballabio/QuantLib-SWIG/tree/master/SWIG)中查找重命名命令。

---

**C++ 代码：**

```c++
// Binomial method: Jarrow-Rudd
method = "Binomial Jarrow-Rudd";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<JarrowRudd>(bsmProcess, timeSteps)));
bermudanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<JarrowRudd>(bsmProcess, timeSteps)));
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<JarrowRudd>(bsmProcess, timeSteps)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Binomial method: Cox-Ross-Rubinstein
method = "Binomial Cox-Ross-Rubinstein";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<CoxRossRubinstein>(bsmProcess, timeSteps)));
bermudanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<CoxRossRubinstein>(bsmProcess, timeSteps)));
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<CoxRossRubinstein>(bsmProcess, timeSteps)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Binomial method: Additive equiprobabilities
method = "Additive equiprobabilities";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<AdditiveEQPBinomialTree>(
            bsmProcess, timeSteps)));
bermudanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<AdditiveEQPBinomialTree>(
            bsmProcess, timeSteps)));
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<AdditiveEQPBinomialTree>(
            bsmProcess, timeSteps)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Binomial method: Binomial Trigeorgis
method = "Binomial Trigeorgis";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Trigeorgis>(bsmProcess, timeSteps)));
bermudanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Trigeorgis>(bsmProcess, timeSteps)));
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Trigeorgis>(bsmProcess, timeSteps)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Binomial method: Binomial Tian
method = "Binomial Tian";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Tian>(bsmProcess, timeSteps)));
bermudanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Tian>(bsmProcess, timeSteps)));
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Tian>(bsmProcess, timeSteps)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Binomial method: Binomial Leisen-Reimer
method = "Binomial Leisen-Reimer";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<LeisenReimer>(bsmProcess, timeSteps)));
bermudanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<LeisenReimer>(bsmProcess, timeSteps)));
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<LeisenReimer>(bsmProcess, timeSteps)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;

// Binomial method: Binomial Joshi
method = "Binomial Joshi";
europeanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Joshi4>(bsmProcess, timeSteps)));
bermudanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Joshi4>(bsmProcess, timeSteps)));
americanOption.setPricingEngine(
    ext::shared_ptr<PricingEngine>(
        new BinomialVanillaEngine<Joshi4>(bsmProcess, timeSteps)));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << bermudanOption.NPV()
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;
```

**Python 代码：**

```python
# Binomial method: Jarrow-Rudd
method = 'Binomial Jarrow-Rudd'
jrengine = ql.BinomialJRVanillaEngine(bsmProcess, timeSteps)
europeanOption.setPricingEngine(jrengine)
bermudanOption.setPricingEngine(jrengine)
americanOption.setPricingEngine(jrengine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])

# Binomial method: Cox-Ross-Rubinstein
method = 'Binomial Cox-Ross-Rubinstein'
crrengine = ql.BinomialCRRVanillaEngine(bsmProcess, timeSteps)
europeanOption.setPricingEngine(crrengine)
bermudanOption.setPricingEngine(crrengine)
americanOption.setPricingEngine(crrengine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])

# Binomial method: Additive equiprobabilities
method = 'Additive equiprobabilities'
eqpengine = ql.BinomialEQPVanillaEngine(bsmProcess, timeSteps)
europeanOption.setPricingEngine(eqpengine)
bermudanOption.setPricingEngine(eqpengine)
americanOption.setPricingEngine(eqpengine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])

# Binomial method: Binomial Trigeorgis
method = 'Binomial Trigeorgis'
trengine = ql.BinomialTrigeorgisVanillaEngine(bsmProcess, timeSteps)
europeanOption.setPricingEngine(trengine)
bermudanOption.setPricingEngine(trengine)
americanOption.setPricingEngine(trengine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])

# Binomial method: Binomial Tian
method = 'Binomial Tian'
tiengine = ql.BinomialTianVanillaEngine(bsmProcess, timeSteps)
europeanOption.setPricingEngine(tiengine)
bermudanOption.setPricingEngine(tiengine)
americanOption.setPricingEngine(tiengine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])

# Binomial method: Binomial Leisen-Reimer
method = 'Binomial Leisen-Reimer'
lrengine = ql.BinomialLRVanillaEngine(bsmProcess, timeSteps)
europeanOption.setPricingEngine(lrengine)
bermudanOption.setPricingEngine(lrengine)
americanOption.setPricingEngine(lrengine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])

# Binomial method: Binomial Joshi
method = 'Binomial Joshi'
j4engine = ql.BinomialJ4VanillaEngine(bsmProcess, timeSteps)
europeanOption.setPricingEngine(j4engine)
bermudanOption.setPricingEngine(j4engine)
americanOption.setPricingEngine(j4engine)
tab.add_row([method, europeanOption.NPV(), bermudanOption.NPV(), americanOption.NPV()])
```

对于 C++ 中的模板，SWIG 在包装 Python 接口时只包装模板的实例化，并且会为模板的实例化取一个新名字。这时，用户需要到 QuantLib-SWIG 的[接口文件](https://github.com/lballabio/QuantLib-SWIG/tree/master/SWIG)中查找模板的实例化，看看取了什么新名字。继续前面的例子，SWIG 代码 `%template(BinomialJRVanillaEngine) BinomialVanillaEngine<JarrowRudd>;` 表示 `BinomialVanillaEngine<JarrowRudd>` 在 Python 中对应的类叫做 `BinomialJRVanillaEngine`。

> 经验 8：遇到模板实例化的情况，到 QuantLib-SWIG 的[接口文件](https://github.com/lballabio/QuantLib-SWIG/tree/master/SWIG)中查找实例化后新的类名。

---

**C++ 代码：**

```c++
// Monte Carlo Method: MC (crude)
timeSteps = 1;
method = "MC (crude)";
Size mcSeed = 42;
ext::shared_ptr<PricingEngine> mcengine1;
mcengine1 = MakeMCEuropeanEngine<PseudoRandom>(
                bsmProcess)
                .withSteps(timeSteps)
                .withAbsoluteTolerance(0.02)
                .withSeed(mcSeed);
europeanOption.setPricingEngine(mcengine1);
// Real errorEstimate = europeanOption.errorEstimate();
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << "N/A"
          << std::endl;

// Monte Carlo Method: QMC (Sobol)
method = "QMC (Sobol)";
Size nSamples = 32768;    // 2^15
ext::shared_ptr<PricingEngine> mcengine2;
mcengine2 = MakeMCEuropeanEngine<LowDiscrepancy>(
                bsmProcess)
                .withSteps(timeSteps)
                .withSamples(nSamples);
europeanOption.setPricingEngine(mcengine2);
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << "N/A"
          << std::endl;

// Monte Carlo Method: MC (Longstaff Schwartz)
method = "MC (Longstaff Schwartz)";
ext::shared_ptr<PricingEngine> mcengine3;
mcengine3 = MakeMCAmericanEngine<PseudoRandom>(
                bsmProcess)
                .withSteps(100)
                .withAntitheticVariate()
                .withCalibrationSamples(4096)
                .withAbsoluteTolerance(0.02)
                .withSeed(mcSeed);
americanOption.setPricingEngine(mcengine3);
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << "N/A"
          << std::setw(widths[2]) << std::left << "N/A"
          << std::setw(widths[3]) << std::left << americanOption.NPV()
          << std::endl;
```

**Python 代码：**

```python
timeSteps = 1
# Monte Carlo Method: MC (crude)
method = 'MC (crude)'
mcSeed = 42
mcengine1 = ql.MCPREuropeanEngine(
    bsmProcess,
    timeSteps=timeSteps,
    requiredTolerance=0.02,
    seed=mcSeed)

europeanOption.setPricingEngine(mcengine1)
tab.add_row([method, europeanOption.NPV(), 'N/A', 'N/A'])

# Monte Carlo Method: QMC (Sobol)
method = 'QMC (Sobol)'
nSamples = 32768  # 2^15
mcengine2 = ql.MCLDEuropeanEngine(
    bsmProcess,
    timeSteps=timeSteps,
    requiredSamples=nSamples)
europeanOption.setPricingEngine(mcengine2)
tab.add_row([method, europeanOption.NPV(), 'N/A', 'N/A'])

# Monte Carlo Method: MC (Longstaff Schwartz)
method = 'MC (Longstaff Schwartz)'
mcengine3 = ql.MCPRAmericanEngine(
    bsmProcess,
    timeSteps=100,
    antitheticVariate=True,
    nCalibrationSamples=4096,
    requiredTolerance=0.02,
    seed=mcSeed)
americanOption.setPricingEngine(mcengine3)
tab.add_row([method, 'N/A', 'N/A', americanOption.NPV()])

tab.float_format = '.6'
tab.align = 'l'
print(tab)
```

`MakeMCEuropeanEngine<PseudoRandom>` 是 QuantLib 中工厂模式的一个实现，对于拥有较多默认参数的类，QuantLib 会提供一个对应的工厂类，用户借助工厂类“制造”一个半成品对象，并通过一组成员函数以流水线的方式配置这个半成品的参数，以实现对默认参数的灵活配置。这些流水线函数有一致的命名格式——`withArgument`，`Argument` 通常是某个默认参数的名字。这套机制也被称为“命名参数惯用法”。这些工厂类有一致的命名规范——`MakeClass`，其中 `Class` 是一个类的名字或实例化的模板，`MakeClass` 将制造出一个 `Class` 对象。

Python 中存在“关键字参数”的机制，因此，上述“流水线函数”显得非常笨拙，对于这类代码的改写，用户只要知道“`MakeClass` 将制造出一个 `Class` 对象”这一点，并理解流水线函数所配置的参数，然后应用前面总结的 8 条经验就可以成功改写。

> 经验 9：名为 `MakeClass` 的工厂类将制造出一个 `Class` 对象，后续的成员函数表示配置的参数。

### 对比结果

**C++ 代码运行结果：**

```
Option type = Put
Maturity = May 17th, 1999
Underlying price = 36
Strike = 40
Risk-free interest rate = 6.000000 %
Dividend yield = 0.000000 %
Volatility = 20.000000 %


Method                             European      Bermudan      American      
Black-Scholes                      3.844308      N/A           N/A           
Heston semi-analytic               3.844306      N/A           N/A           
Bates semi-analytic                3.844306      N/A           N/A           
Barone-Adesi/Whaley                N/A           N/A           4.459628      
Bjerksund/Stensland                N/A           N/A           4.453064      
Integral                           3.844309      N/A           N/A           
Finite differences                 3.844330      4.360765      4.486113      
Binomial Jarrow-Rudd               3.844132      4.361174      4.486552      
Binomial Cox-Ross-Rubinstein       3.843504      4.360861      4.486415      
Additive equiprobabilities         3.836911      4.354455      4.480097      
Binomial Trigeorgis                3.843557      4.360909      4.486461      
Binomial Tian                      3.844171      4.361176      4.486413      
Binomial Leisen-Reimer             3.844308      4.360713      4.486076      
Binomial Joshi                     3.844308      4.360713      4.486076      
MC (crude)                         3.834522      N/A           N/A           
QMC (Sobol)                        3.844613      N/A           N/A           
MC (Longstaff Schwartz)            N/A           N/A           4.456935      
```

---

**Python 程序运行结果：**

```
Option type = -1
Maturity = May 17th, 1999
Underlying price = 36.0
Strike = 40.0
Risk-free interest rate = 6.000000%
Dividend yield = 0.000000%
Volatility = 20.000000%

+------------------------------+----------+----------+----------+
| Method                       | European | Bermudan | American |
+------------------------------+----------+----------+----------+
| Black-Scholes                | 3.844308 | N/A      | N/A      |
| Heston semi-analytic         | 3.844306 | N/A      | N/A      |
| Bates semi-analytic          | 3.844306 | N/A      | N/A      |
| Barone-Adesi/Whaley          | N/A      | N/A      | 4.459628 |
| Bjerksund/Stensland          | N/A      | N/A      | 4.453064 |
| Integral                     | 3.844309 | N/A      | N/A      |
| Finite differences           | 3.844330 | 4.360765 | 4.486113 |
| Binomial Jarrow-Rudd         | 3.844132 | 4.361174 | 4.486552 |
| Binomial Cox-Ross-Rubinstein | 3.843504 | 4.360861 | 4.486415 |
| Additive equiprobabilities   | 3.836911 | 4.354455 | 4.480097 |
| Binomial Trigeorgis          | 3.843557 | 4.360909 | 4.486461 |
| Binomial Tian                | 3.844171 | 4.361176 | 4.486413 |
| Binomial Leisen-Reimer       | 3.844308 | 4.360713 | 4.486076 |
| Binomial Joshi               | 3.844308 | 4.360713 | 4.486076 |
| MC (crude)                   | 3.834522 | N/A      | N/A      |
| QMC (Sobol)                  | 3.844613 | N/A      | N/A      |
| MC (Longstaff Schwartz)      | N/A      | N/A      | 4.456935 |
+------------------------------+----------+----------+----------+
```

完全一样！

![](/img/meme/ok.gif)

## 总结

* 经验 1：对象声明语句 `BaseClass object = Class(...)` 和 `Class object(...)` 统一改写成 `object = Class(...)`。
* 经验 2：用来对 `Settings::instance()` 进行配置的成员函数，例如 `evaluationDate()`，在 Python 中以类的 `property` 形式出现，不过名称不变。
* 经验 3：对于类中的枚举类型，`Class::Enum object(Class::element)` 语句统一改写成 `object = Class.element`。
* 经验 4：对于基本类型，`Type object = value` 语句统一改写成 `object = value`。
* 经验 5：隐式转换成 `Period` 对象的代码在改写时要改成显式声明的格式，这类代码通常与枚举类型 `TimeUnit` 有关。
* 经验 6：对于智能指针，`shared_ptr<BaseClass> object(new Class(...))` 统一改写成 `object = Class(...)`。
* 经验 7：疑似遇到重命名的情况（常见于名字特别长的类），到 QuantLib-SWIG 的[接口文件](https://github.com/lballabio/QuantLib-SWIG/tree/master/SWIG)中查找重命名命令。
* 经验 8：遇到模板实例化的情况，到 QuantLib-SWIG 的[接口文件](https://github.com/lballabio/QuantLib-SWIG/tree/master/SWIG)中查找实例化后新的类名。
* 经验 9：名为 `MakeClass` 的工厂类将制造出一个 `Class` 对象，后续的成员函数表示配置的参数。

需要注意的是，QuantLib 中并非所有的功能都有对应的 Python 接口，如果用户需要的功能未被包装，用户只好修改 SWIG 代码，自行生成 Python 接口，可以参考一下文章：
* [《自己动手封装 Python 接口（1）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%B0%81%E8%A3%85-Python-%E6%8E%A5%E5%8F%A3-1/)
* [《自己动手封装 Python 接口（2）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%B0%81%E8%A3%85-Python-%E6%8E%A5%E5%8F%A3-2/)
* [《自己动手封装 Python 接口（3）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%B0%81%E8%A3%85-Python-%E6%8E%A5%E5%8F%A3-3/)
