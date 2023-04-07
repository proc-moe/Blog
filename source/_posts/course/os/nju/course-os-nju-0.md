---
title: course_os_nju_0_gdb 如何配置gdb
date: 2022-09-08 01:58:42
tags: 
categories: os_course
---
# steps
- b main: set a breakpoint
- s: go to next line
- si: into next line

# layouts
- layout split
- layout asm
- layout regs

# mems
- x/nfu addr: print mem
    - n: how many units to print
    - f: Format character
        - x,d,u,o,t,a,c,f,s,i,m
    - u: unit
        - b: byte
        - h: half-word
        - w: word
        - g: giant-word
        - suggestion: use w(32bit) to catch up with int
    - addr: 0x54320

- i r reg: print register 