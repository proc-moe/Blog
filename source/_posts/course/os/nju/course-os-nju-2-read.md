---
title: course-os-nju-2-read 多线程初步
date: 2022-12-02 02:18:26
tags: 
---
# Chapter 20 Concurrency: An Introduction
- What is a thread
    - how do thread switch in a core
    - data storage
    - 
- Difference between thread & process

- Use of threads

# Chapter 27 Interlude: Thread API
## Thread Creation
``` c
#include <pthread.h>
int
pthread_create(pthread_t *thread,
const pthread_attr_t *attr,
void *(*start_routine)(void*),
void *arg);
```
- pthread_t thread 


# 生词
Fortunately, this is usually OK, as stacks do not generally have to be very large (the exception being in programs that **make heavy use** of recursion).

Instead of waiting, your program may wish to do something else, including **utilizing** the CPU to perform computation, or even issuing further I/O requests.
# Questions
- 线程如何存储本地数据。能不能有线程自己的堆？

应该不行，一般都是上锁。
``` c
__thread int a;
```

- cache miss 计数器？

https://stackoverflow.com/questions/10082517/simplest-tool-to-measure-c-program-cache-hit-miss-and-cpu-time-in-linux

- Procedure when blocked by IO



- IO block 是什么？
IO 锁呗 （被打死