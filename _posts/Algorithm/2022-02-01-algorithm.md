---
title: "算法笔记"
author: Lin Han
date: "2022-01-03 02:07"
published: true
math: true
categories:
  - Note
  - Algorithm
tags:
  - Algorithm
---

时间复杂度
https://www.khanacademy.org/computing/computer-science/algorithms/asymptotic-notation/a/asymptotic-notation

![log calculations](/assets/img/post/Algorithm/log-calculations.png)

五个要素
- 输入
- 输出
- 有穷性：算法在有限步内终止
- 确定性：明确定义每一步操作
- 可行性：每一步都能在一定时间内完成


![for runtime](/assets/img/post/Algorithm/for-runtime.png)

![running time](/assets/img/post/Algorithm/running-time.png)


# 渐近运行时间
- 运行时间随着输入规模增长怎么增长
- 只看最高次项

计法


- 大写：存在c，n使得大于等于/小于等于
- 小写：对所有c，存在n，使得大于/小于（没有等于）

- 大O

![O](/assets/img/post/Algorithm/o.png)

- 大$\Omega$

![big omega](/assets/img/post/Algorithm/big-omega.png)

- 大$\Theta$

![big theta](/assets/img/post/Algorithm/big-theta.png)

- 小o
- 小$\omega$

![notations](/assets/img/post/Algorithm/notations.png)

性质
- $f(n)=\Theta(g(n))\Leftrightarrow f(n)=O(n) \quad and \quad f(n)=\Theta(n)$
- 传递性：$f(n)=\Theta(g(n)) \quad and \quad g(n)=\Theta(h(n)) \Rightarrow f(n)=\Theta(h(n))$
- 自反性：$f(n)=\Theta(f(n))$
- 对称性：$f(n)=\Theta(g(n))\Leftrightarrow g(n)=\Theta(f(n))$
- 转制对称性：$f(n)=O(g(n))\Leftrightarrow g(n)=\Omega(f(n))$
