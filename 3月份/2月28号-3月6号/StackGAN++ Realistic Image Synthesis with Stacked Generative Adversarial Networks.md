# StackGAN++: Realistic Image Synthesis with Stacked Generative Adversarial Networks

## 作者：Han Zhang，T ao Xu，Hongsheng Li，Shaoting Zhang，Xiaogang Wang，Xiaolei Huang，Dimitris N 时间：2018年

## 会议/期刊：IEEE Trans

## 核心思想：

* 为了使StackGAN更具有通用性，对于原先的Stack GAN的两阶段的堆叠结构改为了树状结构。包含有多个生成器和判别器，并且每个生成器产生的样本分辨率不一样，该文中依次为64 x 64 128 x 128 256 x 256。生成的图片分别送入各自对应的判别器进行判断。这样的多尺度的图片分布的好处在于：如果任何一个尺度的生成图片与该尺度的真实图片的分布尽可能的近似，那么就能够提供很好的梯度信号去稳定或促进真个网络的训练。
* 使用Color-consistency regularization来使得不同发生器的相同输入生成的样本颜色更加一致，从而提高生成图像的质量![image-20220305191046689](D:\workplace\note\数字水印论文\笔记\3月份\2月28号-3月6号\StackGAN++ Realistic Image Synthesis with Stacked Generative Adversarial Networks_img\image-20220305191046689.png)
* 既可以用于text-to-image这样的条件图像生成任务，也可以用于更一般的无条件图像生成任务，均可以取得比其他模型更优异的结果

## 网络结构：

* ![在这里插入图片描述](D:\workplace\note\数字水印论文\笔记\3月份\2月28号-3月6号\StackGAN++ Realistic Image Synthesis with Stacked Generative Adversarial Networks_img\20190506100210717.png)
* 左边为整个网络的结构，右边是判别器网络D的结构图，可以看出，该网络结构有三个生成器和三个判别器。

