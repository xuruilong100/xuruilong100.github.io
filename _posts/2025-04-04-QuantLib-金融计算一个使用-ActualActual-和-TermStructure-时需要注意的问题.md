---
title: QuantLib 金融计算——一个使用 ActualActual 和 TermStructure 时需要注意的问题
date: 2025-04-04 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: ActAct 是一个非常特殊的天数计算规则，这里记录一个和期限结构有关的使用问题。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——一个使用 `ActualActual` 和 `TermStructure` 时需要注意的问题

## 问题

`ActualActual` 是分析债券时最常用的天数计算规则，根据 StackExchange 上的讨论（<https://quant.stackexchange.com/questions/12707/pricing-a-fixedratebond-in-quantlib-yield-vs-termstructure>），在使用 `ActualActual` 时最好附加上债券的 `Schedule` 对象，否则在计算贴现因子的时候可能产生偏差。

然而，当为 `ActualActual` 对象附加 `Schedule` 对象之后，`ActualActual` 对象能正确处理的日期范围就是 `Schedule` 对象的日期范围。没有附加 `Schedule` 对象的 `ActualActual` 对象几乎没有限制。

深入到代码细节，较新版本的 QuantLib 中，

```c++
Time ActualActual::ISMA_Impl::yearFraction(const Date& d1,
                                           const Date& d2,
                                           const Date& d3,
                                           const Date& d4) const {
    if (d1 == d2) {
        return 0.0;
    } else if (d2 < d1) {
        return -yearFraction(d2, d1, d3, d4);
    }

    std::vector<Date> couponDates =
        getListOfPeriodDatesIncludingQuasiPayments(schedule_);

    Date firstDate = *std::min_element(couponDates.begin(), couponDates.end());
    Date lastDate = *std::max_element(couponDates.begin(), couponDates.end());

    QL_REQUIRE(d1 >= firstDate && d2 <= lastDate, "Dates out of range of schedule: "
                    << "date 1: " << d1 << ", date 2: " << d2 << ", first date: "
                    << firstDate << ", last date: " << lastDate);

    Real yearFractionSum = 0.0;
    for (Size i = 0; i < couponDates.size() - 1; i++) {
        Date startReferencePeriod = couponDates[i];
        Date endReferencePeriod = couponDates[i + 1];
        if (d1 < endReferencePeriod && d2 > startReferencePeriod) {
            yearFractionSum +=
                yearFractionWithReferenceDates(*this,
                                                std::max(d1, startReferencePeriod),
                                                std::min(d2, endReferencePeriod),
                                                startReferencePeriod,
                                                endReferencePeriod);
        }
    }
    return yearFractionSum;
}
```

`ActualActual` 首先会检索 `Schedule` 对象决定的付息日序列，找到 `firstDate` 和 `lastDate`，当要计算的日期超出范围的时候就会报错。这就导致下面的[案例](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%A1%88%E4%BE%8B%E4%B9%8B%E5%9B%BA%E6%81%AF%E5%80%BA%E7%9A%84%E4%BB%B7%E6%A0%BC-%E4%B9%85%E6%9C%9F-%E5%87%B8%E6%80%A7%E5%92%8C-BPS/)无法得到结果，

