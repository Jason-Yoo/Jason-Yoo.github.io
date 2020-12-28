---
title: Coordinate Transformation Matrix
typora-root-url: ../../source
typora-copy-images-to: ../images/Transformation_matrix.assets
mathjax: true
categories: Computer Vision
tags:
- 视觉测量
- 位姿估计
---

### 三维坐标系之间的转换关系：R(旋转矩阵) 、T（平移矩阵）

&emsp;&emsp;在大地测量、工程测量、摄影测量等领域中，坐标系之间的转换是必不可少的。空间坐标转换的实质是用公共点的２套坐标和非公共点的１套坐标推估非公共点的另１套坐标。

&emsp;&emsp;坐标转换过程通常分２步，先由公共点坐标解算转换参数，再由转换参数转换非公共点。转换参数通常分为旋转、平移和尺度参数，其中旋转参数的确定是坐标转换的核心。

&emsp;&emsp;传统的三维坐标转换模型是用３个旋转角作为旋转参数，建立的模型是非线性的，常需要用泰勒级数展开的方法将模型线性化，计算比较繁杂。

&emsp;&emsp;在小角度旋转情况下，可对旋转矩阵作近似处理，得到线性模型，如常用的布尔莎模型。

&emsp;&emsp;针对大旋角的坐标转换问题，多采用罗德里格矩阵表示旋转矩阵的坐标转换方法，仅有３个旋转参数，计算过程无需线性化，且能适用大旋角转换。

<img src="https://picture-share-1303204700.cos.ap-nanjing.myqcloud.com/uPic/WzYxlM.png" style="zoom:50%;" />

<!--more-->

### 数学模型描述

##### 布尔莎模型:前提是旋转角为微小旋转角

设矩阵 A 为 A 坐标系下公共点的三维坐标，矩阵B为B坐标系下公共点的三维坐标。由三维坐标转换模型可得，A、B 两坐标系的三维坐标转换方程如下所示：
$$
\left[\begin{array}{l}x \\ y \\ z\end{array}\right]_{B}=\left[\begin{array}{l}\Delta x \\ \Delta y \\ \Delta z\end{array}\right]+(1+k) \boldsymbol{R}\left[\begin{array}{l}x \\ y \\ z\end{array}\right]_{A}\tag{1}
$$
式中：$Δx、Δy、Δz$ 表示坐标原点的平移量，$k$ 为尺度因子，$R$ 为 A 站位到 B 站位的旋转矩阵。为了对坐标转换精度进行分析，将三维坐标转换模型作如下简化:
$$
\left[\begin{array}{c}x \\ y \\ z\end{array}\right]_{B}=k\left[\begin{array}{c}x \\ y \\ z\end{array}\right]_{A}+\left[\begin{array}{ccc}0 & -\varepsilon_{z} & \varepsilon_{y} \\ \varepsilon_{z} & 0 & -\varepsilon_{x} \\ -\varepsilon_{y} & \varepsilon_{x} & 0\end{array}\right] \cdot\left[\begin{array}{l}x \\ y \\ z\end{array}\right]_{A}+\left[\begin{array}{c}\Delta x \\ \Delta y \\ \Delta z\end{array}\right]  \tag{2}
$$
应注意到，式(2)中 $ε_x 、ε_y 、ε_z$ 均为微小角，则$sin{x} = x ,cos{x}=1 $因此，应在利用坐标转换模型前通过初步的坐标系转换得到一组转站参数的初值，使 A、B 两坐标系之间的转换满足微小角近似。式(2)可进一步写成：
$$
\left[\begin{array}{l}x \\ y \\ z\end{array}\right]_{B}=\left[\begin{array}{lllllll}1 & 0 & 0 & 0 & z & -y & x \\ 0 & 1 & 0 & -z & 0 & x & y \\ 0 & 0 & 1 & y & -x & 0 & z\end{array}\right]_{A} \cdot\left[\begin{array}{l}\Delta x \\ \Delta y \\ \Delta z \\ \varepsilon_{x} \\ \varepsilon_{y} \\ \varepsilon_{z} \\ k\end{array}\right] \tag{3}
$$
根据式(3)，可将 n 个公共点的坐标转换写为$ B=AX$ 的形式，其中：
$$
\boldsymbol{A}=\left(\begin{array}{ccccccc}1 & 0 & 0 & 0 & z_{1} & -y_{1} & x_{1} \\ 0 & 1 & 0 & -z_{1} & 0 & x_{1} & y_{1} \\ 0 & 0 & 1 & y_{1} & -x_{1} & 0 & z_{1} \\ 1 & 0 & 0 & 0 & z_{2} & -y_{2} & x_{2} \\ 0 & 1 & 0 & -z_{2} & 0 & x_{2} & y_{2} \\ 0 & 0 & 1 & y_{2} & -x_{2} & 0 & z_{2} \\ 1 & 0 & 0 & 0 & z_{n} & -y_{n} & x_{n} \\ 0 & 1 & 0 & -z_{n} & 0 & x_{n} & y_{n} \\ 0 & 0 & 1 & y_{n} & -x_{n} & 0 & z_{n}\end{array}\right)_{A}
$$
利用最小二乘法对转换参数进行参数估计，可得转换参数 $X$ 的估值为:
$$
\boldsymbol{X}=\left(\boldsymbol{A}^{\mathrm{T}} \boldsymbol{Q}^{-1} \boldsymbol{A}\right)^{-1} \boldsymbol{A}^{\mathrm{T}} \boldsymbol{Q} \boldsymbol{B}
$$
式中：$Q$ 为B 坐标系下$ n$ 个公共点坐标测量误差的协方差矩阵,则得转换参数方差阵为：
$$
\boldsymbol{Q}_{x}=\left(\boldsymbol{A}^{\mathrm{T}} \boldsymbol{Q}^{-1} \boldsymbol{A}\right)^{-1}=\left[\begin{array}{cccc}q_{11} & q_{12} & \cdots & q_{17} \\ q_{21} & q_{22} & \cdots & q_{27} \\ & & \vdots & \\ q_{71} & q_{72} & \cdots & q_{77}\end{array}\right]
$$
对于空间任意一点 $P$，其坐标转换误差方差阵为:
$$
\boldsymbol{Q}_{P}=\boldsymbol{B}_{P} \boldsymbol{Q}_{x} \boldsymbol{B}_{P}^{\mathrm{T}}=\left[\begin{array}{lll}q_{x x} & q_{x y} & q_{x z} \\ q_{y x} & q_{y y} & q_{y z} \\ q_{z x} & q_{z y} & q_{z z}\end{array}\right]
$$

