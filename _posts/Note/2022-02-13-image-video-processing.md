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

2D傅立叶旋转：空间可频域一起转

![2d ft rotate](/assets/img/post/Note/2d-ft-rotate.png)

# DSFT, DFT
![image](https://user-images.githubusercontent.com/29757093/154558905-18148dd9-20e4-4141-8c40-4b3f3d652b80.png)

空间内直线在dft里垂直方向会有一条亮线，用所有的频率做一条直线出来

![image](https://user-images.githubusercontent.com/29757093/154559492-b0e67d8e-aba1-403b-892f-45f0a8384dbd.png)

![image](https://user-images.githubusercontent.com/29757093/154559546-ce9974ad-125e-492a-9ffc-3acbd9c0632e.png)

有周期的会有亮点；亮线和图像线垂直；很规整的sinc，比较发散

# 卷积

![image](https://user-images.githubusercontent.com/29757093/154561517-ac5d5d52-9aae-440e-92a9-333e2e6aad4e.png)

2D卷积两个坐标轴翻转，到第三象限

滤波器：h(m,n) 一个系统的冲击反馈（point spread function）

point spread function：一个点的输入一般result in一个区域，point spread越小分辨率越高

![image](https://user-images.githubusercontent.com/29757093/154564052-8bb8a906-f029-44d1-b060-32f0448dfa15.png)

- 图像M\*N，filter K*L，输出 M+N-1， K+L-1
- 图像M*N，filter (2k+1, 2k+1)，蓝色和橙色区域取决于padding

![image](https://user-images.githubusercontent.com/29757093/154564909-a85e6421-fa64-4dc4-86c0-ff77f7274c93.png)

可分滤波器可以在x，y上单独做

![image](https://user-images.githubusercontent.com/29757093/154567418-4bcd6c8a-bebd-4391-9a49-6626ef6ac10d.png)

![image](https://user-images.githubusercontent.com/29757093/154567739-7750ed85-249a-44ed-92e9-657134f94244.png)

![image](https://user-images.githubusercontent.com/29757093/154567903-a7563d93-3f26-418e-8db5-4c38e983fc9c.png)


# 采样&插值

![sample](/assets/img/post/Note/sample.png)
- 采样：连续到离散
- 插值：离散到连续

符号
- 采样间隔：$\Delta_x$，$\Delta_y$
- 采样频率：$f_{s,x}=\frac{1}{\Delta_x}$，$f_{s,y}=\frac{1}{\Delta_y}$，时间维度的采样率fps
- nyquist频率：$f_{m,x}$，$f_{m,x}$，采样频率的一半
- 连续图像：$f(x, y)$
- 采样图像：$f_s(m, n)$
- 重建图像：$\hat{f}(x, y)$

- 采样
  - 采样频率大于1/2信号最大频率
  - 时域乘脉冲序列，间隔 $1/\Delta$，幅度1
  - 频域卷脉冲序列，间隔 $\Delta$，幅度 $\frac{1}{\Delta_x\Delta_y}$
    - 脉冲序列傅立叶还是脉冲序列
    - 通过加频域的多个信号返回时域信号，频域信号的幅度小
- 重建
  - 频域乘低通，截止频率 = 1/2采样频率，幅度 $\Delta_x\Delta_y$
  - 时域和sinc卷积
    - 把sinc放在每一个像素上，幅度是像素强度，最后求和
    - 采样图像里m，n对重建图像x，y的贡献权重是2d sinc在x,y和m，n距离位置的取值

采样：采样结果M行N列
$$
f_s(m, n)=f(m\Delta_x, n\Delta_y) \quad m=0,1,...,M, \quad n=0,1,...,N \\
$$

脉冲序列：
$$
\begin{aligned}
p(x, y)&=\sum_{m=0}^{M-1} \sum_{n=0}^{N-1} \delta\left(x-m \Delta_{x}, y-n \Delta_{y}\right)\\
\end{aligned}
$$

脉冲序列的傅立叶：
- 1D：采样结果长度为N

$$
\begin{aligned}
&p(t)=\sum_{n=0}^{N-1} \delta(t-n \Delta t) \Leftrightarrow P(u)=\frac{1}{\Delta t} \sum_{n=0}^{N-1} \delta(u-nf_s) \\
&\text { where } f_{s}=\frac{1}{\Delta t}
\end{aligned}
$$

频率是 $f_s$，所以频率是 $f_s$ 整数倍的都过采样点
- 2D

$$
\begin{aligned}
&p(x, y)=\sum_{m, n} \delta(x-m \Delta x, y-n \Delta y) \Leftrightarrow P(u, v)=\frac{1}{\Delta x \Delta y} \sum_{m, n} \delta\left(u-m f_{s, x}, v-n f_{s, y}\right) \\
&\text { where } f_{s, x}=\frac{1}{\Delta x}， f_{s, y}=\frac{1}{\Delta y}
\end{aligned}
$$

- 采样时域：信号 $\times$ 脉冲序列
$$
\begin{aligned}
\tilde{f_{s}}(x, y)&=f(x, y) \times p(x, y)\\
&=\sum_{m=0}^{M-1} \sum_{n=0}^{N-1} f\left(m \Delta_{x}, n \Delta_{y}\right) \delta\left(x-m \Delta_{x}, y-n \Delta_{y}\right) \\
&=\sum_{m=0}^{M-1} \sum_{n=0}^{N-1} f_{s}(m, n) \delta\left(x-m \Delta_{x}, y-n \Delta_{y}\right) \\
\end{aligned}
$$
- 采样频域：信号 * impulse train
$$
\begin{aligned}
F_{s}(u, v)&=F(u, v) * P(u, v) \\
P(u, v)&=\frac{1}{\Delta x \Delta y} \sum_{m, n} \delta\left(u-m f_{s, x}, v-n f_{s, y}\right) \\
\Rightarrow F_{s}(u, v)&=\frac{1}{\Delta x \Delta y} \sum_{m, n} F\left(u-m f_{s, x}, v-n f_{s, y}\right) \\
&\text { where } f_{s, x}=\frac{1}{\Delta x}, f_{s, y}=\frac{1}{\Delta y}\\
\end{aligned}
$$
这里如果采样频率不大于固有频率的二倍，频域信号就会有重合，就会有伪影

- 重建时域：采样信号 * 2d sinc
$$
\begin{aligned}
H(u, v)&=\left\{\begin{array}{cc}
\Delta x \Delta y & |u| \leq \frac{f_{s, x}}{2},|v| \leq \frac{f_{s, y}}{2} \\
& 0 \quad \text { otherwise }
\end{array} \Leftrightarrow h(x, y)=\frac{\sin \pi f_{s, x} x}{\pi f_{s, x} x} \cdot \frac{\sin \pi f_{s, y} y}{\pi f_{s, y} y}\right.\\

\hat{f}(x, y)&=\tilde{f}_{s}(x, y) * h(x, y) \\
&=\sum_{m=0}^{M-1} \sum_{n=0}^{N-1} f_{s}(m, n) \delta\left(x-m \Delta_{x}, y-n \Delta_{y}\right) * h(x, y) \\
&=\sum_{m=0}^{M-1} \sum_{n=0}^{N-1} f_{s}(m, n)
h\left(x-m \Delta_{x}, y-n \Delta_{y}\right) \\
h(x, y)&=\frac{\sin \left(\pi f_{s, x} x\right)}{\pi f_{s, x} x} \frac{\sin \left(\pi f_{s, y} y\right)}{\pi f_{s, y} y} \\
\hat{f}(x, y)&=\sum_{m=0}^{M-1} \sum_{n=0}^{N-1} f_{s}(m, n) \frac{\sin \pi f_{s, x}(x-m \Delta x)}{\pi f_{s, x}(x-m \Delta x)} \frac{\sin \pi f_{s, y}(y-m \Delta y)}{\pi f_{s, y}(y-m \Delta y)}
\end{aligned}
$$


$采样率低\to空间间隔大\to频域间隔小\to和冲击卷完有叠加\to采样完图像里能看到比原图周期低的信号\to伪影\to重建信号比原信号频率低$

采样的频率应该大于信号固有频率的2倍，一个周期至少2个点

![1d aliasing ](/assets/img/post/Note/1d-aliasing.png)

![2d sampling](/assets/img/post/Note/2d-sampling.png)

![2d aliasing](/assets/img/post/Note/2d-aliasing.png)

x采样率够，y采样率不够

![2d cos](/assets/img/post/Note/2d-cos.png)

jaggie

![image downsample](/assets/img/post/Note/image-downsample.png)


插值滤波不可能是理想低通，那样空间域无限大。Nyquist filter 性质：
- 低通：超过采样频率一半的信号都是重复的
- 递减：离一个点越远，对这个点插值的贡献应该越小
- 偶函数
- 可分： $h(x,y)=h_v(x) \cdot h_h(y)$，减少计算
- $h(0,0)=1$,  $h(m\Delta_x, n\Delta_y)=0$：采样值原样进入重建图像

![Nyquist Filters](/assets/img/post/Note/nyquist-filters.png)

prefilter，sampling filter：实际图像里可能包含非常高频的信号，在采样之前先过一个截止频率 $f_c=\frac{f_s}{2}$ 的滤波器。不做prefilter可以保留更多细节，但是会有低频的纹理

![prefilter](/assets/img/post/Note/prefilter.png)

- aliasing：采样前滤波带来的问题。原信号经过prefilter还是有高频信号，带来比1/2采样频率低的噪声。体现在直的线变阶梯，条纹有比原来频率低的。
- imaging：插值的滤波器带来的问题。不是理想低通，截止频率之外还是有信号，带来比1/2采样频率高的噪声
![aliasing imaging](/assets/img/post/Note/aliasing-imaging.png)

相机
- 相邻两个同颜色传感器中心的距离是采样间隔
- 传感器的输出是一个面积上的光强总数，相当于采样前滤波去掉高频信号

aliasing 例子 https://www.red.com/red-101/cinema-temporal-aliasing

观察的时候眼睛和脑子做插值的低通滤波

角频率：看的时候重要的是单位角度的频率
- 屏幕越远频率越高
- 屏幕越大频率越低

![angular frequency](/assets/img/post/Note/angular-frequency.png)

$$
\begin{aligned}
&\theta=2 \arctan (h / 2 d)(\operatorname{radian}) \approx 2 \mathrm{~h} / 2 \mathrm{~d}(\operatorname{radian})=\frac{180}{\pi} \frac{h}{d}(\text { degree }) \\
&\mathrm{f}_{\theta}=\frac{\mathrm{f}_{s}}{\theta}=\frac{\pi}{180} \frac{d}{h} \mathrm{f}_{s}(\text { cycle/degree })
\end{aligned}
$$

![eye spatial](/assets/img/post/Note/eye-spatial.png)

![eye temperal](/assets/img/post/Note/eye-temperal.png)

- 时间上通常能看到60hz，实际可以更快因为眼睛会跟着物体动，相对频率变小。显示60hz的信号需要120hz的采样率。
- 空间上通常能看到30 cycle per degree，空间上应该捕捉到60 cycle per degree
- 时间上频率越高，空间上同样角频率的信号越不敏感。时间快的场景分辨率就可以低
- 亮度越高，能分辨的帧率越高

![cpd example](/assets/img/post/Note/cpd-example.png)

眼睛对亮度更敏感，4个Y不需要4个cbcr

![4210](/assets/img/post/Note/4210.png)


下采样周期变短，频率变高，可能会alias，下采样前先过一个half band filter，cutoff频率应该是固有频率1/4

![down half band](/assets/img/post/Note/down-half-band.png)

upsample：填0，卷积
![upsample process](/assets/img/post/Note/upsample-process.png)
$$
\begin{aligned}
&\tilde{f}(m, n)=\left\{\begin{array}{cc}
f(m / K, n / K) & \text { if } m, n \text { are multiple of } K \\
0 & \text { otherwise }
\end{array}\right. \\
&f_{u}(k, l)=\sum_{k, l} \tilde{f}(m, n) h(k-m, l-n)=\tilde{f}(k, l) * h(k, l)
\end{aligned}
$$
- nearest neighbor：新值取决于跟原图里哪个像素最近
  - $\mathrm{O}\left[\mathrm{m}^{\prime}, \mathrm{n}^{\prime}\right]=\mathrm{I}[(\text { int })(\mathrm{m}+0.5),(\text { int })(\mathrm{n}+0.5)], \mathrm{m}=\mathrm{m}^{\prime} / \mathrm{M}, \mathrm{n}=\mathrm{n}^{\prime} / \mathrm{M}$
- bilinear 双线性内插值：新值用原图里最近的四个点线性加权平均
  ![seperate bilinear](/assets/img/post/Note/seperate-bilinear.png)
  - 分开插值
    - 比如先沿行算整行算新点的所在的列应该是多少
    F[m,n’]=(1-a)*I[m,n]+a*I[m,n+1], a=n’-n.
    - 之后再沿整列算新点所在的行应该是多少
    O[m’,n’]=(1-b)*F[m’,n]+b*F[m’+1,n], b=m’-m
  - 整体插值
    - 每个点的权重是新点和对角点组成矩形的面积
    O[m’,n’]=(1-a)*(1-b)*I[m,n]+a*(1-b)*I[m,n+1]+(1-a)*b*I[m+1,n]+a*b*I[m+1,n+1]
- bicubic 双三次插值：新值用原图里最近16个点线性加权平均，系数里最高是a和b的立方
$$
\begin{aligned}
F\left[m^{\prime}, n\right] &=-b(1-b)^{2} I[m-1, n]+\left(1-2 b^{2}+b^{3}\right) I[m, n]+b\left(1+b-b^{2}\right) I[m+1, n]-b^{2}(1-b) I[m+2, n] \quad
m= (int) \frac{m^{\prime}}{M}, \quad b=\frac{m^{\prime}}{M}-m \\

O\left[m^{\prime}, n^{\prime}\right]&=-a(1-a)^{2} F\left[m^{\prime}, n-1\right]+\left(1-2 a^{2}+a^{3}\right) F\left[m^{\prime}, n\right]+a\left(1+a-a^{2}\right) F\left[m^{\prime}, n+1\right]-a^{2}(1-a) F\left[m^{\prime}, n+2\right],
where n= (int) \frac{n^{\prime}}{M}, a=\frac{n^{\prime}}{M}-n
\end{aligned}
$$

![bicubic interpolation](/assets/img/post/Note/bicubic-interpolation.png)

上采样周期变长，频率变小，可能会引入高频信号，上采样之后过一个half band filter
![up half band](/assets/img/post/Note/up-half-band.png)


H hermission 转制+共扼


![typo](/assets/img/post/Note/typo.png)

rou d