```c++
#include <iostream>
#include <ql/quantlib.hpp>

void test() {
    using namespace QuantLib;
    auto today = Date(28, July, 2020);
    Settings::instance().evaluationDate() = today;

    Size settlementDays = 1;
    Real faceAmount = 100.0;

    auto effectiveDate = Date(10, March, 2020);
    auto terminationDate = Date(10, March, 2030);
    auto tenor = Period(1, Years);
    auto calendar = China(China::IB);
    auto convention = Unadjusted;
    auto terminationConvention = convention;
    auto rule = DateGeneration::Backward;
    auto endOfMonth = false;

    Schedule schedule(effectiveDate, terminationDate, tenor, calendar,
                      convention, terminationConvention, rule, endOfMonth);

    std::vector<Real> coupons = {3.07 / 100.0};
    auto accrualDayCounter = ActualActual(ActualActual::Bond, schedule);
    auto paymentConvention = Unadjusted;

    auto bond = FixedRateBond(settlementDays, faceAmount, schedule, coupons,
                              accrualDayCounter, paymentConvention);

    Rate bondYield = 3.4124 / 100.0;
    auto compounding = Compounded;
    auto frequency = Annual;

    auto termStructure =
        Handle<YieldTermStructure>(ext::make_shared<FlatForward>(
            settlementDays, calendar, bondYield, accrualDayCounter, compounding,
            frequency));

    auto engine = ext::make_shared<DiscountingBondEngine>(termStructure);

    bond.setPricingEngine(engine);

    auto dirtyPrice = bond.dirtyPrice();
    auto cleanPrice = bond.cleanPrice();
    auto accruedAmount = bond.accruedAmount();

    std::cout << "Clean Price: " << cleanPrice << std::endl;
    std::cout << "Dirty Price: " << dirtyPrice << std::endl;
    std::cout << "Accrued Amount: " << accruedAmount << std::endl;
}

int main() {
    try {
        test();
    } catch (std::exception& e) {
        std::cerr << e.what() << std::endl;
        return EXIT_FAILURE;
    } catch (...) {
        std::cerr << "unknown error" << std::endl;
        return EXIT_FAILURE;
    }
    return EXIT_SUCCESS;
}
```

而是得到报错：

```
Dates out of range of schedule: date 1: July 29th, 2020, date 2: December 31st, 2199, first date: March 10th, 2020, last date: March 10th, 2030
```

`TermStructure` 的所有计算大部分都会调用 `checkRange` 方法，用来检查日期是否超出 `TermStructure` 能处理的范围。在检查的过程中 `TermStructure` 需要用持有的 `DayCounter` 对象计算 `maxDate` 方法返回日期对应的时间。`maxDate` 是 `TermStructure` 的纯虚方法。

`FlatForward` 作为 `TermStructure` 的派生类，需要实现 `maxDate` 方法，并且当前的实现如下：

```c++
Date FlatForward::maxDate() const { return Date::maxDate(); }

Date Date::maxDate() {
    static const Date maximumDate(maximumSerialNumber());
    return maximumDate;
}

Date::serial_type Date::maximumSerialNumber() {
    return 109574;    // Dec 31st, 2199
}
```

> 这就是为什么报错中出现了 `December 31st, 2199`。

## 解决

好在 Luigi 在 GitHub 社区已经提出了[解决办法](https://github.com/lballabio/QuantLib/issues/1613)，只需在 `termStructure` 创建之后加上一句：

```c++
termStructure->enableExtrapolation();
```

**这样在调用 `checkRange` 方法时就会跳过计算 `maxDate` 对应的时间这一步，也就是调用函数 `maxTime`。**

```c++
void TermStructure::checkRange(Time t,
                               bool extrapolate) const {
    QL_REQUIRE(t >= 0.0,
                "negative time (" << t << ") given");
    QL_REQUIRE(extrapolate || allowsExtrapolation()
                || t <= maxTime() || close_enough(t, maxTime()),
                "time (" << t << ") is past max curve time ("
                        << maxTime() << ")");
}
```

按照 Luigi 的方法修改就能得到正确答案：

```
Clean Price: 97.2212
Dirty Price: 98.4071
Accrued Amount: 1.18595
```

添加 `termStructure->enableExtrapolation()` 明显是“打补丁”的行为，更系统性的方法应该是：
1. `DayCounter` 类中添加虚函数 `maxDate`，默认返回 `Date::maxDate`；
2. `TermStructure` 的 `maxDate` 方法直接调用 `DayCounter` 的方法；
3. `DayCounter` 的派生类们可以实现自己的 `maxDate`。

## 扩展阅读

* [《一个使用 `ActualActual` 时需要注意的陷阱》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E4%B8%80%E4%B8%AA%E4%BD%BF%E7%94%A8-ActualActual-%E6%97%B6%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E9%99%B7%E9%98%B1/)
* [《案例之固息债的价格、久期、凸性和 BPS》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%A1%88%E4%BE%8B%E4%B9%8B%E5%9B%BA%E6%81%AF%E5%80%BA%E7%9A%84%E4%BB%B7%E6%A0%BC-%E4%B9%85%E6%9C%9F-%E5%87%B8%E6%80%A7%E5%92%8C-BPS/)