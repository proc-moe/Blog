---
title: 教你在C语言中写汇编
date: 2022-12-26 08:06:42
tags:
---

http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html

# gcc inline assembly
## 1. syntax （句法规则）
gcc中的汇编采用  `AT&T` 结构， 与常用的 `Intel` 不同， 区别在于以下五点。

### 1. src dest 位置不同
`AT&T` src -> dest

`Intel` dest := src

```
"Op-code dst src" in Intel syntax changes to

"Op-code src dst" in AT&T syntax.
```

### 2. 寄存器命名不同
AT&T 中， 寄存器以 % 开头。

Intel 中， 寄存器是裸的
```
+------------------------------+------------------------------------+
|       Intel Code             |      AT&T Code                     |
+------------------------------+------------------------------------+
| mov     eax,1                |  movl    $1,%eax                   |   
+------------------------------+------------------------------------+
```

### 3. 立即数命名不同

AT&T 中， 立即数以 $ 开头

Intel 中， 立即数是裸的
```
+------------------------------+------------------------------------+
|       Intel Code             |      AT&T Code                     |
+------------------------------+------------------------------------+
| mov     eax,1                |  movl    $1,%eax                   |   
+------------------------------+------------------------------------+
```


### 4. 操作数(operand)大小的命名不同
### 4. Operand Size

在 AT&T 中， 操作数大小决定于`操作命令`的最后一段，比如说"b""w""l",分别代表byte(8b),word(16b),long(32b)。

比如说 
"movb $1,%eax"

b被添加在mov后面，意思是mov


Intel 在 `内存操作数` 前面（不是操作命令）带上 ’byte ptr’, ’word ptr’, and ’dword ptr’.

例子：
"mov al, byte ptr foo" --- at&t ---> "movb foo, %al" foo这里是段内存

### 5. 内存操作数不同

```
In Intel syntax the base register is enclosed in ’[’ and ’]’ where as in AT&T they change to ’(’ and ’)’. Additionally, in Intel syntax an indirect memory reference is like

section:[base + index*scale + disp], which changes to

section:disp(base, index, scale) in AT&T.
```

### 6. 总结
```
mov eax, [ecx+3]  -------> mov 3(%ecx),%eax
mov eax, [ebx+20h] ------> mov 0x20(%ebx),%eax
mov eax, [ebx+ecx*2h] ---> mov (%ebx,%ecx,0x2),%eax
mov eax, [ebx+ecx*2h-20h] -> mov -0x20(%ebx,%ecx,0x20),%eax
```

## 2. Basic Inline (C代码中的汇编)
### 基础形式
```
asm("assembly code")

__asm__ ("assembly code")
```
为什么多出来一个带下划线的呢？ 因为如果 asm 是关键词， 就只能使用 `__asm__` 了。


### 多行情况
通过在最后加上\n\t解决，
```
 __asm__ ("movl %eax, %ebx\n\t"
          "movl $56, %esi\n\t"
          "movl %ecx, $label(%edx,%ebx,$4)\n\t"
          "movb %ah, (%ebx)");
```


因为 gcc 将每个指令发送到 as(GAS)。 通过添加newline\tab， 我们可以正确的把格式化好的行发送给汇编器。

## 3. Extended Asm (延展Asm)
如果在我们的代码中，我们改变了一些寄存器，然后从汇编代码中返回，返回前并没有回复那些寄存器的状态。就会发生不好的事情了。

这是因为gcc并不知道寄存器中的内容被改变了，特别是编译器想要做一些优化的时候。它将假设某个寄存器包含某个变量的值，而我们可能在没有通知GCC的情况下改变了这个值，然后它就像什么都没发生过一样继续工作。

我们可以做这两件事情。要不使用没有副作用的指令，要不在退出asm时恢复寄存器的状态。 这就是为什么需要 extended asm，它可以帮助我们。



在基本的内联汇编中，我们只有指令(instructions)。在扩展汇编中，我们也可以指定操作数。它允许我们指定输入寄存器、输出寄存器和一个clobbered寄存器的列表。

