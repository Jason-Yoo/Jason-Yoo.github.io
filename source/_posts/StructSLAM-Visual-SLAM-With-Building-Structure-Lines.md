---
title: SLAM
typora-root-url: ../../source
categories: Computer Vision
tags:
- 视觉测量
- SLAM
---

##### StructSLAM: Visual SLAM With Building Structure Lines

建筑物结构线做为特征用于定位和构图，不同于其他线特征，建筑物结构线对全局方向信息进行编码，该信息随着时间的推移限制了摄像机的航向，消除了累积的方向误差，从而减小了位置漂移。

##### Real-Time Loop Closure in 2D LIDAR SLAM

摘要：

这篇论文主要研究减少闭环检验约束条件和闭环计算检验的计算量，提高计算效率，达到实时闭环检测和大平面地图的构建。

<!--more-->

研究方法：

 当一个新的laser scan加入到地图中时，如果该laser scan的估计位姿与地图中某个submap的某个laser scan的位姿比较接近的话，那么通过某种 scan match策略就会找到该闭环（submap和Laser scan的位姿的原始数据是怎么来的呢？人为的存进去的吗？）

 If a sufficiently good match is found in a search window around the currently estimated pose, it is added as a loop closing constraint to theoptimization problem.(什么叫做足够好的匹配结果？将足够好的匹配结果做为约束是指作为位置估计的一个判断吗？)

  We achieve this by using a branch-and-bound approach and several precomputed grids per finished submap.（分支上界法和预处理网格，预处理网格是什么？）

1.Scans

每个点的角度是相对于那条线的角度呢？（代码中得到答案：原始的激光扫描是由浮点的（角度，距离值）组成，离散化后变成整型的（地图索引x，地图索引y））

2.Submaps

这个概率网络是怎么映射的呢？Submap是由多个Scans 建立起来的，包含的信息也只有距离值和角度值，概率网络的P怎么得到？

3.Ceres scan matching

在将一个scan点集插入submap中之前，这个scan点集的位置必须使用Ceres-based的scan matcher的方法进行优化（这个优化是相对于submap的位置）。这个scan matcher目的是为了找到一个点集位置，这个点集的位置在submap中的概率最大。   

优化问题

使用损失函数的目的是减少加入到优化问题中的离群点对于系统的影响。

##### Improved Techniques for Grid Mapping with Rao-Blackwellized Particle Filters

该论文主要解决两个问题：1.粒子自适应重采样；2.考虑机器人移动和观测值的建议分布。

如何在已知地图的情况下采用粒子滤波算法进行精确定位，一般包括以下几个步骤：

（1）给定初始位姿，初始化粒子群，采用高斯分布进行随机采样；

（2）根据运动模型模拟粒子运动；

（3）计算粒子评分

每个粒子的位姿即为假设的机器人位姿，采用bresenham直线段扫面算法，可计算出粒子当前位置与障碍物之间的栅格占据集合，计算出的栅格占据集合与给定的地图进行匹配计算，从而对每个粒子进行评分，选择得分高的粒子作为该时间点的机器人位姿。

（4）粒子群重采样

将评分低的粒子舍弃，将评分高且很接近的粒子都保留下来，并对评分高的粒子进行复制，保持粒子数量不变。

 