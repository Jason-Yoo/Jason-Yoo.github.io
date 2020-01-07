---
title: 相机畸变模型
typora-root-url: ../../source
typora-copy-images-to: ../images/3D-2D_match.assets
mathjax: true
categories: Computer Vision
tags:
- 视觉测量
- 相机模型
---

计算机视觉领域广泛应用的畸变模型：

$\begin{equation}
\left[\begin{array}{lll}{x,} & {y,} & {1+k\left(x^{2}+y^{2}\right)}\end{array}\right]^{\top} \simeq P X
\end{equation}$

其中 $(x,y)$ 为投影的3D点 $X$ 的中心图像坐标,其中 P 为摄像机投影矩阵，$k$为畸变系数；

来源：

Simultaneous linear estimation of multiple view geometry and lens distortion

解决的问题：3D-2D点的矫正（由于镜头畸变造成）

解决模型：

1.真正的镜头畸变曲线通常非常复杂，而处理非线性畸变的系统需要使用高阶模型或查找表来校准相机。然而，对于计算机视觉，特别是对于匹配，像素顺序的精度是很重要的。因此，通常将失真函数L扩展为泰勒级数，并且只保留第一个非线性偶数项；

2.新的失真方程

$$\begin{aligned}\left(\begin{array}{c}{p} \\ {q} \\ {1}\end{array}\right) &=\left(\begin{array}{c}{x} \\ {y} \\ {1+\lambda\left(x^{2}+y^{2}\right)}\end{array}\right) \\ &=\left(\begin{array}{c}{x} \\ {y} \\ {1}\end{array}\right)+\lambda\left(\begin{array}{c}{0} \\ {0} \\ {\|x\|^{2}}\end{array}\right) \end{aligned}$$