##### 罗德里格矩阵模型

**空间直角坐标系转换模型**
$$
\left[\begin{array}{l}X \\ Y \\ Z\end{array}\right]=\lambda \boldsymbol{R}\left[\begin{array}{l}x \\ y \\ z\end{array}\right]+\left[\begin{array}{l}\Delta X \\ \Delta Y \\ \Delta Z\end{array}\right] \tag{1}
$$
$\lambda$为尺度比例因子，假设其初值是１；$R$ 为$3×3$的旋转矩阵；

引入一个具有３个独立元素的反对称矩阵:
$$
S=\left[\begin{array}{rrr}0 & -c & -b \\ c & 0 & -a \\ b & a & 0\end{array}\right]	\tag{2}
$$
式中 $ a,b,c$ 相互独立，则罗德里格矩阵可由反对称矩阵构建为：
$$
R=(I+S)(I-S)^{-1} \tag{3}
$$
式中$R$是个正交矩阵，其中 $I$ 是 ３ 阶单位阵,将 $R$ 展开为:
$$
\begin{array} & R=\frac{1}{1+a^{2}+b^{2}+c^{2}}  {\left[\begin{array}\\1+a^{2}-b^{2}-c^{2} & -2 c-2 a b & -2 b+2 a c \\ 2 c-2 a b & 1-a^{2}+b^{2}-c^{2} & -2 a-2 b c \\ 2 b+2 a c & 2 a-2 b c & 1-a^{2}-b^{2}+c^{2}\end{array}\right]}\end{array} \tag{4}
$$
将旋转矩阵带入式（1）可解得平移矢量:
$$
\left[\begin{array}{c}X_{i} \\ Y_{i} \\ Z_{i}\end{array}\right]=\frac{\lambda}{1+a^{2}+b^{2}+c^{2}}\left[\begin{array}{ccc}1+a^{2}-b^{2}-c^{2} & -2 a b-2 c & -2 b+2 a c \\ 2 c-2 a b & 1-a^{2}+b^{2}-c^{2} & -2 a-2 b c \\ 2 a c+2 b & 2 a-2 b c & 1-a^{2}-b^{2}+c^{2}\end{array}\right]\left[\begin{array}{c}x_{i} \\ y_{i} \\ z_{i}\end{array}\right]+\left[\begin{array}{c}\Delta X \\ \Delta Y \\ \Delta Z\end{array}\right]\tag{5}
$$
式（5）中未知参数共有 ７ 个，即 $λ 、ａ 、 ｂ 、 ｃ 、 ΔＸ 、  Δ Ｙ 、 ΔＺ $，进行线性化处理得到误差方程：
$$
\boldsymbol{v}=\boldsymbol{A} \Delta \boldsymbol{x}+\boldsymbol{l} \tag{6}
$$
对式（6）,利用最小二乘法可得到：
$$
\boldsymbol{\Delta} \boldsymbol{x}=\left(\boldsymbol{A}^{\mathrm{T}} \boldsymbol{P} \boldsymbol{A}\right)^{-1}\left(\boldsymbol{A}^{\mathrm{T}} \boldsymbol{P} \boldsymbol{l}\right) \tag{7}
$$
式（7）中，$P$位单位矩阵；
$$
\boldsymbol{A}=\left[\begin{array}{ccccccc}\frac{\partial X_{i}}{\partial \lambda} & \frac{\partial X_{i}}{\partial a} & \frac{\partial X_{i}}{\partial b} & \frac{\partial X_{i}}{\partial c} & \frac{\partial X_{i}}{\partial \Delta X} & \frac{\partial X_{i}}{\partial \Delta Y} & \frac{\partial X_{i}}{\partial \Delta Z} \\ \frac{\partial Y_{i}}{\partial \lambda} & \frac{\partial Y_{i}}{\partial a} & \frac{\partial Y_{i}}{\partial b} & \frac{\partial Y_{i}}{\partial c} & \frac{\partial Y_{i}}{\partial \Delta X} & \frac{\partial Y_{i}}{\partial \Delta Y} & \frac{\partial Y_{i}}{\partial \Delta Z} \\ \frac{\partial Z_{i}}{\partial \lambda} & \frac{\partial Z_{i}}{\partial a} & \frac{\partial Z_{i}}{\partial b} & \frac{\partial Z_{i}}{\partial c} & \frac{\partial Z_{i}}{\partial \Delta X} & \frac{\partial Z_{i}}{\partial \Delta Y} & \frac{\partial Z_{i}}{\partial \Delta Z}\end{array}\right]
$$

