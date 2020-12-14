---
title: Transformation Matrix
typora-root-url: ../../source
typora-copy-images-to: ../images/Transformation_matrix.assets
mathjax: true
categories: Computer Vision
tags:
- 视觉测量
- 位姿估计
---

### 两个坐标系之间的转换关系：R(旋转矩阵) 、T（平移矩阵）

&emsp;&emsp;三维重建方法通常会自己估计相机的$R,T $矩阵，这些矩阵定义了一个世界坐标系，在使用客观的评估方法如[Middlebury](http://vision.middlebury.edu/mview/eval/)来评估精度时，需要使用评估方法提供的相机的R,TR,T矩阵，这些矩阵定义了另外一个世界坐标系，两者通常会有**尺度、旋转、平移**的差别，这就需要在坐标系之间进行转换。

<!--more-->

### 尺度相同

&emsp;&emsp;两个相同尺度的世界坐标系可以通过$R,T$进行转换，计算转换关系需要知道双方$N$个对应点的坐标，设为$A$,$B$，则求解$B=R∗A+T$即可。由于$N$可能比较大，因此此方程通常为**超定方程**，可使用**奇异值分解**(Singular Value Decomposition (SVD))进行计算，其内部原理是最小二乘法。
$$
centroid_A =\frac{1}{N} ∑^N_{i=1}(A)
$$

$$
H=∑^N_{i=1}(P^{i}_{A}−centroid_A)(P^i_B−centroid_B)^T
$$

$$
[U,S,V]=SVD(H)
$$

$$
R=V U^T
$$

$$
T=−R∗centroid_A+centroid_B
$$

&emsp;&emsp;其中$centroid_A$和$centroid_B$是$A,B$的**平均中心**。

```matlab
%计算平均中心点
  centroid_A = mean(A);
  centroid_B = mean(B);

  N = size(A,1);
  H = (A - repmat(centroid_A, N, 1))' * (B - repmat(centroid_B, N, 1));

  [U,S,V] = svd(H);

  R = V*U';
  if det(R) < 0
  	printf('Reflection detected\n');
  	V(:,3) = -1*V(:,3);
  	R = V*U';
  end

  t = -R*centroid_A' + centroid_B';
  detr=det(R)
```

### 存在的一些问题：

&emsp;&emsp;转换模型精度与公共控制点的数量的关系值得探讨