---
layout: "post"
author: Lin Han
title: "video image processing"
date: "2022-02-13 01:04"
math: true
categories:
  -
tags:
  -
public: false
---

![color](/assets/img/post/Note/color.png)

- Rod：亮度，120M
- Cone：颜色，6M


# 颜色表示
- 三元色
  - RGB
  - CMY
  - XYZ
- 亮度+颜色
  - Luminance
  - Chrominance
    - Hue：色调，色轮上的角度$\theta$，颜色的波长，渐变的
    - Saturation：饱和度，有多纯，有多靠近边缘，最饱和最纯，白色最不饱和
  ![color-lhs](/assets/img/post/Note/color-lhs.png)
  - CIELAB
  - HSI
  ![convert rgb hsi](/assets/img/post/Note/convert-rgb-hsi.png)
  - YIQ：NTSC美国标准
    - Y亮度
    - IQ颜色
    ![yiq rgb](/assets/img/post/Note/yiq-rgb.png)
    - 模拟彩电颜色标准，黑白电视只放YIQ里的Y
  - YUV：PAL欧洲标准
  - YCbCr
    - 很多压缩图像用这个格式
  ![ycvcr rgb](/assets/img/post/Note/ycvcr-rgb.png)





- Illuminating source
  - 颜色取决于发光频率
  - R+G+B=White
  - 一般用RGB
- Reflecting source
  - 颜色=照射频率-吸收频率
  - R+B+G=Black
  - 一般用CMY
    - CMYK：Cyan(青), Magenta(洋红), Yellow(黄), Black(黑)


Gamma Correction

视频
- Standard Definition：720x480，4：2，25-30fps，隔行扫描
- High Definition，1080p，2K：1920x1080，16：9，最高60fps
- Ultra High Definition，4K：3840x2160，16：9，最高120fps，10～12位颜色，色域更广


# 对比度
低对比度的图像看起来颜色非常贴近，理想图像颜色在直方图里应该分布宽而且均匀

![contrast](/assets/img/post/Note/contrast.png)

直方图只能总结图像强度分布，很不一样的图片也可以有一样的直方图

![histogram](/assets/img/post/Note/histogram.png)

增强对比度
- 线性拉伸

![linear stretching](/assets/img/post/Note/linear-stretching.png)

- Log变换：适合图像中很多像素值都很小

![log transform](/assets/img/post/Note/log-transform.png)