我们不一定要指定要使用的寄存器，我们可以把这个问题留给GCC来解决，这样可能更适合GCC的优化方案。总之，基本的格式是
```
    asm ( assembler template 
           : output operands                  /* optional */
           : input operands                   /* optional */
           : list of clobbered registers      /* optional */
           );
```
### 3.1 Assembler template
汇编器模板有些特殊格式：

```
  int a=10, b;
        asm ("movl %1, %%eax; 
              movl %%eax, %0;"
             :"=r"(b)        /* output */
             :"r"(a)         /* input */
             :"%eax"         /* clobbered register */
             );       
```
b 是输出操作数， 在模板中被%0替代

"r" 是对操作数的一个约束条件。我们将在后面详细介绍约束。

寄存器前有两个%%。 操作数前有一个%

第三个冒号后面的`clobbered register` %eax告诉GCC，%eax的值将在 asm 里面被修改，所以GCC不会用这个寄存器来存储任何其他的值。

```
tips:
mov 与 lea
https://blog.csdn.net/farmwang/article/details/75103220

mov %eax, %ebx 将 eax里的值给ebx
例如在32位环境下, 有内存位置标签l1, 则下面两行效果相同:

movl $l1, %edi
leal l1, %edi
 
lea传送的是寄存器里面的值，mov传送的是主存中以寄存器的值为地址里面的值。
例如。
leal 18（%eax），%ebx
movl 18(%eax) ,%ebx
这样，两条指令，传送的值是不一样的
leal 是传送 18+%eal（值）到 寄存器%ebx
movl 是传送的是在主存中以18+%eax为地址的存储序列里面存的值到%ebx。
```

### 3.2 操作数

命名顺序： output -> input 从 0 至 n-1

分割方式： 如果有多个操作数，需要被，分割开

引用方式： 通过 %数字 引用。 

左值限定： output 必须是lvalues， input可以是表达式

其他： 如果output expression不能被直接取指， 则必须允许存在一个寄存器作为中介。

如上所述，普通的输出操作数必须是只写的；GCC将假定这些操作数在指令前的值是死的，不需要生成。扩展asm也支持输入-输出或读写操作数。

#### 例子a： five_times_x = x * 5
```
asm ("leal (%1,%1,4), %0"
             : "=r" (five_times_x)
             : "r" (x) 
             );
```

这里我们的输入是在'x'中。我们没有指定要使用的寄存器。GCC会选择一些寄存器作为输入，一个作为输出，然后做我们想要的事情。如果我们想让输入和输出驻留在同一个寄存器中，我们可以指示GCC这样做。这里我们使用这些类型的读写操作数。通过指定适当的约束条件实现

---
Break:
#### **Quiz**: 输入输出是什么？
例子a被编译后是这样的：
```
    1141:       c7 45 f8 14 45 11 00    movl   $0x114514,-0x8(%rbp)
    1148:       8b 45 f8                mov    -0x8(%rbp),%eax
    114b:       67 8d 04 80             lea    (%eax,%eax,4),%eax
    114f:       89 45 fc                mov    %eax,-0x4(%rbp)
    1152:       8b 45 fc                mov    -0x4(%rbp),%eax
```
写代码时，我们直接对内存进行输入输出，但是在汇编后。是这样的情景：
1. 将 4 byte 的 0x114514 送入堆栈 -0x8(%rbp) 中

2. 进行 lea 操作： lea (%eax,%eax,4), %eax. 即 eax * 4 + eax -> eax

3. 将 eax 送入 -0x4(%rbp) 中。

汇编器通过eax中转了

---

#### 例子b: 送到同一个寄存器里
``` c
    asm ("leal (%0,%0,4), %0"
             : "=r" (five_times_x)
             : "0" (x) 
             );
```
输入变成了”0“，这是啥意思呢？

哦哦，这下输入和输出都放在一个寄存器里了。

***是不是这个指南太老了???*** 因为看起来不像。。。

```
    1141:       c7 45 f8 14 45 11 00    movl   $0x114514,-0x8(%rbp)
    1148:       8b 45 f8                mov    -0x8(%rbp),%eax
    114b:       67 8d 04 80             lea    (%eax,%eax,4),%eax
    114f:       89 45 fc                mov    %eax,-0x4(%rbp)
    1152:       8b 45 fc                mov    -0x4(%rbp),%eax
```

