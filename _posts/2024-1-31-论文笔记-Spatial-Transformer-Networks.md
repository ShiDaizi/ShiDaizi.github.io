---
title: "论文笔记：Spatial Transformer Network"
categories:
  - Thesis Notes
tags:
  - Image Registration
mathjax: true
classes: wide
---





论文：https://proceedings.neurips.cc/paper_files/paper/2015/hash/33ceb07bf4eeb3da587e268d663aba1a-Abstract.html

### 平移不变性(Translation Invariance)

CNN网络具有平移不变性，简单地说，卷积+最大池化约等于平移不变性。当目标在池化单元（Max pooling）内任意变换的话，激活的值可能是相同的，这就带来了一点点的不变性。但是池化单元一般都很小（一般是2*2），只有在深层的时候特征被处理成很小的feature map的时候这种情况才会发生。

### Spatial Transformer Network(STN)

STN对feature map（包括输入图像）进行空间变换，输出一张新的图像。我们希望STN对feature map进行变换后能把图像纠正到成理想的图像，然后丢进网络去识别。下图中，通过对输入的MNIST图像(a)进行STN变换后得到理想的数字图像(c)，以便于网络对图像进行识别(d)。

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/STN_1.png" alt="">

STN将输入图像$$U$$输入网络，生成形变场$$\mathcal{T}_{\theta}$$，其中$$\theta$$为网络输出的参数，形变场依据$$\theta$$生成。具体的，对于目标图像的网格$$G$$，$$G_i=(x_i^s,y_i^s)$$即表示生成图像$$x_i^t,x_j^t$$坐标点的像素取自原图的$$x_i^s,y_i^s$$像素点。以仿射变换为例：


$$
\begin{equation*}
\begin{pmatrix}
x_i^s \\ y_i^s
\end{pmatrix}
=\mathcal{T}_{\theta}(G_i)=
A_\theta
\begin{pmatrix}
x_i^t\\ y_i^t\\ 1
\end{pmatrix}
\end{equation*}
=
\begin{bmatrix}
\theta_{11}\ \theta_{12}\ \theta_{13}\\
\theta_{21}\ \theta_{22}\ \theta_{23}
\end{bmatrix}
\begin{pmatrix}
x_i^t\\ y_i^t\\ 1
\end{pmatrix}
$$


其中第三维度数字$$1$$表示平移变换(Translation)。

网络结构如下图：

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/STN_2.png" alt="">

### Differentiable Image Sampling

为了实现sample之后的图像关于网络参数$$\theta$$是可导的，论文采用了下述的计算公式：


$$
\begin{equation*}
V_i^c = \sum\limits_{n}^{H}\sum\limits_{m}^{W}U_{nm}^{c}k(x_i^s - m, \Phi_x)k(y_i^s-n,\Phi_y)\forall i\in[1,H'W']\forall c\in[1..C]
\end{equation*}
$$


其中$$U$$，$$V$$分别是$$H\times W\times C$$和$$H'\times W'\times C'$$的图像，上下标对应像素点与通道，$$\Phi_x,\Phi_y$$是核函数$$k()$$的参数。

一般采用双线性插值（bilinear sampling）：


$$
\begin{align*}
V_i^c = \sum\limits_{n}^{H}\sum\limits_{m}^{W}U_{nm}^{c}\times \max(0,1-|x_i^s-m|)\times\max(0, 1-|y_i^s-n|)\forall i\in[1,H'W']\forall c\in[1..C]
\end{align*}
$$


也有integer sampling kernel：


$$
\begin{align*}
V_i^c = \sum\limits_{n}^{H}\sum\limits_{m}^{W}U_{nm}^{c}\times \delta(\lfloor x_i^s+0.5\rfloor -m)\times\delta(\lfloor y_i^s+0.5\rfloor-n)\forall i\in[1,H'W']\forall c\in[1..C]
\end{align*}
$$


$$\lfloor x\rfloor$$表示对$$x$$进行四舍五入操作。



### Conclusion

STN模块可以插入CNN网络中作为一个独立模块以提高网络准确性。另外，STN所提出的形变场也可以利用与图像配准领域(Image Registration)
