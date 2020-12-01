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

研究现有相机标定的方法的步骤和机理，探讨主动视觉的标定新方法，张正友法，相机纯旋转运动，相机平移运动的对极几何约束，三角测量方法，变焦距成像模型；

#### 张正友法机理

<!--more-->

1）标定模型的建立
$$
\lambda p=K\left[\mathbf{r}_{1} \quad \mathbf{r}_{2} \quad t\right]\left[\begin{array}{c}X^{w} \\ Y^{w} \\ 1\end{array}\right] \tag{1}
$$

$$
\lambda p=H \hat{P}\tag{2}
$$

$$
H=\left[\begin{array}{lll}h_{1} & h_{2} & h_{3} \\ h_{4} & h_{5} & h_{6} \\ h_{7} & h_{8} & h_{9}\end{array}\right] \tag{3}
$$

 2）单映性矩阵H的求解

将H矩阵变换成向量的形式： 
$$
h=\left[h_{1}, h_{2}, h_{3}, h_{4}, h_{5}, h_{6}, h_{7}, h_{8}, h_{9}\right]^{T} \tag{4}
$$
带入成像模型得到下式子，含有 8 个未知数，因此只需要获取到 4 个及以上的图像的空间参考点世界坐标，便可以利用最小二乘法求解得到 h ，进而恢复单映性矩阵 H；
$$
\left[\begin{array}{ccccccccc}X^{w} & Y^{w} & 1 & 0 & 0 & 0 & -X^{w} u & -Y^{w} u & -u \\ 0 & 0 & 0 & X^{w} & Y^{w} & 1 & -X^{w} v & -Y^{w} v & -v\end{array}\right] h=\left[\begin{array}{l}0 \\ 0\end{array}\right] \tag{5}
$$
3）内参数的求解 
$$
\left[\begin{array}{lll}\mathbf{h}_{1} & \mathbf{h}_{2} & \mathbf{h}_{3}\end{array}\right]=s K\left[\begin{array}{lll}\mathbf{r}_{1} & \mathbf{r}_{2} & t\end{array}\right] \tag{6}
$$
因为R为正交矩阵：
$$
\mathbf{r}_{1}^{T} \mathbf{r}_{2}=0 \quad\left\|\mathbf{r}_{1}\right\|=\left\|\mathbf{r}_{2}\right\|=1 \tag{7}
$$
可得到下列关系：
$$
\left\{\begin{array}{c}\mathbf{h}_{1}^{T} K^{-T} K^{-1} \mathbf{h}_{2}=0 \\ \mathbf{h}_{1}^{T} K^{-T} K^{-1} \mathbf{h}_{1}=\mathbf{h}_{2}^{T} K^{-T} K^{-1} \mathbf{h}_{2}\end{array}\right. \tag{8}
$$

$$
B = K^{-T} K^{-1}  \tag{9}
$$

B 为一个对称矩阵:
$$
B=\left[\begin{array}{lll}B_{11} & B_{12} & B_{13} \\ B_{12} & B_{22} & B_{23} \\ B_{13} & B_{23} & B_{33}\end{array}\right] \tag{10}
$$
将B矩阵变成向量形式，然后可得到如下关系：
$$
\left[\begin{array}{c}V_{12}^{T} \\ \left(V_{11}-V_{22}\right)^{T}\end{array}\right] b=0 \tag{11}
$$
其中：
$$
\begin{array}{l}V_{12}=\left[\begin{array}{lllcc}h_{1} h_{2} & h_{4} h_{5}+h_{2} h_{4} & h_{4} h_{5} & h_{2} h_{7}+h_{1} h_{8} & h_{7} h_{8}+h_{2} h_{6} & h_{3} h_{6}\end{array}\right]^{T} \\ V_{11}=\left[\begin{array}{lllll}h_{1}^{2} & h_{2} h_{4}+h_{1} h_{4} & h_{4}^{2} & 2 h_{1} h_{7} & h_{7}^{2}+h_{2} h_{3} & h_{3}^{2}\end{array}\right]^{T} \\ V_{22}=\left[\begin{array}{lllll}h_{2}^{2} & h_{5}^{2}+h_{2} h_{5} & h_{5}^{2} & 2 h_{2} h_{8} & h_{8}^{2}+h_{5} h_{6} & h_{6}^{2}\end{array}\right]^{T}\end{array} \tag{12}
$$
Extraction of the Intrinsic Parameters from Matrix B：$B = λK^{−T}K$
$$
\begin{aligned} v_{0} &=\left(B_{12} B_{13}-B_{11} B_{23}\right) /\left(B_{11} B_{22}-B_{12}^{2}\right) \\ \lambda &=B_{33}-\left[B_{13}^{2}+v_{0}\left(B_{12} B_{13}-B_{11} B_{23}\right)\right] / B_{11} \\ \alpha &=\sqrt{\lambda / B_{11}} \\ \beta &=\sqrt{\lambda B_{11} /\left(B_{11} B_{22}-B_{12}^{2}\right)} \\ \gamma &=-B_{12} \alpha^{2} \beta / \lambda \\ u_{0} &=\gamma v_{0} / \beta-B_{13} \alpha^{2} / \lambda \end{aligned} \tag{13}
$$


4）内外参数的优化求解

以上步骤得到的相机参数是从数值意义上得到的（即为数值解），并不具有物理意义。因此，Zhang 的方法接下来利用最大似然估计的方法，从物理意义上进一步优化求解以上参数。 对于拍摄的n幅图像（每幅图像上选择 m 个特征点），建立以空间特征点实际投影和理想投影误差最小为目标的代价函数：
$$
K, R, t=\arg \min \left(\sum_{i=1}^{n} \sum_{j=1}^{m} \| p_{i j}-\left.\hat{p}\left(K, R, t, P_{i j}\right)\right|^{2}\right) \tag{14}
$$
通过将步骤 3）中计算得到的结果作为初值，带上式 ，利用 Levenberg-Marquardt 迭代 算法优化求解得到最终的相机内外参数。

通常情况下，相机的镜头是包含畸变的。因此，在综合考虑畸变和噪声的影响后：
$$
K, R, t, k_{1}, k_{2}, p_{1}, p_{2}=\arg \min \left(\sum_{i=1}^{n} \sum_{j=1}^{m} \| p_{i j}-\left.\hat{p}\left(K, R, t, k_{1}, k_{2}, p_{1}, p_{2}, P_{i j}\right)\right|^{2}\right) \tag{15}
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

<img src="/images/Camera-calibration.assets/image-20201105151604443.png" alt="image-20201105151604443" style="zoom: 33%;" />

如图1所示，以其中一对匹配点为例。$P$为空间中的一点（坐标未知），其在左边图像中的投影为$P_l =  [u_l,v_l,1 ] ^T$，在右边图像中的投影为$P_r =  [u_r,v_r,1 ] ^T$,当我们以相机坐标系$O_L$为参考坐标系时，有：
$$
\left\{\begin{array}{l}s_{l} p_{l}=K P \\ s_{r} p_{r}=K(R P+t)\end{array} \Longrightarrow\left\{\begin{array}{l}s_{l} K^{-1} p_{l}=P \\ s_{r} K^{-1} p_{r}=R P+t\end{array}\right.\right. \tag{1}
$$
其中${s}_l、{s}_r$分别为点$P$在相机坐标系$O_L$、$O_L$中的$ z $坐标值（即深度），$K $为3 X 3的相机内参矩阵，$R 、t $即为$O_L$、$O_L$之间的相对位姿关系。取$x_l = K^{-1} P_l$，$x_r = K^{-1} P_r$， 则有：
$$
\left\{\begin{array}{l}s_{l} x_{l}=P \\ s_{r} x_{r}=R P+t\end{array} \Longrightarrow s_{r} x_{r}=R\left(s_{l} x_{l}\right)+t \Longrightarrow s_{r} x_{r}=s_{l} R x_{l}+t\right. \tag{2}
$$
上式两边同时左乘$t$的**反对称矩阵**$t^{\wedge}$，相当于两侧同时与t做外积（$t$ 与自身的外积$t^{\wedge} t = 0$）:
$$
s_{r} t^{\wedge} x_{r}=s_{l} t^{\wedge} R x_{l} \tag{3}
$$
然后，两侧同时左乘$x^{T}_{r}$ :
$$
s_{r} x^{T}_{r} t^{\wedge} x_{r}=s_{l} x^{T}_{r} t^{\wedge} R x_{l} \tag{4}
$$
上式左侧中，$ t $ 与 $ x_{r}$ 的外积$t^{\wedge} x_{r}$ 是一个与$ t $ 和 $ x_{r} $都垂直的向量，所以，再将$x^{T}_{r}$ 与 $t^{\wedge} x_{r}$做内积，其结果必为0，从而有：
$$
 s_{l} x_{r}^{\mathrm{T}} t^{\wedge} R x_{l}=0 \Longrightarrow s_{l}\left(K^{-1} p_{r}\right)^{\mathrm{T}} t^{\wedge} R\left(K^{-1} p_{l}\right)=0 \Longrightarrow p_{r}^{\mathrm{T}} K^{-\mathrm{T}} t^{\wedge} R K^{-1} p_{l}=0 \tag{5}
$$
(5)式以非常简洁的形式描述了两幅图像中匹配点对$ p_l$和$ p_r$之间存在的数学关系，这种关系就叫**对极约束**。另外，我们称$E = t^\wedge R$ 为**本质矩阵**（[Essential Matrix](https://en.wikipedia.org/wiki/Essential_matrix)），称$ F = K^{-T} t^\wedge  R K^{-1} = K^{-T} E K^{-1}$为**基础矩阵**（[Fundamental Matrix](https://en.wikipedia.org/wiki/Fundamental_matrix_(computer_vision))），于是(5)式可以进一步简化为：
$$
x_r^{\text T}Ex_l=p_r^{\text T}Fp_l=0\tag{6}
$$
现在我们需要重新明确一下对极几何的工作：对极几何是要利用已知的$n$对如上述$p_l$、$ p_r$这样的匹配点（二维像素坐标），来计算相机的运动$R$, $t$，即求解本质矩阵 $ E $ 或基础矩阵 $F $。

对极几何存在的问题：根据对极约束求解得到的相机旋转运动R R*R*是准确的，平移运动t t*t*是不具备真实尺度的。

##### 三角测量原理：

 三角测量（[Triangulation](https://en.wikipedia.org/wiki/Triangulation)）又叫三角化，是根据前后两帧图像中匹配到的特征点像素坐标以及两帧之间的相机运动$R$、$t$，计算特征点三维空间坐标的一种算法。直观来讲，当有两个相对位置已知的相机同时拍摄到同一物体时，如何根据两幅图像中的信息估计出物体的实际位姿，即通过三角化获得二维图像上对应点的三维结构，这正是三角测量要解决的问题。

<img src="/images/Camera-calibration.assets/image-20201108164010432.png" alt="image-20201108164010432" style="zoom:33%;" />

如图所示为三角测量基本原理的示意图，该图与相机平移运动对极几何关系图非常相似，但应该注意其中的区别：此时我们已经知道了$O_L$与$O_R$的关系，$p_l$、$p_r$也是已知的，要求的是$P$的三维空间坐标。由(1)式我们知道，要求点$P$的三维坐标，需要先求深度$ s$ ；由(2)式我们有：
$$
s_rx_r = s_lRx_l+t\tag{7}
$$
对上式两侧左乘一个$x_r$的反对称矩阵$x_r^\wedge$，得：
$$
s_rx_r^\wedge x_r = s_lx_r^\wedge Rx_l+x_r^\wedge t\implies s_lx_r^\wedge Rx_l+x_r^\wedge t=\bf 0 \tag{8}
$$
  (8)式是一个超定方程组（[Overdetermined system](https://en.wikipedia.org/wiki/Overdetermined_system)），通常只能求得它的最小二乘解。当我们求出$s_l$后，根据(7)式就很容易求出$s_r$，最后由(1)式，我们有：
$$
\left\{\begin{array}{ll}s_{l} K^{-1} p_{l}=P_{O L}, & P_{O L} \text { 表示点 } P \text { 在相机坐标系 } O_{L} \text { 下的三维坐标 } \\ s_{r} K^{-1} p_{r}=P_{O_{R}}, & P_{O_{R}} \text { 表示点 } P \text { 在相机坐标系 } O_{R} \text { 下的三维坐标 }\end{array}\right.\tag {9}
$$
上式中$P_{O_L}$与$P_{O_R}$之间的关系为：
$$
P_{O_R}=RP_{O_L}+t\tag{10}
$$
即：以$O_L$标系为参考，$O_R$经过运动$R$ 、$t$变换到$O_L$，那么原来在$O_L$坐标系下的点$P$在$O_R$坐标系下表示为$P_{O_R}=RP_{O_L}+t$。如果我们还知道相机与世界坐标系的变换关系$T$，就可以将$P_{O_L}$或$P_{O_R}$转换到世界坐标系下表示，得到点$P$的世界坐标。

##### 同一特征点点变焦距成像模型：

<img src="/images/Camera-calibration.assets/image-20200924092818130.png" alt="image-20200924092818130" style="zoom:67%;" />

该模型包括 5 个坐标系：世界坐标系$O_wX_wY_wZ_w$ 、变焦相机基坐标系${O}_{c0}X_{c0}Y_{c0}Z_{c0}$ 、变焦相机动坐标系${O}_{cn}X_{cn}Y_{cn}Z_{cn}$、图像物理坐标$O_{1xy}$、图像像素坐标系$O_{0uv}$；

当变焦相机的焦距位于任意位置$f_n$时，物体的三维空间点P通过变焦相机的动坐标系光心$O_{cn}$，并映射到变焦相机的像平面上点$p_{cn}$位置。当变焦相机的焦距位于$f_0$时，点P通过变焦相机的基坐标系光心$O_{c0}$，并映射到变焦相机的像平面上点$p_{c0}$位置。

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
