---
title: 'Codeforces Round 934 (Div. 2) C: MEX Game 1'
date: 2024-03-27 18:40:06
tags:
categories: algo
---
**博弈**

二人竞争的这种。。。 

> Alice can adapt to Bob's strategy. Try to keep that in mind。

一开始想错了，以后要多复习。

我一开始想的是 Bob 努力删掉某一个数字，从而使 Alice 的 MEX 限制在某个区间内。二分查找这个 MEX。但是战场是动态的， Bob 删去 `x`, Alice 可以立刻提取 `x`。 这我万万没想到。

