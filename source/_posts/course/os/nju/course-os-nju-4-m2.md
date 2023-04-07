---
title: course-os-nju-m2
date: 2022-12-20 23:37:43
tags:
---
https://jyywiki.cn/OS/2022/labs/M2

*assembly alert*
# 如今的栈帧

``` asm
mov $0x114,%esi
mov $0x514,%edi
call 1139 <add> # rip下个指令 入栈

000001139 <add>:
push %rbp # 现在的rbp 入栈
mov %rsp,%rbp # 新栈帧的 rbp
sub $0x10, $rsp # 计算当前rsp

// 代码

pop %rbp # rbp 归位
ret # equal (pop %rip) 
```
函数调用时有这样的一些变化：


# 什么是协程

在这个实验中，我们实现轻量级的用户态协程 (coroutine，“协同程序”)，也称为 green threads、user-level threads，可以在一个不支持线程的操作系统上实现共享内存多任务并发。即我们希望实现 C 语言的 “函数”，它能够：

被 start() 调用，从头开始运行；
在运行到中途时，调用 yield() 被 “切换” 出去；
稍后有其他协程调用 yield() 后，选择一个先前被切换的协程继续执行。
协程这一概念最早出现在 Melvin Conway 1963 年的论文 "Design of a separable transition-diagram compiler"，它实现了 “可以暂停和恢复执行” 的函数。

# tips
1. 协程除非执行 co_yield() 主动切换到另一个协程运行，当前的代码就会一直执行下去。

2. co_wait(co) 表示当前协程需要等待，直到 co 协程的执行完成才能继续执行

- Quiz:
由 1 知道 co_wait() 等待后不会主动切换协程， 由 2 知 协程会死等下去。是否会 
    - co_wait(co) 自带自旋锁和 co_yield(co)? 


答案很简单，读完` 2.2. 协程的使用` 的示例代码后就知道了
``` c
#include <stdio.h>
#include "co.h"

int count = 1; // 协程之间共享

void entry(void *arg) {
  for (int i = 0; i < 5; i++) {
    printf("%s[%d] ", (const char *)arg, count++);
    co_yield();
  }
}

int main() {
  struct co *co1 = co_start("co1", entry, "a");
  struct co *co2 = co_start("co2", entry, "b");
  co_wait(co1);
  co_wait(co2);
  printf("Done\n");
}
```

t1: co_yield() -> t2: co_wait(co1)中

此时应当重新co_yield()切换协程，直到co1完成

# 动手做一做
## 实验指南
- 保存实验状态，恢复现场
co_start时保存父协程状态。
co_yield时保存当前协程状态。

## asm
``` c
asm ( assembler template 
           : output operands                  /* optional */
           : input operands                   /* optional */
           : list of clobbered registers      /* optional */
           );
```
实际上来说c99后面需要 \
``` c
    printf("start reg asm test");
       int a=10, b;
        asm ("movl %1, %%eax \
              movl %%eax, %0;"
             :"=r"(b)        /* output */
             :"r"(a)         /* input */
             :"%eax"         /* clobbered register */
        );       
```

## 协助资料