#### 例子c: 如果我们想在特定的寄存器里计算呢？

``` c
        asm ("leal (%%ecx,%%ecx,4), %%ecx"
             : "=c" (x)
             : "c" (x) 
             );
```

```
    1141:       c7 45 f8 14 45 11 00    movl   $0x114514,-0x8(%rbp)
    1148:       8b 45 f8                mov    -0x8(%rbp),%eax
    114b:       89 c1                   mov    %eax,%ecx
    114d:       67 8d 0c 89             lea    (%ecx,%ecx,4),%ecx
    1151:       89 c8                   mov    %ecx,%eax
    1153:       89 45 f8                mov    %eax,-0x8(%rbp)
```

好吧，看起来eax，被送到ecx作为输入计算，再作为output送出来。


#### 为什么不送进clobber list?
在前两个例子中，GCC决定了寄存器，gcc知道发生什么变化。

在最后一个例子中，GCC知道ecx进入了x。因此，由于它可以知道ecx的值，所以它不被认为是clobbered。

#### How to write constraints?

> TODO

https://stackoverflow.com/questions/3898704/gcc-inline-assembly-constraints

### 3.3 clobber list

写在第三个冒号后面的寄存器。作用是通知gcc，这寄存器归我了你别动。

input,output(前两个冒号后面的) 声明的寄存器就不需要填写了， 如果用到了其他的寄存器，则需要写。

如果通过不可预测的方式修改了内存，需要将”memory" 添加倒 clobbered registers中， 这将让gcc不保留汇编指令中缓存在reg中的内存，如果受影响的内存没有在input/output中列出，则需要添加volatile关键字 

形如 ： asm volatile ()

### 3.4 volatile?

volatile 是什么？

如果我们的汇编语句必须在我们放置的地方执行，那么就可以用 asm volatile ()，如果没有副作用只是做点计算的话， 那就不需要了用volatile了，交给gcc就好。

## 4. Contraints
### 4.1 常使用的constraints
#### 1.Register operand constraint (r)
```
asm ("movl %%eax, %0\n" :"=r"(myval));
```
这个语句中， eax 先被复制倒 reg 中去， 然后从这个reg中更新到内存中，
指定内存的话，可以将r改写成如下内容：
```
+---+--------------------+
| r |    Register(s)     |
+---+--------------------+
| a |   %eax, %ax, %al   |
| b |   %ebx, %bx, %bl   |
| c |   %ecx, %cx, %cl   |
| d |   %edx, %dx, %dl   |
| S |   %esi, %si        |
| D |   %edi, %di        |
+---+--------------------+
```

#### 2.Memory operand constraint(m)
When the operands are in the memory, （<-这句话不太理解，应该是 **使用的constraint是"m"的话**），任何操作将会直接在内存中反映出来。和r不太一样，r首先将值存储在一个要修改的寄存器中，然后写回到内存。

但是register constraints一般只在进程需要性能优化时使用。memory constraint 在 c 变量需要在asm被处理，但是你又不想让寄存器保存这个值。

#### 3. Matching constraints
```
asm ("incl %0" :"=a"(var):"0"(var));
```
这里的0 指明了使用与第0个相同的限制。


举个例子：
```
        asm ("leal (%0,%0,4), %0"
             : "=r" (x)
             : "0" (x) 
             );
```
```
    1148:       8b 45 f8                mov    -0x8(%rbp),%eax
    114b:       67 8d 04 80             lea    (%eax,%eax,4),%eax
    114f:       89 45 f8                mov    %eax,-0x8(%rbp)
    1152:       8b 45 fc                mov    -0x4(%rbp),%eax
    1155:       89 c6                   mov    %eax,%esi
```

```
        asm ("leal (%0,%0,4), %0"
             : "=r" (x)
             : "0" (x) 
             );
```

```
    1148:       8b 45 f8                mov    -0x8(%rbp),%eax
    114b:       89 c1                   mov    %eax,%ecx
    114d:       67 8d 0c 89             lea    (%ecx,%ecx,4),%ecx
    1151:       89 c8                   mov    %ecx,%eax
    1153:       89 45 f8                mov    %eax,-0x8(%rbp)
```

