---
title: optimization method
typora-root-url: ../../source
typora-copy-images-to: ../images/optimization method.assets
mathjax: true
categories: Computer Vision
tags:
- 视觉测量
- 位姿估计
---

##### 最小二乘法

##### 非线性：最速下降法、牛顿法、高斯牛顿法（GN）、列文伯格-马夸尔特（LM）联系

##### 线性：QR分解、乔姆斯基分解法求解（Cholesky）、奇异值分解（SVD） 

##### 一.最小二乘问题

​	通常可以表述为,通过搜集到的一些数据(获取得到的样本),对某一个模型进行拟合,并尽可能的使得模型结果和样本达到某种程度上的最佳拟合，在SLAM中亦可以看作为观测值和模型估计值之间的误差；

1.线性最小二乘与非线性最小二乘的关系

线性：直接对目标函数求导，令其等于零，以此求得极值，比较后得到全局最小值。

非线性：由于函数复杂无法直接写出导数形式，无法直接得到全局最小值。退而求其次，从一个初始估计值通过不断迭代寻找局部最小值，不断寻找梯度并下降。

