---
layout: "post"
author: Lin Han
title: "图像处理笔记"
date: "2022-02-13 01:04"
math: true
categories:
  - Note
  - "Video Image Processing"
tags:
  - Note
  - "Video Image Processing"
---

![color](/assets/img/post/Note/color.png)

![eye camera](/assets/img/post/Note/eye-camera.png)

- Rod：视杆，亮度，120M
- Cone：视椎，颜色，6M

[//]: # (TODO:怎么判断三元色选的好不好)
# 颜色表示
- 三元色：Trichromatic color mixing
  - RGB
    - 波长：R>G>B，紫外线波长小，能量大，杀菌
  - CMY
    - cyan：青
    - magenta：品红
    - yellow
  - XYZ
    ![xyz color](/assets/img/post/Note/xyz-color.png)
    - Y：亮度
    - XZ：颜色

- 亮度+颜色
  - Luminance 亮度
  - Chrominance 色度
    - Hue：色调，色轮上的角度 $\theta$，颜色的主导波长，渐变的
    - Saturation：饱和度，有多纯，掺了多少白。越纯越饱和，白色最不饱和
  ![color-lhs](/assets/img/post/Note/color-lhs.png)
  - CIELAB
  ![cie rbg](/assets/img/post/Note/cie-rbg.png)
    - 一些颜色需要负的红
  - HSI(HSB)
  ![convert rgb hsi](/assets/img/post/Note/convert-rgb-hsi.png)
  - YIQ：模拟，NTSC(National Television System
Committee)美国标准
    - Y亮度
    - IQ颜色
    ![yiq rgb](/assets/img/post/Note/yiq-rgb.png)
    - 模拟彩电颜色标准，黑白电视只放YIQ里的Y
  - YUV：模拟，PAL欧洲标准
  - YCbCr：数字版YUV
    - 很多压缩图像用这个格式
    - RBG->Y：彩色变黑白
  ![ycvcr rgb](/assets/img/post/Note/ycvcr-rgb.png)

颜色范围
- SDR：Standard Dynamic Range，8 bit
- HDR：High dynamic range，16 bit

- Illuminating source
  - 发光频率决定颜色
  - R+G+B=White
  - 一般用RGB
- Reflecting source
  - 颜色=照射频率-吸收频率
  - R+B+G=Black
  - 一般用CMY
    - CMYK：Cyan(青), Magenta(洋红), Yellow(黄), Black(黑)

传感器上RGB交替

![color sensor](/assets/img/post/Note/color-sensor.png)

![color capture process](/assets/img/post/Note/color-capture-process.png)
4：demosaic

颜色校准：白色的RGB要相等

Gamma Correction：显示的强度和真实的强度是非线性的

[//]: # (TODO:)

视频
- Standard Definition：720x480，4：2，25-30fps，隔行或逐行扫描，8 bit
- High Definition，1080p，2K：1920x1080，16：9/2：1，最高60fps
- Ultra High Definition，4K：3840x2160，16：9，最高120fps，16 bit，发行10～12bit，色域更广


# 对比度
低对比度的图像看起来颜色非常贴近，理想图像颜色在直方图里分布宽且均匀

![contrast](/assets/img/post/Note/contrast.png)

直方图只能总结图像强度分布，很不一样的图片也可以有一样的直方图

![histogram](/assets/img/post/Note/histogram.png)

## 增强对比度
逐个像素改变强度值，函数非递减
- 固定函数
  - 线性拉伸
  ![linear stretching](/assets/img/post/Note/linear-stretching.png)
    - 分段线性
    ![piece wise linear](/assets/img/post/Note/piece-wise-linear.png)
  - 非线性拉伸
    - Log变换：往上弯，拉伸暗区域
      - g = blog(af+1)
      ![log transform](/assets/img/post/Note/log-transform.png)
    - 指数变换：往下弯，拉伸亮区域
      - $g = b(e^{af}-1)$
    - 指数变换，Power law
      - 只能拉伸一头，集中中间的处理不了
      ![power low](/assets/img/post/Note/power-low.png)
      ![power law example](/assets/img/post/Note/power-law-example.png)

[//]: # (TODO:图片power law)
- 自适应变换：根据原图和目标的直方图确定函数
  - 直方图增强
  - 原像素变成累积概率分布*最大强度，总能得到一个比较平的histogram
![cpdf](/assets/img/post/Note/cpdf.png)

![flat](/assets/img/post/Note/flat.png)

局部增强：图像整体强度分布平均，但是局部不平均，可以滑动窗口针对局部增强对比度

在一些不重叠的区域里计算变换函数
- 区域里的所有像素用这个函数
![block histogram](/assets/img/post/Note/block-histogram.png)
  - 区域边缘会有跳变
- 其他的位置向区域中心点距离加权平均
![adaptive histogram](/assets/img/post/Note/adaptive-histogram.png)


# 傅立叶变换

f(m,n)

可分：函数可以表示为两个函数积 $f(m,n)=f_v(m)f_h(n)$，2D矩阵可以表示为两个1D矩阵积(没一行都成比例，每一列都成比例)

函数内积，投影：一个函数乘另一个函数的共扼在整个定义域上积分

$\phi(x,idx)$ 第idx个基底。

orthonormal：单位，垂直
$$
\int_{-\infty}^\infty \phi(x, u_1)\phi^*(x,u_2)dx=\{
\begin{matrix}
1, \quad u_1=u_2 \\
0, \quad u_1\ne u_2
\end{matrix}
$$

正变换
![forward transform](/assets/img/post/Note/forward-transform.png)

逆变换
![inverse transform](/assets/img/post/Note/inverse-transform.png)

![forier](/assets/img/post/Note/forier.png)

用的是 $2\pi u$ 而不是 $\omega$ ，所以没有 $\frac{1}{2\pi}$

![fourier pair](/assets/img/post/Note/fourier-pair.png)

时域宽，低频信号多，频域窄。时域越宽越接近常数的频率0，频域越接近冲击。频域零点在1/时域方波长度(连续是1/2时域零点，离散是1/N，离散从0开始)
![sinc](/assets/img/post/Note/sinc.png)

![fourier properties](/assets/img/post/Note/fourier-properties.png)
时域卷积，频域相乘

时域是实函数，频域性质

![real ft](/assets/img/post/Note/real-ft.png)

2D傅立叶

![2d ft](/assets/img/post/Note/2d-ft.png)

1D单位正交基底相乘得到的2D基底还是单位正交的

![2d frequendy](/assets/img/post/Note/2d-frequendy.png)
2D信号频率
- 两个垂直方向的频率：比如x和y方向上单位长度(整幅图)分别 $f_x, f_y$ 周期
- 频率和角度：角度是频率最高的方向 $atan(\frac{y}{x})$，这个方向上的频率 $f_m$ 是 $\sqrt{f_x^2+f_y^2}$

$F\{f(x)\}=\frac{1}{2j}(\delta(u-f_x,v-f_y)-\delta(u+f_x, v+f_y))$
![delta 2d](/assets/img/post/Note/delta-2d.png)

![2d ft prop](/assets/img/post/Note/2d-ft-prop.png)

![2d ft prop 1](/assets/img/post/Note/2d-ft-prop-1.png)

可分2D傅立叶：如果f(x, y)可以表示为g(x)*h(y)的形式，用g(x)傅立叶得F(u)，用h(y)傅立叶得F(v)，F(u, v)=F(u)乘F(v)

![2d seperate ft](/assets/img/post/Note/2d-seperate-ft.png)

![2d seperate ft 2](/assets/img/post/Note/2d-seperate-ft-2.png)

2D傅立叶旋转

![2d ft rotate](/assets/img/post/Note/2d-ft-rotate.png)
