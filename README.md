# 每个程序员都应该了解的内存知识

> **What Every Programmer Should Know About Memory**

| 项目 | 内容 |
|---|---|
| 作者 | Ulrich Drepper（Red Hat, Inc.） |
| 原文版本 | 1.0，2007 年 11 月 21 日 |
| 原文篇幅 | 114 页 |
| 译者 | 简体中文整理版（见下文“译本说明”） |
| 文档结构 | 按原书章节拆分为 14 个 Markdown 文档 |

## 摘要

随着 CPU 核心变得更快、数量也更多，对大多数程序而言，当前乃至未来一段时间的限制因素都是内存访问。硬件设计者已经提出了越来越复杂的内存处理与加速技术，例如 CPU 缓存；但如果没有程序员的配合，这些技术无法发挥最佳效果。遗憾的是，大多数程序员既不清楚计算机内存子系统的结构，也不清楚使用内存子系统或 CPU 缓存的代价。

本文解释现代商用硬件中内存子系统的结构，说明 CPU 缓存为何出现、如何工作，以及程序应当怎样利用它们来获得最佳性能。

## 目录

| 章节 | 文档 | 原 PDF 页码 |
|---|---|---|
| 第 1 章 引言 | [01-引言.md](chapters/01-引言.md) | 1–2 |
| 第 2 章 现代商用硬件 | [02-现代商用硬件.md](chapters/02-现代商用硬件.md) | 3–12 |
| 第 3 章 CPU 缓存 | [03-CPU缓存.md](chapters/03-CPU缓存.md) | 13–36 |
| 第 4 章 虚拟内存 | [04-虚拟内存.md](chapters/04-虚拟内存.md) | 37–42 |
| 第 5 章 NUMA 支持 | [05-NUMA支持.md](chapters/05-NUMA支持.md) | 43–46 |
| 第 6 章 程序员能做些什么 | [06-程序员能做什么.md](chapters/06-程序员能做什么.md) | 47–77 |
| 第 7 章 内存性能工具 | [07-内存性能工具.md](chapters/07-内存性能工具.md) | 78–88 |
| 第 8 章 未来的技术 | [08-未来技术.md](chapters/08-未来技术.md) | 89–96 |
| 附录 A 示例与基准测试程序 | [09-附录A-示例与基准测试程序.md](chapters/09-附录A-示例与基准测试程序.md) | 97–101 |
| 附录 B 一些 OProfile 提示 | [10-附录B-OProfile提示.md](chapters/10-附录B-OProfile提示.md) | 102–104 |
| 附录 C 内存类型 | [11-附录C-内存类型.md](chapters/11-附录C-内存类型.md) | 105–107 |
| 附录 D libNUMA 简介 | [12-附录D-libNUMA简介.md](chapters/12-附录D-libNUMA简介.md) | 108–110 |
| 参考文献 | [13-参考文献.md](chapters/13-参考文献.md) | 113 |
| 修订历史 | [14-修订历史.md](chapters/14-修订历史.md) | 114 |

原书的索引（附录 E）在 Markdown 中意义不大，未单独收录。

## 译本说明

- 本译本依据作者 2007 年 11 月 21 日发布的 1.0 版 PDF 整理，正文、脚注、图注、代码清单、表格与参考文献均按原书顺序编排。
- 第 1–7 章及附录 A.1 的译文，在 [sysprog21/cpumemory-zhtw](https://github.com/sysprog21/cpumemory-zhtw) 繁体中文社区译本的基础上做了简体化、术语统一、标点调整与排版重构；第 8 章、附录 A.2–A.3、B、C、D、G 依据英文原版新译。繁简转换使用了 [OpenCC](https://github.com/BYVoid/OpenCC) 的字符与词组表。
- 原文中的图表已尽可能从 PDF 中裁出，放在 [`assets/figures/`](assets/figures/) 下并以相对路径嵌入；第 8 章的代码清单以带语法高亮的代码块呈现，附录 C 的图 C.1、C.2 为原 PDF 页面裁剪。
- 公式使用 LaTeX 语法（`$...$`、`$$...$$`）。建议使用支持 MathJax/KaTeX 的 Markdown 阅读器（如 Typora、Obsidian、VS Code + Markdown Preview Enhanced）查看。
- 原文中的英文术语在首次出现时尽量保留，以便对照；后续使用统一的中文译名。寄存器名、函数名、命令、代码标识符保持原样。

## 术语约定

| 英文 | 中文 |
|---|---|
| memory | 内存 |
| main memory | 主存 |
| cache / cache line | 缓存 / 缓存行 |
| cache miss | 缓存未命中 |
| virtual memory | 虚拟内存 |
| page table | 页表 |
| TLB | 转译后备缓冲器（保留 TLB） |
| NUMA | 非统一内存访问（保留 NUMA） |
| prefetch | 预取 |
| atomic operation | 原子操作 |
| transactional memory | 事务内存 |
| lock-free | 无锁 |
| false sharing | 伪共享 |
| huge page | 大页 |
| DRAM / SRAM | 动态随机存取存储器 / 静态随机存取存储器 |

## 版权与使用说明

原文版权归 Ulrich Drepper 所有：

> Copyright © 2007 Ulrich Drepper  
> All rights reserved. No redistribution allowed.

本简体中文译本仅供个人学习、研究和技术参考使用，不得用于商业用途，也不得再次分发。原文中的图表、代码与文字版权均归原作者所有。若需公开转载或用于其他用途，请先获得原作者许可。

## 反馈

译文中的术语不统一、漏译或排版问题，欢迎在本地文档上直接批注。技术内容以英文原版为准；如中英文表述冲突，请以原 PDF 为准。
