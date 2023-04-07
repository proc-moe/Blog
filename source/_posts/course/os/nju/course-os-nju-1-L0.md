---
title: course_os_nju_1_L0 实验环境准备
date: 2022-09-10 00:38:36
tags:
categories: os_course
---
# Links
https://zhuanlan.zhihu.com/p/499141891

https://simplestupidcode.github.io/post/2022062501.html

# Environment
## OS
Ubuntu 20.01
- download from 
https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
- run in `vmware`
## Packages
Inspired by https://jyywiki.cn/OS/2022/labs/Labs
```
Ubuntu 20.04 容器 (Docker)。容器中仅有最小的必要系统工具。使用以下 Dockerfile 配置与在线评测一致的环境；我们开放了容器的 SYS_PTRACE 权限；

FROM ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update
RUN apt-get install -y build-essential gcc-multilib qemu-system strace gdb sudo python3 libsdl2-dev libreadline-dev llvm-11 gcc-riscv64-linux-gnu
RUN useradd -ms /bin/bash user
USER user
WORKDIR /home/user
```
- [IMPORTANT] packages were installed seperatedly as `sudo apt-get install -y build-essential` and `sudo apt-get install -y gcc-multilib` and etc

notes: exec the instruction below
```
$ apt-get install -y build-essential gcc-multilib qemu-system strace gdb sudo python3 libsdl2-dev libreadline-dev llvm-11 gcc-riscv64-linux-gnu
```
may returns
```
The following packages have unmet dependencies:
 gcc-riscv64-linux-gnu : Depends: gcc-9-riscv64-linux-gnu (>= 9.3.0-3~)
E: Unable to correct problems, you have held broken packages.
```

## Lab
```
$ git clone https://github.com/NJU-ProjectN/os-workbench-2022
$ git pull origin L0
```

# How to RUN
cd am-games/build
```
$ qemu-system-x86_64 -drive format=raw,file=amgame-x86_64-qemu
```

# NOTES
## What is QEMU?
A virutal machine.
## QEMU  QUICKSTART

### Installation
pacman -S qemu

```
qemu-img create -f qcow2 Image.img 10G
```
Then we have a image.img

```
qemu-system-x86_64 -enable-kvm -cdrom Manjaro.iso -boot menu=on -drive file=Image.png -m 2G
``` 
we use a x86 architecture.

-enable-kvm make it all run better,better vm virtualzation

-cdrom load this to cdrom 

-menu=on select from the menu you build from

-drive select a virtual hard drive
## How to Start QEMU
https://www.youtube.com/watch?v=AAfFewePE7c

https://drewdevault.com/2018/09/10/Getting-started-with-qemu.html


-nic user,model=virtio: Adds a virtual network interface controller, using a virtual LAN emulated by qemu. This is the most straightforward way to get internet in a guest, but there are other options (for example, you will probably want to use -nic tap if you want the guest to do networking directly on the host NIC). model=virtio specifies a special virtio NIC model, which is used by the virtio kernel module in the guest to provide faster networking.
