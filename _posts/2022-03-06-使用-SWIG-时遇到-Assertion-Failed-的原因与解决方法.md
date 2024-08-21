---
title: 使用 SWIG 时遇到 Assertion Failed 的原因与解决方法
date: 2022-03-06 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [FRM] # TAG names should always be lowercase
description: 发现并解决 32-bit 版本 SWIG 特有的一个问题。
---

# 使用 SWIG 时遇到 Assertion Failed 的原因与解决方法

## 问题

近日在 Windows（64 bit） 上使用 swig 时总是遇到“Assertion Failed”。不仅是我，另一位[用户](https://github.com/olegat)也遇到了同样的[麻烦](https://github.com/swig/swig/issues/1901)。

追踪报错来源发现，swig 在函数 `realloc` 失败之后直接以“断言”的形式处理失败，这才造成了“Assertion Failed”。

究其根源，从官网上下载的 swig.exe 是 32 位的，而在 64 位操作系统上，一个 32 位 exe 能处理的内存有上限——[通常不超过 2Gb](https://docs.microsoft.com/zh-cn/windows/win32/memory/memory-limits-for-windows-releases)。当工程规模扩大之后，swig 生成的 C++ 代码文件可能超过上百万行，在此期间需要占用大量内存，要求太多内存会导致函数 `realloc` 失败。

## 解决

swig 的[维护人员](https://github.com/ojwb)已经开始处理内存的[问题](https://github.com/swig/swig/issues/1901)，相信在新版本可以得到解决。

此外，还有一个“土”办法——用 swig 的源代码重新编译一个 64 位的 exe 替换掉官网发布的文件。