$$
\Delta \boldsymbol{x}=\left[\begin{array}{llllll}\Delta \lambda & \Delta a & \Delta b & \Delta c & \Delta  X & \Delta  Y & \Delta Z\end{array}\right]^{\mathrm{T}}
$$

$$
\boldsymbol{l}=\left[\begin{array}{lll}l_{1} & l_{2} & l_{3}\end{array}\right]^{\mathrm{T}}
$$

**转换参数求解**

参数解算过程可分３步，先求尺度参数，再求旋转参数，最后求平移参数。 尺度参数可由２个公共点在不同坐标系下的距离之比算出：
$$
\lambda=\frac{\sqrt{\left(X_{2}-X_{1}\right)^{2}+\left(Y_{2}-Y_{1}\right)^{2}+\left(Z_{2}-Z_{1}\right)^{2}}}{\sqrt{\left(x_{2}-x_{1}\right)^{2}+\left(y_{2}-y_{1}\right)^{2}+\left(z_{2}-z_{1}\right)^{2}}}
$$
公共点较多时，可求出各点间多个距离比，再取平均值；

在解算旋转参数 $ａ,b，c $时，可以先消去平移参数，将２个公共点的坐标代入式（ 1）,分别按照式（1） 做差得:
$$
\left[\begin{array}{l}X_{2}-X_{1} \\ Y_{2}-Y_{1} \\ Z_{2}-Z_{1}\end{array}\right]=\lambda \boldsymbol{R}\left[\begin{array}{l}x_{2}-x_{1} \\ y_{2}-y_{1} \\ z_{2}-z_{1}\end{array}\right]
$$
联系式（2）式（3） 可得：
$$
\left[\begin{array}{ccc}0 & \lambda z_{21}+Z_{21} & \lambda y_{21}+Y_{21} \\ \lambda z_{21}+Z_{21} & 0 & -\lambda x_{21}-X_{21} \\ -\lambda y_{21}-Y_{21} & -\lambda x_{21}-X_{21} & 0\end{array}\right]\left[\begin{array}{l}a \\ b \\ c\end{array}\right]=\left[\begin{array}{l}\lambda x_{21}-X_{21} \\ \lambda y_{21}-Y_{21} \\ \lambda z_{21}-Z_{21}\end{array}\right]
$$
式中：
$$
\begin{array}{l}x_{21}=x_{2}-x_{1} ; y_{21}=y_{2}-y_{1} ; z_{21}=z_{2}-z_{1} \\ X_{21}=X_{2}-X_{1} ; Y_{21}=Y_{2}-Y_{1} ; Z_{21}=Z_{2}-Z_{1}\end{array}
$$
这个方程组左边的系数矩阵为奇异阵，３个方程里仅有２个独立，需要至少２个这样的方程组才能解算出ａ，ｂ，ｃ，也就是至少需要 ３ 个公共点。 当有ｎ个公共点时，可列出（ n－１）个形如上式的方程 ，共有 3（n-1）个方程，其总误差方程为：
$$
V=\underset{3(n-1) \times 3} {B} {\space\space} \underset{3  \times 1}{X}-\underset{3(n-1) \times 3}{L}
$$
 式中：
