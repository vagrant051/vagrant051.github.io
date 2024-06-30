---
layout: post
title: "STL iterator"
categories: C++
tags: [STL]
---

STL迭代器概念

## 简介

***迭代器是一个“可以遍历STL容器全部或部分元素”的对象***，迭代器用来表现容器中的某一元素位置

## 迭代器的通用功能

操作符重载 | 说明
------ | ------ 
Operator* | 解引用运算符：返回当前位置上的元素值
Operator++ | 令迭代器前进至下一个元素
Operator==/!= | 判断两个迭代器是否指向同一个位置
Operator= | 对迭代器赋值 

## 分类

操作符重载 | 说明
------ | ------ 
输入迭代器 （input iterator） | 只读
输出迭代器 （output iterator） | 只写
前向迭代器 （forward iterator） | 在迭代器所形成的区间上进行读写操作
双向迭代器 （bidirectional iterator） | 可双向移动（如：list,set,map）
随机访问迭代器（ random-access iterator） | 前三种支持operator++，第四种再加上operator--，第五种涵盖所有指针能力（如vector,deque,array,string）

![My helpful screenshot](/assets/iterator/1.png)
