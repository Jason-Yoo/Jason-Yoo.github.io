---
title: 线性代数
typora-root-url: ../../source
typora-copy-images-to: ../images/linear algebra.assets
mathjax: true
categories: Computer Vision
tags:
- 视觉测量
- 数学分析
---

条件数（Condition number）：

条件数是[线性方程组](https://baike.baidu.com/item/线性方程组/5904308)Ax=b的解对b中的误差或不确定度的敏感性的度量。数学定义为矩阵A的条件数等于A的[范数](https://baike.baidu.com/item/范数/10856788)与A的逆的范数的乘积，即
$$
k\left( A \right)=\left| \left| A^{-1} \right| \right|\; \cdot \; \left| \left| A \right| \right|
$$
对应矩阵的3种范数（1范数，2范数，无穷范数），相应地可以定义3种条件数。

数值分析中，一个问题的**条件数**是该数量在数值计算中的容易程度的衡量，也就是该问题的适定性。一个低条件数的问题称为**良态**的，而高条件数的问题称为**病态**（或者说非良态）的。

正则化(Regularization)：

在线性代数中，[不适定问题](https://baike.baidu.com/item/不适定问题)通常是由一组线性代数[方程](https://baike.baidu.com/item/方程/6306)定义的，而且这组方程组通常来源于有着很大的条件数的不适定反问题。大[条件数](https://baike.baidu.com/item/条件数/5293168)意味着舍入误差或其它误差会严重地影响问题的结果。通过正则化的方法可以改善或者减少过度拟合问题。

L~0~正则化：模型参数中非零参数的个数。

从直观上看，利用非零参数的个数，可以很好的来选择特征，实现特征稀疏的效果，具体操作时选择参数非零的特征即可。但因为L~0~正则化很难求解，是个NP难问题，因此一般采用L~1~正则化。L~1~正则化是L~0~正则化的最优凸近似，比容易求解，并且也可以实现稀疏的效果。

L~1~正则化：模型参数中各个参数绝对值之和。

L~2~正则化：各个参数的平方的和的开方值。





