---
title: Camera Calibration
typora-root-url: ../../source
typora-copy-images-to: ../images/Camera-calibration.assets
mathjax: true
categories: Computer Vision
tags:
- 视觉测量
- 位姿估计
---

#### Camera calibration

研究现有相机标定的方法的步骤和机理，探讨主动视觉的标定新方法；

##### 张正友法

<!--more-->

1）标定模型的建立
$$
\lambda p=K\left[\mathbf{r}_{1} \quad \mathbf{r}_{2} \quad t\right]\left[\begin{array}{c}X^{w} \\ Y^{w} \\ 1\end{array}\right]
$$

$$
\lambda p=H \hat{P}
$$

$$
H=\left[\begin{array}{lll}h_{1} & h_{2} & h_{3} \\ h_{4} & h_{5} & h_{6} \\ h_{7} & h_{8} & h_{9}\end{array}\right]
$$

 2）单映性矩阵H的求解

将H矩阵变换成向量的形式： 
$$
h=\left[h_{1}, h_{2}, h_{3}, h_{4}, h_{5}, h_{6}, h_{7}, h_{8}, h_{9}\right]^{T}
$$
带入成像模型得到下式子，含有 8 个未知数，因此只需要获取到 4 个及以上的图像的空间参考点世界坐标，便可以利用最小二乘法求解得到 h ，进而恢复单映性矩阵 H；
$$
\left[\begin{array}{ccccccccc}X^{w} & Y^{w} & 1 & 0 & 0 & 0 & -X^{w} u & -Y^{w} u & -u \\ 0 & 0 & 0 & X^{w} & Y^{w} & 1 & -X^{w} v & -Y^{w} v & -v\end{array}\right] h=\left[\begin{array}{l}0 \\ 0\end{array}\right]
$$
3）内参数的求解 
$$
\left[\begin{array}{lll}\mathbf{h}_{1} & \mathbf{h}_{2} & \mathbf{h}_{3}\end{array}\right]=s K\left[\begin{array}{lll}\mathbf{r}_{1} & \mathbf{r}_{2} & t\end{array}\right]
$$
因为R为正交矩阵：
$$
\mathbf{r}_{1}^{T} \mathbf{r}_{2}=0 \quad\left\|\mathbf{r}_{1}\right\|=\left\|\mathbf{r}_{2}\right\|=1
$$
可得到下列关系：
$$
\left\{\begin{array}{c}\mathbf{h}_{1}^{T} K^{-T} K^{-1} \mathbf{h}_{2}=0 \\ \mathbf{h}_{1}^{T} K^{-T} K^{-1} \mathbf{h}_{1}=\mathbf{h}_{2}^{T} K^{-T} K^{-1} \mathbf{h}_{2}\end{array}\right.
$$

$$
B = K^{-T} K^{-1} 
$$

B 为一个对称矩阵:
$$
B=\left[\begin{array}{lll}B_{11} & B_{12} & B_{13} \\ B_{12} & B_{22} & B_{23} \\ B_{13} & B_{23} & B_{33}\end{array}\right]
$$
将B矩阵变成向量形式，然后可得到如下关系：
$$
\left[\begin{array}{c}V_{12}^{T} \\ \left(V_{11}-V_{22}\right)^{T}\end{array}\right] b=0
$$
4）内外参数的优化求解

以上步骤得到的相机参数是从数值意义上得到的（即为数值解），并不具有物理意义。因此，Zhang 的方法接下来利用最大似然估计的方法，从物理意义上进一步优化求解以上参数。 对于拍摄的n幅图像（每幅图像上选择 m 个特征点），建立以空间特征点实际投影和理想投影误差最小为目标的代价函数：
$$
K, R, t=\arg \min \left(\sum_{i=1}^{n} \sum_{j=1}^{m} \| p_{i j}-\left.\hat{p}\left(K, R, t, P_{i j}\right)\right|^{2}\right)
$$
通过将步骤 3）中计算得到的结果作为初值，带上式 ，利用 Levenberg-Marquardt 迭代 算法优化求解得到最终的相机内外参数。

通常情况下，相机的镜头是包含畸变的。因此，在综合考虑畸变和噪声的影响后：
$$
K, R, t, k_{1}, k_{2}, p_{1}, p_{2}=\arg \min \left(\sum_{i=1}^{n} \sum_{j=1}^{m} \| p_{i j}-\left.\hat{p}\left(K, R, t, k_{1}, k_{2}, p_{1}, p_{2}, P_{i j}\right)\right|^{2}\right)
$$

##### 加入相机二维纯旋转运动（俯仰运动和偏航运动）约束后：

$H$为相机内参数,$[R_{1},T_{1}]$为初始相机外参数，$s$为尺度因子，$R_{Ti}$为第$i$幅图像与初始图像的旋转矩阵，可以通过经纬仪得到；
$$
\begin{aligned}\left[\begin{array}{c}s_{i} \boldsymbol{p}_{i} \\ s_{i} \\ 1\end{array}\right]=\left[\begin{array}{cc}\boldsymbol{H} & \boldsymbol{0} \\ \boldsymbol{0} & \boldsymbol{1}\end{array}\right] \times\left[\begin{array}{cc}\boldsymbol{R}_{\mathrm{ct}} & \boldsymbol{T}_{\mathrm{ct}} \\ \boldsymbol{0} & \boldsymbol{1}\end{array}\right]^{-1} \times\left[\begin{array}{cc}\boldsymbol{R}_{T i} & \boldsymbol{0} \\ \boldsymbol{0} & \boldsymbol{1}\end{array}\right] \times \left[\begin{array}{cc}\boldsymbol{R}_{\mathrm{ct}} & \boldsymbol{T}_{\mathrm{ct}} \\ \mathbf{0} & \mathbf{1}\end{array}\right] \times\left[\begin{array}{cc}\boldsymbol{R}_{1} & \boldsymbol{T}_{1} \\ \mathbf{0} & \mathbf{1}\end{array}\right] \times \boldsymbol{P} \end{aligned}
$$

$$
\boldsymbol{R}_{Ti}=\left[\begin{array}{ccc}\cos \varphi_{i} & 0 & -\sin \varphi_{i} \\ 0 & 1 & 0 \\ \sin \varphi_{i} & 0 & \cos \varphi_{i}\end{array}\right]\left[\begin{array}{ccc}1 & 0 & 0 \\ 0 & \cos \psi_{i} & \sin \psi_{i} \\ 0 & -\sin \psi_{i} & \cos \psi_{i}\end{array}\right]
$$

##### 加入相机纯平移运动约束后：

极几何约束：

<img src="/images/Camera-calibration.assets/image-20200923111946021.png" alt="image-20200923111946021" style="zoom:50%;" />

相机不同位置拍摄同一个点时，这两幅图存在如下约束：
$$
x_{1}^{T}\left(t \times d_{1} x_{1}\right)=x_{1}^{T} t \times R d_{0} x_{0}
$$

##### 同一特征点点变焦距成像模型：

<img src="/images/Camera-calibration.assets/image-20200924092818130.png" alt="image-20200924092818130" style="zoom:67%;" />

该模型包括 5 个坐标系：世界坐标系$O_wX_wY_wZ_w$ 、变焦相机基坐标系${O}_{c0}X_{c0}Y_{c0}Z_{c0}$ 、变焦相机动坐标系${O}_{cn}X_{cn}Y_{cn}Z_{cn}$、图像物理坐标$O_{1xy}$、图像像素坐标系$O_{0uv}$；

当变焦相机的焦距位于任意位置fn时，物体的三维空间点P通过变焦相机的动坐标系光心$O_{cn}$，并映射到变焦相机的像平面上点$p_{cn}$位置。当变焦相机的焦距位于$f_0$时，点P通过变焦相机的基坐标系光心$O_{c0}$，并映射到变焦相机的像平面上点$p_{c0}$位置。

变焦相机动坐标系${O}_{cn}X_{cn}Y_{cn}Z_{cn}$的$Z_c$轴与相机的光轴 $Z_{c0}$重合，它垂直于图像平面并通过图像坐标系的中心，$X_{cn}$轴和$Y_{cn}$轴分别平行于图像坐标系的u轴和v轴，光心到图像平面的距离 $O_{cn}O_1$称为变焦相机的实时焦距$f_n$，设变焦相机坐标系${O}_{cn}X_{cn}Y_{cn}Z_{cn}$中的物体空间点 P 在图像物理坐标系$O_{1xy}$下的成像像点为$p_{cn}=[x,y]T$ ，则可以得到：
$$
\left\{\begin{array}{l}u=u_{0}+\frac{f_{n}}{d x} \frac{X_{c n}}{Z_{c n}} \\ v=v_{0}+\frac{f_{n}}{d y} \frac{Y_{c n}}{Z_{c n}}\end{array}\right.
$$
根据相机坐标系下空间点和图像点之间的转换关系，相机的内参数矩阵由式中矩阵 M1 表示。当空间变焦相机的焦距位于基准焦距 f0 时，可以得到：
$$
\left[\begin{array}{l}u \\ v \\ 1\end{array}\right]=\left[\begin{array}{ccc}\frac{f_{0}}{d x} & 0 & u_{0} \\ 0 & \frac{f_{0}}{d y} & v_{0} \\ 0 & 0 & 1\end{array}\right]\left[\begin{array}{c}\frac{X_{c 0}}{Z_{c 0}} \\ \frac{Y_{C 0}}{Z_{c 0}} \\ 1\end{array}\right]=\mathbf{M}_{1}\left[\begin{array}{c}\frac{X_{c 0}}{Z_{c 0}} \\ \frac{Y_{c 0}}{Z_{c 0}} \\ 1\end{array}\right]
$$
当相机的焦距发生改变，$\alpha_{x}=\frac{f_{c n}}{d x}$为u轴上的放大系数,$\alpha_{y}=\frac{f_{c n}}{d y}$为v轴上的放大系数，相机的外参数矩阵用$M_2$表示，$T_{wen}$为3×1平移矩阵，$R_{wen}$为3×3为正交旋转矩阵,则三维空间点与二维图像点之间的映射关系可以通过下式进行描述：
$$
Z_{c n}\left[\begin{array}{l}u \\ v \\ 1\end{array}\right]=\left[\begin{array}{cccc}\alpha_{x} & 0 & u_{0} & \\ 0 & \alpha_{y} & v_{0} & 0 \\ 0 & 0 & 1 & 0\end{array}\right]\left[\begin{array}{cc}X_{w} & \\ 0 & 1\end{array}\right]\left[\begin{array}{c}\mathbf{R}_{\mathrm{wen}} \mathbf{T}_{\mathrm{wen}} \\ Y_{w} \\ Z_{w} \\ 1\end{array}\right]=\mathbf{M}_{1} \mathbf{M}_{2}\left[\begin{array}{c}X_{w} \\ Y_{w} \\ Z_{w} \\ 1\end{array}\right]=\mathbf{M}\left[\begin{array}{c}X_{w} \\ Y_{w} \\ Z_{w} \\ 1\end{array}\right]
$$
