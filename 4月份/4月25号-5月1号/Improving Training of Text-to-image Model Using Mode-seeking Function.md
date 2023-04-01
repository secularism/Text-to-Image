# Improving Training of Text-to-image Model Using Mode-seeking Function

## 作者：Naitik Bhise、Zhenfei Zhang、Tien D. Bui 时间：2020年

## 期刊：CVPR

## 论文主旨：

* GAN网络一直以来被用来理解文本和图像之间的语义关系。然而，图像生成中的模式崩溃（也叫做模式坍缩 mode collapsing）会导致一些首选输出模式。**该文的目的是通过一个专门的模式寻找损失函数来改进网络的训练，以避免这个问题。**

## 模式崩溃（mode collapsing）：

### 介绍：

* mode collapse是指生成的图像多样性较差，非常接近数据集中的某一种，以试图蒙骗判别器。比如在数据集中既有猫的数据，也有狗和其他动物的图片，但是网络最终只生成或是大部分生成猫的图片，对于其他动物的图片，生成较少或是没有生过。
* 其产生的原因是：数据集中的图像可分为多个mode(其实就是几大类，也就是上面例子中的各类动物)，有的mode中的图像比较多，称为large mode，有的称为small mode。对应到分布中，就是large mode是比较高的峰值，而small mode对应较低的峰值，采样可能性更小。**因此如果生成large mode的图像，会产生更大的梯度，便于优化目标函数，同时也可以骗过判别器**，因此生成器就会越来越趋向于生成large mode中的图像。
* ![image-20220430154021668](./Improving%20Training%20of%20Text-to-image%20Model%20Using%20Mode-seeking%20Function_img/image-20220430154021668.png)
* 上图$M_1$，$M_3$，$M_5$为small mode，其他是large mode。

### 解决方法：

* 解决模态崩溃问题有两种方法：

  1）通过引入不同的发散度和优化过程来研究鉴别器。

  2）处理多发生器和额外编码器等辅助网络。

* 但是该文使用的是模式寻找损失函数（2019年由MSGAN这篇论文提出），因为与传统方法相比，该方法能最大限度地提高了图像之间的差异，增强了模式，并以一种简单的方式分离它们。当然传统方法本身计算量大，泛化能力差也是其中一个原因。

## 该文的贡献：

* 分析了现有技术的局限性，为了解决这些问题，采用了一种新的损耗函数来改进网络的训练。
* 证明了方法是有效的，参数开销低。
* 该文的工作在CUB数据集和COCO数据机上取得了比以前模型更好的性能。

## 该文提出方法的示意图：

* ![image-20220430155345474](./Improving%20Training%20of%20Text-to-image%20Model%20Using%20Mode-seeking%20Function_img/image-20220430155345474.png)
* 该文使用的是DM-GAN的初始网络。然后使用了AttnGAN中的DAMSM Loss模块来度量图像与文本描述的匹配程度。

## Mode-seeking loss function：

* 该函数是2019年Mao等人在MSGAN论文中提出的。在提取潜在先验的分布时，由于不同的初始向量，会产生许多输出模式。由于随机分布而产生的每一种模式都会产生不同的图像。有些模式比其他模式更受欢迎（可以理解为能更大程度的优化目标函数）。因此Mode-seeking loss函数试图加强主模态间的次模态分布，其表达式为：
* ![image-20220430160806553](./Improving%20Training%20of%20Text-to-image%20Model%20Using%20Mode-seeking%20Function_img/image-20220430160806553.png)
* 其中$D(.,.)$是一个距离函数，它计算两个张量或矢量之间距离的大小。分子是图像向量之间的距离，分母是潜在向量（latent vector）之间的距离，$c$是输入模型的条件。该函数的目的是最大化图像向量之间的距离，从而讲模式从重叠中分离出来。**所以公式中的值越大，就说明latent vector能映射到各种各样的图像，也包括上面提到的smaller mode**，也就避免了模式坍缩这个问题。
* ![image-20220430162350125](./Improving%20Training%20of%20Text-to-image%20Model%20Using%20Mode-seeking%20Function_img/image-20220430162350125.png)
* 上图是MSGAN论文的图，这里用来说明Mode seeking损失函数的作用。图中$M_1$，$M_3$，$M_5$为small mode，其他是large mode。在没有添加损失函数时，可以清楚的看到，模型陷入了Mode collapse（中间第二个图）：也即是latent vector（下方红线的图是latent vector向量的分布）只生成了$M_2$和$M_4$两个large mode的图像（$M_2$和$M_4$代表一种模式，比如$M_2$代表长而黑的耳朵，且脸部为黑色的猫）。但是使用Mode seeking损失函数之后，此时的latent vector不光能生成large mode图像也能生成small mode图像（$Z_1$生成$M_1$的图像）。

## 实验;

* ![image-20220430163814255](./Improving%20Training%20of%20Text-to-image%20Model%20Using%20Mode-seeking%20Function_img/image-20220430163814255.png)

* 这里的$\lambda$是调节因子，用于调节mode-seeking损失函数，其在目标函数中给出：

  ![image-20220430165339644](./Improving%20Training%20of%20Text-to-image%20Model%20Using%20Mode-seeking%20Function_img/image-20220430165339644.png)

  可以看到加入了该损失函数，效果要好很多（比起DMGAN）

* ![image-20220430165927925](./Improving%20Training%20of%20Text-to-image%20Model%20Using%20Mode-seeking%20Function_img/image-20220430165927925.png)
* 可以看到$\lambda$的增加随着清晰度的增加而改善了图像的质量。但它在增加寻模项的强度下，降低了原始损失函数中DAMSM损失和条件损失的重要性（该文使用到了AttnGAN的DAMSM损失模块），可以看到黑色的鸟在更高的$\lambda$中被放大得更好，而其他特征被渲染得不那么重要，（个人认为，$\lambda$并不是越大越好）。

## 总结：

* 该文的主要工作个人认为是将19年MSGAN提出的mode-seek损失函数加入到T2I这项工作中，并对其做了细致化的实验来验证其有效性。阅读完该文，其实给个人更多的感觉是清楚理解了GAN网络中模式崩溃这一问题。