---
title: QuantLib 金融计算——QuantLib 入门
date: 2018-03-16 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 针对 QuantLib 初学者的入门建议。
---

# QuantLib 金融计算——QuantLib 入门

## 简介

纷繁复杂、瞬息万变的金融市场开发出了太多的金融产品，产生了太多的计算问题，这对于 Fintech 来讲是一个巨大的挑战，无论是计算能力上的，还是软件设计上的。好在开源软件界从来都不缺少英雄，QuantLib 正是其中的佼佼者。

[QuantLib](https://www.quantlib.org/) 是一个免费、开源的软件库，旨在为量化金融计算提供一个统一的、综合的软件框架。QuantLib 的源代码由 C++ 编写，得力于 C++ 在面向对象和泛型编程方面强大的表现力，以及对贴近底层所带来的出众执行效率，QuantLib 一方面可以清晰地描述各种复杂的金融产品，同时兼顾了计算速度。

## 主要功能

QuantLib 所提供的功能聚焦在两大领域：
1. **期权定价**以及相关计算；
2. **固定收益产品定价**以及相关计算。

与期权相关的主要内容有：
* 表示亚式期权、欧式期权、美式期权、百慕大期权等等不同种类期权的数据结构；
* 基于解析法、有限差分法、二（三）叉树法和 Monte Carlo 的定价引擎；
* 多种波动率模型，例如 Heston 模型、GARCH 模型和局部波动率模型；
* 校准波动率期限结构的方法。
* ...

与固定收益相关的主要内容有：
* 表示固息债、浮息债、零息债、通胀挂钩债券、利率互换、可转债等等不同种类固定收益产品的数据结构；
* 表示利率期限结构的数据结构；
* 现金流分析；
* 若干种利率曲线的插值方法；
* 若干种计息方法，例如 Actual / 365、Actual / 360、30 / 360 等等。
* ...

## 安装与使用

推荐在 Ubuntu 操作系统下安装和使用 QuantLib ，如果使用的是 Ubuntu 16.04 或 17.04，请先在系统中添加 [Dirk Eddelbuettel](https://dirk.eddelbuettel.com/) 维护的 [PPA](https://launchpad.net/~edd/+archive/ubuntu/misc)，以便轻松地安装最新版本。

```bash
sudo add-apt-repository ppa:edd/misc
sudo apt-get update
```

QuantLib 高度依赖 Boost 库，在安装 QuantLib 之前务必安装 Boost，只需要在终端键入：

```bash
sudo apt-get install libboost-all-dev
```

安装 QuantLib：

```bash
sudo apt-get install libquantlib0-dev libquantlib0v5
```

在 C++ 的 IDE 中配置编译器的连接器和搜索路径，让编译器能够找到文件 `/usr/lib/libQuantLib.so` 和路径 `/usr/include/ql` 就可以探索和使用 QuantLib 了 :)。

## 学习指南

作为金融实务、学术研究和软件设计三者的交叉点，学习和使用 QuantLib 并非一项简单的任务。要掌握这一得力工具，你必须成为一个多面手。

John Hull 编写的 *Risk Management and Financial Institutions* 和 *Options, Futures and other Derivatives* 是两本非常出色的书，能够提供金融实务和学术研究方面足够的基础知识让你可以开始探索 QuantLib。除此之外，QuantLib 提供了一套非常详尽的[文档](https://www.quantlib.org/reference/)，更加深入细致的专业知识可以在这里获得。例如，可以在亚式期权定价引擎类 `AnalyticDiscreteGeometricAverageStrikeAsianEngine` 的[页面](https://www.quantlib.org/reference/class_quant_lib_1_1_analytic_discrete_geometric_average_strike_asian_engine.html)中找到介绍这种定价方法的学术论文，“The formula is from "Asian Option", E. Levy (1997) in "Exotic Options: The State of the Art", edited by L. Clewlow, C. Strickland, pag 65-97”。

> *纸上得来终觉浅*
>
> *绝知此事要躬行*

上手编程、操作软件是掌握 QuantLib 的过程中要面对的一大挑战。

### The HARD Way

QuantLib 的源代码由 C++ 编写，使用 C++ 编程是学习、探索 QuantLib 最直接的方式，不过也是最具挑战性的，因为 C++ 本身是一门非常“硬核”的计算机语言，而且 QuantLib 目前的体量和结构已经很庞大和复杂。

在上手之前，你需要了解、掌握 C++ 编程的基本知识（语法、函数、类、模板和 STL），*C++ Primer* 是一个非常好的开始。想要熟练使用 QuantLib 必须要能够理解其复杂的内部架构，这就需要一点“设计模式”的知识，*Head First Design Patterns* 是入门的不二选择（Gof4 的 *Design Patterns* 过于晦涩了），*Modern C++ Design* 适合进阶。

为了帮助使用者深入了解 QuantLib 的设计细节和思路，QuantLib 的核心作者 [Luigi Ballabio](https://leanpub.com/u/lballabio) 专门编写了 [*Implementing QuantLib*](https://leanpub.com/implementingquantlib)（本书的中译版[《构建 QuantLib》](https://leanpub.com/implementingquantlib-cn)已经在 leanpub.com 出版）。

有了上述知识和技能的准备，就可以从 github 上作者提供的[例子](https://github.com/lballabio/QuantLib)开始了，不要忘了勤快地查看文档。

### The EASY Way

如果要快速上手学习、使用 QuantLib，C++ 就显得过于困难了。鉴于 C++ 版的 QuantLib 取得了巨大的成功，许多开源爱好者把 QuantLib 拓展到了其他语言和软件环境下，在 C#、Java、Perl、Python、Julia、Ruby 和 R 等语言中都可以找到对  QuantLib 的封装；在 Microsoft Excel 和  LibreOffice Calc 中也有 QuantLib 的插件。

在 Ubuntu 环境下，常用的三个扩展分别是：
* Python 封装，QuantLib-Python
* LibreOffice Calc 插件，QuantLibAddin
* R 封装，RQuantLib

遗憾的是，这些扩展不能提供 C++ 版本的全部功能。

#### QuantLib-Python

QuantLib-Python 是三个扩展中做的最好的，尽可能的移植了 C++ 版本的架构和使用方法，提供的功能也是最多的。quantlib-python 的安装十分轻松：

```
pip3 install QuantLib
```

感谢 [Gouthaman Balaraman](https://gouthamanbalaraman.com/pages/about.html) 提供了 quantlib-python 详尽的[范例教程](https://gouthamanbalaraman.com/blog/quantlib-python-tutorials-with-examples.html)，和他编写的书——[*QuantLib Python Cookbook*](https://leanpub.com/quantlibpythoncookbook)。

如果想要扩展 QuantLib-Python 目前的功能，实现定制化，你需要一点 [SWIG](https://www.swig.org/) 的知识用来创建自己的封装。

#### QuantLibAddin

[LibreOffice Calc](https://www.libreoffice.org/discover/calc/) 相当于 Ubuntu 上的 Excel，插件 QuantLibAddin 把 QuantLib 中的部分内容封装，你可以像使用 Excel 内置函数一样在电子表格里调用 QuantLib 的功能。插件的下载地址在[这里](https://extensions.libreoffice.org/extensions/quantlib-addin)，更多详尽的说明请参考 QuantLib Addin 的[主页](https://www.quantlib.org/quantlibaddin/)。

#### RQuantLib

和 Python 相比，R 在面向对象编程方面的能力比较弱，所以 RQuantLib 没有保留 QuantLib 原始的架构和用法，而是将部分功能包装成为函数，相对于 QuantLib-Python 而言，RQuantLib 保留的功能更少。

RQuantLib 的[主页](https://dirk.eddelbuettel.com/code/rquantlib.html)。
