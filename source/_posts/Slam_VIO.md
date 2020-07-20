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

##### 视觉与IMU融合的分类：

- 松耦合和紧耦合：

  松耦合（Loosely Coupled）:

  松耦合是指IMU和相机分别进行自身的运动估计，然后对其位姿估计结果进行融合,两个模块更新频率不一致 , 模块之间存在一定的信息交换，在松耦合方式中以惯性数据为核心 , 视觉测量数据修正惯性测量数据的累积误差。
  紧耦合（Tightly Coupled）:

  紧耦合是指把IMU的状态与相机的状态合并在一起，共同构建运动方程和观测方程，然后进行状态估计,IMU的尺度度量信息可以用于辅助视觉中的尺度的估计。

- 滤波法和优化法：

  滤波：**MSCKF**(Multi-State Constraint KF），**ROVIO**

  图优化：**VINS**，**OKVIS，ORB-SLAM**

##### 视觉与IMU的融合的优势：

- 视觉与IMU的融合可以借助IMU较高的采样频率，进而提高系统的输出频率。
- 视觉与IMU的融合可以提高视觉的鲁棒性，如视觉SLAM因为某些运动或场景出现的错误结果。
- 视觉与IMU的融合可以有效的消除IMU的积分漂移。
- 视觉与IMU的融合能够校正IMU的Bias。
- 单目与IMU的融合可以有效解决单目尺度不可观测的问题。
- 可以应对快速的运动变化，相机在快速运动过程中会出现运动模糊。

##### 存在的问题：

1. 使用 IMU 对相机在快门动作期间内估计相机的运动，但是由于 CMOS 相机的快门时间戳和 IMU 的时间戳的同步比较困难，且相机的时间戳不太准确 ；

##### 优秀的开源：

vinsmono、vinsfusion、 vi_orbslam2

##### 紧耦合-MSCKF：



