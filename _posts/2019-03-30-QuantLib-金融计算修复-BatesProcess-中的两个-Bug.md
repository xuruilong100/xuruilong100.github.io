---
title: QuantLib 金融计算——修复 BatesProcess 中的两个 Bug
date: 2019-03-30 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 修复 BatesProcess 中的两个 Bug。
---

# QuantLib 金融计算——修复 BatesProcess 中的两个 Bug

我发现了 `BatesProcess` 中的两个 Bug：
1. 基类 `HestonProcess::factors` 的返回值取决于差分方法 `discretization_` 的类型，结果可能是 2 或 3，但是 `BatesProcess::factors` 的返回值却仅仅只有 4。
2. 因为 `BatesProcess::factors` 的返回值取决于（基类的）差分方法 `discretization_`，因此 `BatesProcess::evolve` 的内部机制需要一个 `switch-case` 结构来匹配 `HestonProcess::discretization_` 的类型。

目前 Bug 已经提交（[#612](https://github.com/lballabio/QuantLib/issues/612)）,并且由 [klausspanderen](https://github.com/klausspanderen) 修复，将在下一个版本（1.16）中更正。
