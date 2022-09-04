---
title: "算法temp"
author: Lin Han
date: "2022-01-03 02:07"
math: true
published: false
categories:
  - Note
  - Algorithm
tags:
  - Algorithm
---

五个要素

- 输入：输入有一定限制，算法要能处理所有符合限制的输入
- 输出
- 有穷性：在有限步内停止
- 确定性：明确定义每一步操作
- 可行性：每一步都能在一定时间内完成

一个问题有多种算法

- 不同的时间空间权衡
- 针对不同类型的输入
  - 逆序，完全排序，范围很小，重复值很多，...
- 实现复杂度

分析

- 正確定
- 复杂度

https://www.khanacademy.org/computing/computer-science/algorithms/asymptotic-notation/a/asymptotic-notation

![log calculations](/assets/img/post/Algorithm/log-calculations.png)

# 时间复杂度

渐进时间复杂度

- 算法运行时间随着输入规模变化怎么变化
  - 复杂度是输入规模的函数
- 只看最高次项
  - $5 n^{3}+100 n^{2}+10 n+50 \sim n^{3}$

输入规模要真的能代表输入规模，比如大数乘法用总位数代表输入规模

![for runtime](/assets/img/post/Algorithm/for-runtime.png)

for 的内容执行了 N 次，但是比较执行了 N+1 次

做题基本渐进 $10^{10}$ 到 $10^{12}$ 1s

![running time](/assets/img/post/Algorithm/running-time.png)

## 计法

分类

- 最好：不会低于这个时间 $\Omega$，$\omega$
- 最坏：不会高于这个时间 O，o
- 平均：平均是这个时间 $\Theta$，$\theta$
  - 小心定义平均
    - 随机输入
    - 真实场景下的输入：随机输入加权平均

记法

- 大写：存在 c，n 使得大于等于/小于等于
  - $T(n)=O(f(n))$：存在$c>0，n_0\ge 0$使得对所有$n \ge n_0$，$cf(n)\ge T(n)$
    ![O](/assets/img/post/Algorithm/o.png)
  - $T(n)=\Omega(f(n))$：存在$c>0，n_0\ge 0$使得对所有$n \ge n_0$，$cf(n)\le T(n)$
    ![big omega](/assets/img/post/Algorithm/big-omega.png)
  - $T(n)=\Theta(f(n))$：存在$c_1>0，c_2>0，n_0\ge 0$使得对所有$n \ge n_0$，$c_1f(n)\le T(n) \le c_2f(n)$
    ![big theta](/assets/img/post/Algorithm/big-theta.png)
- 小写：对所有 c，存在 n，使得大于/小于（没有等于）
  - $T(n)=o(f(n))$：对所有的$c>0$，存在$n_0\ge 0$使得对所有$n\ge n_0，cf(n)>T(n)$
  - $T(n)=\omega(f(n))$：对所有的$c>0$，存在$n_0\ge 0$使得对所有$n\ge n_0，cf(n)<T(n)$
  <!-- - $T(n)=\theta(f(n))$：对所有的$c_1>0，c_2>0$，存在$n_0\ge 0$使得对所有$n\ge n_0，c_1f(n)<T(n)<c_2f(n)$ -->

![notations](/assets/img/post/Algorithm/notations.png)

性质

- $f(n)=\Theta(g(n))\Leftrightarrow f(n)=O(n) \quad and \quad f(n)=\Omega(n)$
- 传递性：$f(n)=\Theta(g(n)) \quad and \quad g(n)=\Theta(h(n)) \Rightarrow f(n)=\Theta(h(n))$
- 自反性：$f(n)=\Theta(f(n))$
- 对称性：$f(n)=\Theta(g(n))\Leftrightarrow g(n)=\Theta(f(n))$
- 转制对称性：$f(n)=O(g(n))\Leftrightarrow g(n)=\Omega(f(n))$
