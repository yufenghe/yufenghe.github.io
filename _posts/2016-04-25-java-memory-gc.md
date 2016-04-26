---
layout: postlayout
title: 浅谈Java内存模型
description: 最近在看关于Java内存模型的内容，整理了一些关于Java垃圾回收器的内容
thumbimg: 1346208288725.jpg
categories: [java]
tags: [java,gc]
---

java作为面向对象的高级语言，底层由c++实现，与c/c++最大的不同在于垃圾的回收由虚拟机（jvm）实现，使得人们专心于编程，无需考虑垃圾回收问题。虽然一般不用管理垃圾回收，但是依然会发生内存溢出问题：

# JVM结构

JVM主要分为：

**堆heap

**方法区method area

**栈stack

**寄存器register pc

**本地方法区native method area

其中堆与方法区是共享区域，栈是以栈帧形式存在，进行的是压栈和出栈操作

## 堆和栈

堆是对象存储区域，是全局共享的，栈是程序运行区域，即一个存储区域，一个逻辑区域，一个线程就有一个线程栈与之对应。

堆与栈分离的好处：
>

# 常见的内存溢出