$$
B=\left[\begin{array}{ccc}0 & -\lambda z_{21}-Z_{21} & -\lambda y_{21}-Y_{21} \\ -\lambda z_{21}-Z_{21} & 0 & \lambda x_{21}+X_{21} \\ \lambda y_{21}+Y_{21} & \lambda x_{21}+X_{21} & 0 \\ \vdots & \vdots & \vdots \\ 0 & -\lambda z_{n 1}-Z_{n 1} & -\lambda y_{n 1}-Y_{n 1} \\ -\lambda z_{n 1}-Z_{n 1} & 0 & \lambda x_{n 1}+X_{n 1} \\ \lambda y_{n 1}+Y_{n 1} & \lambda x_{n 1}+X_{n 1} & 0\end{array}\right]
$$

$$
X=[a, b, c]^{\mathrm{T}}
$$

$$
\begin{array}{cccc}L= & {\left[X_{21}-\lambda x_{21}\right.} & Y_{21}-\lambda y_{21} & Z_{21}-\lambda z_{21} & \cdots \\ X_{n 1}-\lambda x_{n 1} & Y_{n 1}-\lambda y_{n 1} & \left.Z_{n 1}-\lambda z_{n 1}\right]^{\mathrm{T}}\end{array}
$$

按最小二乘法间接平差原理求解未知数:
$$
X=\left(A^{\mathrm{T}} A\right)^{-1} A^{\mathrm{T}} L
$$
计算出 $ａ,b，c $后，即可求出旋转矩阵，然后按下式求解平移参数：
$$
\left[\begin{array}{l}\Delta X \\ \Delta Y \\ \Delta Z\end{array}\right]=\left[\begin{array}{l}X_{i} \\ Y_{i} \\ Z_{i}\end{array}\right]-\lambda R\left[\begin{array}{l}x_{i} \\ y_{i} \\ z_{i}\end{array}\right]
$$

### 尺度相同:

&emsp;&emsp;三维重建方法通常会自己估计相机的$R,T $矩阵，这些矩阵定义了一个世界坐标系，在使用客观的评估方法如[Middlebury](http://vision.middlebury.edu/mview/eval/)来评估精度时，需要使用评估方法提供的相机的$R,T$矩阵，这些矩阵定义了另外一个世界坐标系，两者通常会有**尺度、旋转、平移**的差别，这就需要在坐标系之间进行转换。

&emsp;&emsp;两个相同尺度的世界坐标系可以通过$R,T$进行转换，计算转换关系需要知道双方$N$个对应点的坐标，设为$A$,$B$，则求解$B=R∗A+T$即可。由于$N$可能比较大，因此此方程通常为**超定方程**，可使用**奇异值分解**(Singular Value Decomposition (SVD))进行计算，其内部原理是最小二乘法。

对公共点坐标进行重心化处理:
$$
centroid_A =\frac{1}{N} ∑^N_{i=1}(A)
$$

$$
centroid_B =\frac{1}{N} ∑^N_{i=1}(B)
$$

求：
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

&emsp;&emsp;转换模型精度与公共控制点的数量的关系值得探讨；目前坐标转换精度的评价方式通常是利用转站前后公共点坐标差值的均方根值来进行评价。该方法只能获得用于计算的公共点坐标转换精度，并不能得到空间中任意被测点的转换精度分布情况，对于提高整体测量精度以及改进公共点布设方案缺乏指导意义。

### 主要参考来源：

 刘猛奎, 赵明金, 石波, 等. 基于 RANSAC 的坐标系转换抗差算法研究[J]. 全球定位系统, 2019, 44(1): 39-47.

韩梦泽, 李克昭. 基于罗德里格矩阵的空间坐标转换[J]. 测绘工程, 2016 (4): 25-27.

张皓琳, 林嘉睿, 邾继贵. 三维坐标转换精度及其影响因素的研究[J]. 光电工程, 2012, 39(10): 26-31.

https://proj.org/operations/transformations/index.html；



