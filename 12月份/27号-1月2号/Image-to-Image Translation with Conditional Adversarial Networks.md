# Image-to-Image Translation with Conditional Adversarial Networks

## 作者：Phillip Isola Jun-Yan Zhu Tinghui Zhou Alexei A. Efros 时间：2017年

## 会议：CVPR

## 前言：

* 图像处理的很多问题都是将一张输入的图片转变为一张对应的输出图片，比如灰度图、梯度图、彩色图之间的转换等。通常每一种问题都使用特定的算法（比如：使用CNN来解决图像转换问题时，要根据每个问题设定一个特定的loss函数来让CNN去优化，而一般的方法都是训练CNN去缩小输入和输出的欧氏距离，但这样通常会得到比较模糊的输出）。这些方法的本质其实都是从像素到像素的映射。于是论文在GAN的基础上提出一个通用方法：pix2pix来解决这一类问题。通过pix2pix来完成成对的图像转换，可以得到比较清晰的结果。下面是效果图![img](./Image-to-Image%20Translation%20with%20Conditional%20Adversarial%20Networks_img/20190623102047723.png)

## 方法：

* pix2pix是在CGAN的基础上进行研究的，我们知道CGAN与GAN的区别就是在两个网络的输入中加入了condition，用于生成特定的图像，这个condition可以是标签也可以是其他信息。于是在这样的想法下，pix2pix通过添加condition，来达到相应的目的。
* 值得说明的是，原来CGAN的G网络的输入是噪声z和条件信息（比如标签），但是针对pix2pix其一开始输入是噪声z和条件信息（这里的条件信息是指模式图pattern），但是作者在论文中提到，噪声z的加入会让效果稍微有些不好，因此最后的输入仅仅只是条件信息（模式图），其网络结构为：![img](./Image-to-Image%20Translation%20with%20Conditional%20Adversarial%20Networks_img/13016998-d4a85e19c4122023.png)
* 训练大致过程如上图所示。图片 x 作为此cGAN的条件，需要输入到G和D中。G的输入是{x,z}（其中，x 是需要转换的图片，z 是随机噪声），输出是生成的图片G(x,z)。D则需要分辨出{x,G(x,z)}和{x,y}。

## 网络架构:

* 生成器网络受U-Net架构的启发。U-Net架构和自编码网络的架构非常相似，区别在于U-Net使用了跳跃连接（skip connection），跳跃连接可以将部分有用的重要信息共享到生成器中。![img](./Image-to-Image%20Translation%20with%20Conditional%20Adversarial%20Networks_img/007S8ZIlgy1ghtt7sbqtlj30k60er420.jpg)
* 判别器网络收到了PatchGAN结构的启发。PatchGAN包含8个卷积块、1个全连接层和1个扁平化层组成。
  * 为什么使用PatchGAN？其实是因为一般GAN的判别器只需要针对输入进来的数据输出true or false的矢量，代表对数据的对整张图像的评价。但是PatchGAN输出的是一个NxN的矩阵，这个NxN的矩阵里的每一个元素是true or false。其实这样的做法就是不直接判断一副图像到底是否为真，而是将图像分成多个patch，并将其作为输入，用于判定这些patch是否为真，在一幅图像上使用卷积来进行这个操作，最后为得出多个output，最终输入就是这些patch结果的平均。因此这样做的好处在于判断一个patch而不是整个图像，会让判别器更关注细节，也就是高频信息，作者也提到，PatchGAN可以理解为一个texture/style loss

## 重要思想：

* G网络的输入只有一个参考图像，去掉噪声作为输入
* 在G网络中使用U-net结构（加入了skip connection），使用了patch GAN和L1的双loss组合

## 局限：

* 因为使用skip connection，生成的图像细节更好，但是输入的图像和输出的图像大致相同，生成的图像不可能多样化