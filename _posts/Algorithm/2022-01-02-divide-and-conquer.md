---
title: "分治"
author: Lin Han
date: "2022-01-02 20:21"
published: false
categories:
  -
tags:
  -
---

分：将原问题划分成多个不重合的，跟原问题类似的更小的子问题
治：问题足够小时解决
合：将结果合并

## 主定理
![master therom](/assets/img/post/Algorithm/master-therom.png)
- b代表分的时候每个子问题规模是原问题规模的1/b
- a代表划分之后需要解决几个这样的子问题
- d代表合并一层子问题的代价

a=b的话，只要合并复杂度小于平方，速度就会变快
