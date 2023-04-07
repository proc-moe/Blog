---
title: course-os-nju-1-read 什么是操作系统
date: 2022-11-27 07:50:31
tags:
---
# Chapter 1 A Dialogue on the Book
- 什么是冯诺伊曼机？

二进制逻辑、程序存储执行以及计算机由五个部分组成（运算器、控制器、存储器、输入设备、输出设备）

- What does OS do?

- Main method to realize OS

- bookname: three easy pieces what?

Virtualization, Concurrency, and Consistency

- brief introduction to Thread

You can think of a thread as a function running within the same memory space as other functions, with more than one of them active at a time

- classical code

x++ for 1000 times in 2 thread

result is not 2000

- What is persistence

Difference between ram & rom

# Chapter 2 Intro
## Goal
Virtualize physical resources and store files persistently.

- how?
Protection, energy-effiency, high performance, abstractions.(c - assembly - logic gates - transistors)

# 生词 
the crux of the problem

depicts our first program

It turns out that the operating system, with some help from the hardware, is in charge of this illusion

the OS is juggling many things at once, first running one process, then an other,and so forth.

Indeed, modern multi-threaded programs exhibit the same problems