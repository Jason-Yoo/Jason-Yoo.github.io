---
title: Some steps in slam
typora-root-url: ../../source
typora-copy-images-to: ../images/Slam_VIO.assets
mathjax: true
categories: Computer Vision
tags:
- SLAM
- 位姿估计
---

视觉与IMU融合的分类：

- 松耦合和紧耦合：按照是否把图像的Feature加入到状态向量区分，换句话说就是松耦合是在视觉和IMU各自求出的位姿的基础上做的耦合。

- 滤波法和优化法：

  滤波：**MSCKF**

  图优化：**VINS**，**OKVIS，ORB-SLAM**

那么为什么要进行视觉与IMU的融合呢，自己总结的主要有以下几点：

- 视觉与IMU的融合可以借助IMU较高的采样频率，进而提高系统的输出频率。
- 视觉与IMU的融合可以提高视觉的鲁棒性，如视觉SLAM因为某些运动或场景出现的错误结果。
- 视觉与IMU的融合可以有效的消除IMU的积分漂移。
- 视觉与IMU的融合能够校正IMU的Bias。
- 单目与IMU的融合可以有效解决单目尺度不可观测的问题。

优秀的开源VIO：vinsmono、vinsfusion、 vi_orbslam2

IMU预积分：



