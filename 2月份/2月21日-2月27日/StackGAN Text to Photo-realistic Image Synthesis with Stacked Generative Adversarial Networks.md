# StackGAN: Text to Photo-realistic Image Synthesis with Stacked Generative Adversarial Networks

## 作者：Han Zhang，Tao Xu，Hongsheng Li，Shaoting Zhang，Xiaogang Wang， Xiaolei Huang，Dimitris Metaxas 时间：2017年

## 会议：ICCV

## 前言：

* 从文本描述中合成高质量的图像是计算机视觉中的一个具有挑战性的问题，并具有许多实际应用。现有的文本-图像方法生成的样例可以大致反映给定描述的含义，但不能包含必要的细节和生动的对象部分，因此，在该文中，作者们提出了StackGAN来生成基于文本描述的256x256真实感图像。该文通过一个粗略的细化过程将困难的问题分解为更易于管理的子问题。网络模型分为Stage-ⅠGAN和Stage-Ⅱ GAN。
  * Stage-I GAN根据给定的文本描述勾画出物体的原始形状和颜色，生成Stage-I低分辨率图像
  * Stage-II  GAN将Stage-I结果和文本描述作为输入，并生成具有真实感细节的高分辨率图像。它能够纠正阶段i结果中的缺陷，并在细化过程中添加引人注目的细节。

## 网络模型：

### 1.目标函数

* ![image-20220225235028621](D:\workplace\note\数字水印论文\笔记\2月份\2月21日-2月27日\StackGAN Text to Photo-realistic Image Synthesis with Stacked Generative Adversarial Networks_img\image-20220225235028621.png)

### 2.结构

* ![image-20220225235245906](D:\workplace\note\数字水印论文\笔记\2月份\2月21日-2月27日\StackGAN Text to Photo-realistic Image Synthesis with Stacked Generative Adversarial Networks_img\image-20220225235245906.png)

### 3.条件增强技术（Conditioning Augmentation）

* 由网络结构图可以看出，在第一阶段开始时刻，文本描述t首先经编码器编码得到描述文本的嵌入向量。而在此之前，处理这个问题的方法通常是将其非线性的转换成条件的隐变量输入到G中，但是这样做有一个很大的问题，通过因为隐变量所在的空间维度很高，在数据量有限的情况下，它会造成数据流形的不连续性，不易用来训练G。
* 因此为了缓解这个问题，作者提出了一种新的条件增强技术，他会利用嵌入向量生成一个新的条件标量，并将其输入到G中。并且这个条件标量是从一个独立的高斯噪声中随机采样得到。该高斯分布的均值和对角关系函数都是嵌入文本向量的函数。
* 该文提出的条件增强可以在给定少量图像文本对的情况下产生更多的训练对，从而增强对条件流形上的小扰动的鲁棒性

### 4.Stage-I GAN

* 作者们没有直接根据文本秒速生成高分辨率的图像，而是首先使用Stage-I GAN生成一个低分辨率的图像，它只关注于绘制物体的粗略形状和正确的颜色。损失函数为![image-20220226195039912](D:\workplace\note\数字水印论文\笔记\2月份\2月21日-2月27日\StackGAN Text to Photo-realistic Image Synthesis with Stacked Generative Adversarial Networks_img\image-20220226195039912.png)

### 5.Stage-II GAN

* 由Stage-I GAN生成的低分辨率图像通常缺乏逼真的物体部分，并可能包含形状扭曲。文本中的一些细节也可能在第一阶段被省略，这对于生成真实感图像至关重要。我们的Stage-II GAN是在Stage-I GAN结果的基础上生成高分辨率图像的。它以低分辨率的图像为条件，并再次嵌入文本，以纠正Stage-I结果中的缺陷。Stage-II GAN完成了以前忽略的文本信息，已生成更逼真的细节。损失函数为![image-20220226195453242](D:\workplace\note\数字水印论文\笔记\2月份\2月21日-2月27日\StackGAN Text to Photo-realistic Image Synthesis with Stacked Generative Adversarial Networks_img\image-20220226195453242.png)

## 实验结果

* ![image-20220226195615952](D:\workplace\note\数字水印论文\笔记\2月份\2月21日-2月27日\StackGAN Text to Photo-realistic Image Synthesis with Stacked Generative Adversarial Networks_img\image-20220226195615952.png)