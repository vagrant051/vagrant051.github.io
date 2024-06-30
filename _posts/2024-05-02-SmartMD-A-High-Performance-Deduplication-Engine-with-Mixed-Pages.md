---
layout: post
toc: true
title: "SmartMD: A High Performance Deduplication Engine with Mixed Pages (summary)"
categories: operating-system
tags: [paper summary]
---

SmartMD：一种具有混合页面的高性能去重引擎

## 概要

大页可以带来系统性能的提升，但是内存去重效果较差；而Linux KSM为了去重效果好，拆分大页，牺牲了大页的优势。如何获取大页带来的性能优势，同时又保持较好的去重效果，这正是SmartMD所要解决的问题。

### 个人理解

1.使用大页（如2MB或更大的内存页）可以减少内存管理开销，提高系统性能。这是因为大页可以减少页表项的数量，从而降低内存访问的延迟和页表查找的开销。内存去重是指将系统中多个相同的内存页合并为一个共享页，从而节省内存。由于大页包含更多的数据，找到完全相同的大页的概率较低，所以大页不利于内存去重。

2.拆分大页面后，同样大小的内存需要更多的TLB条目来映射。例如，将一个2MB页面拆分成4KB页面后，需要512个TLB条目来映射这2MB的内存。
由于TLB的容量有限，更多的条目会占用TLB的空间，导致存储其他页面的条目减少。
TLB命中率下降：当更多的内存页面映射需要更多的TLB条目时，TLB中的条目替换频率会增加，这意味着某个时刻需要的地址映射可能不在TLB中。
由于TLB条目被频繁替换，导致TLB未命中率增加。每次TLB未命中都需要进行页表查找，从而增加内存访问的延迟。

## 动机

许多大页面具有高访问频率但很少有重复的子页面。同时，存在具有许多重复子页面和低访问频率的大页面。

![My helpful screenshot](/assets/SmartMD/1.png)

## 设计

目标是在承载多个虚拟机的服务器上最大化内存节省并保持高内存访问性能。其主要思想是将重复率高的冷大页面拆分以节省内存空间，并在这些页面变热时重新构建拆分的大页面以改善内存访问性能。

## 挑战

SmartMD的重构方法旨在恢复大页面的性能优势，但在实现过程中面临以下挑战：

- 调整和更新大量的页表条目和描述符。
- 解决拆分后内存页面不再连续的问题。
- 处理已被释放或重新分配的子页面。

## SmartMD概述

SmartMD 由三个模块组成，分别是 Monitor、Selector 和 Converter。

### Monitor

![My helpful screenshot](/assets/SmartMD/2.png)

#### 检测页面的访问频率

在监视周期开始时清除所有页面的访问位，并在检查间隔秒后检查每个页面。如果页面的访问位是 1，这是由于在周期内对页面的引用而设置的，则 SmartMD 将其访问频率增加一。否则，页面在上个周期内未被访问，其访问频率减少一。如果一个大页面已经被拆分，我们检查其子页面的频率，并查看其中是否有任何一个大于零。如果是，则将原始大页面的频率增加一。然而，频率值保持在 0 到 N 的范围内，其中 N 是一个正整数，并且不会将其更改超出范围。当系统启动时，我们将页面的频率初始化为 N/2。

#### 检测页面的重复率

当检查一个子页面时，SmartMD 对子页面的内容应用三个哈希函数来计算其相应计数器的索引。如果一个页面第一次被检查（即其记录的签名未被找到），则 SmartMD 将其相应的计数器增加一。否则，如果所有计数器都大于一，则认为此页面是一个重复页面。如果一个页面被修改，SmartMD 将每个当前计数器减一，并将每个新计数器增加一。此外，如果一个页面被释放，SmartMD 也会将每个计数器减一。

### 选择器

为了提高内存访问性能，选择单元根据两个度量（即访问频率和重复率）选择候选大页进行拆分。

选择单元判定大页是冷的还是热的，判定大页的重复率，它的工作流程是：
扫描大页时，选择单元首先读取其访问频率， 如果此页面被指定为冷的，则选择单元将进一步确定其重复率是否大于设定的阈值， 如果是的话，就拆分该页面。 另一方面，当选择已拆分大页进行合并时，选择单元只选择热页作为候选对象。

需要注意的是，温态是冷和热状态之间的过渡状态。我们引入它是为了避免在冷和热状态之间频繁切换。

### 转换器

SmartMD 中实现了一个实用程序来重建拆分的大页面。包括以下三个步骤。

- 收集子页面。为了重建一个拆分的大页面，我们需要确保其所有子页面当前都驻留在内存中，并且没有与其他页面去重。如果一些子页面已经被去重，我们为每个这些子页面生成一个副本，并在重建之前将所有子页面迁移到一个连续的内存空间。

- 编写页面描述符。一旦收集到拆分大页面的所有子页面，我们就会从所有子页面的页面描述符重新创建大页面的页面描述符。

- 编写页面表。我们使用单个页面条目来映射重建的大页面，并使原始子页面的旧条目失效。

为了降低大页和小页之间的转换成本，我们提出了一种自适应转换方案，以根据内存空间利用率来提高SmartMD的性能。 这个想法是：如果系统有足够的可用内存空间，我们只使用大页来提高系统性能；如果内存空间不足，我们则将大页分解为小页，以提高重删效率。

在每个监测周期中，我们首先检查内存利用率，然后调整参数 Threscold，使其能够动态地识别要拆分的页面。特别是，如果内存利用率低于 memlow，我们将 Threscold 减一，使更多的页面保持在温暖或热状态，并防止它们因高内存访问性能而被拆分。如果内存利用率高于 memhigh，表示内存需求很高，我们将 Threscold 增加一，以允许更多大页面被视为冷页面，并有资格被拆分，从而实现更高的去重率。