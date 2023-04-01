# UNSUPERVISED REPRESENTATION LEARNING WITH DEEP CONVOLUTIONAL
GENERATIVE ADVERSARIAL NETWORKS
## 作者： Alec Radford  Luke Metz 时间：2016年

## 期刊/会议：ICLR

## 前言

- 近年来，基于卷积神经网络（cnn）的监督学习在计算机视觉应用中得到了大量应用。相比之下基于cnn的无监督学习受到的关注比较少。在这项工作中，该文作者介绍了一类名为深度卷积生曾对抗网络（DCgan）的cnn，并证明了它也是可以进行非监督学习的。
- 另外通过各类的图像数据训练表明。DCGAN的生成器和鉴别器能够学习从物体部分到场景的表示层次。并且能够把学到的特征用于新的任务中，以证明他们可以作为一般图像表征的适用性。
## 架构设计原则
- 该文提出GANs在训练时是不稳定的，经常导致生成器输出没有意义的图片，因此为了GAN能够很好得适应于卷积神经网络架构，DCGAN提出了四点架构设计原则。
	- 使用卷积层替换池化层
		- 具体表现为在D网络中使用strided convolutions，在G网络中使用fractional-strided
		convolutions，这样改进的意义是为了下采样过程不再是固定的抛弃某些位置的像素值，而是可以让网络自己去学习下采样方式。
	- 去除全连接层
		- 全连接层的缺点在于参数过多，当神经网络层数深了以后运算速度变变得非常慢。此外它也会使得网络变得容易过拟合。论文中提出了一种折中的方案，也就是将生成器的随机输入直接与卷积层特征输入进行连接，同样地对于判别器的输出层也是与卷积层的输出特征连接。
		- 作者通过实验发现了全局均值池化有助于模型的稳定性但是降低了模型的收敛速度；并且在这里说明了他们是通过将生成器输入的噪声reshape成4D的张量，来实现不用全连接而是用卷积的。
	- 使用BN层进行批归一化
		- 文中提到通过将每个单元的输入归一化，使其均值和单位方差为零，从而稳定学习。这有助于处理初始化不良而产生的训练问题。
	- 使用恰当的激活函数
		- 生成器除去输出层采用Tanh外，全部使用ReLU作为激活函数，判别器所有层都使用LeakyReLU作为激活函数
	- 上面四点在原文中表现为：
	![](D:\workplace\note\数字水印论文\笔记\12月份\20号-26号\UNSUPERVISED REPRESENTATION LEARNING WITH DEEP CONVOLUTIONAL_img\image.png)

## DCGAN的框架结构

- DCGAN中，判别网络接收图像，使用卷积层和池化层对其进行下采样，然后使用全连接分类层将图像分类为真的或假的。生成网络从潜在空间中获取随机噪声向量，然后通过上采样机制进行上采样，最后生成一张图像。隐藏层是用LeakyReLU作为激活函数，并且使用系数介于0.4-0.7的随机失活来避免过拟合。
- G网络架构图：
	- 如图所示，输入z是100位的随机数据，服从范围在[-1, 1]的均匀分布。经过一系列的空洞卷积之后，形成一张分辨率为64x64x3的图像的过程。
	![](D:\workplace\note\数字水印论文\笔记\12月份\20号-26号\UNSUPERVISED REPRESENTATION LEARNING WITH DEEP CONVOLUTIONAL_img\image_1.png)
- D网络架构图：
	- 判别网络是包含10层（可以添加更多层）的CNN。简单说来，它接收维度为64×64×3的图像，使用2D卷积层对其进行下采样，然后传递给全连接层进行分类。判别网络输出估测，判断给定图像是真是假。输出值为0或1，输出1表示判别网络接收的图像为真，输出0表示该图像为假。
	![](D:\workplace\note\数字水印论文\笔记\12月份\20号-26号\UNSUPERVISED REPRESENTATION LEARNING WITH DEEP CONVOLUTIONAL_img\image_2.png)

## 实验结果

- 使用GANS作为特征提取器对cifar-10进行分类
- 使用GANS作为特征提取器对SVHN数字进行分类
-  WALKING IN THE LATENT SPACE
	- 通过使用插值微调噪音输入z 的方式可以导致隐空间结构发生变化从而引导生成图像发生语义上的平滑过度，比如说从有窗户到没窗户，从有电视到没电视等等。
	![](D:\workplace\note\数字水印论文\笔记\12月份\20号-26号\UNSUPERVISED REPRESENTATION LEARNING WITH DEEP CONVOLUTIONAL_img\image_3.png)
- VISUALIZING THE DISCRIMINATOR FEATURES
	- 在大型图像数据集上训练的无监督DCGAN也可以学习有趣的特征层次结构。识别器学习的特征会在卧室的典型部分激活，比如床和窗户。为了进行比较，在同一张图中，作者给出了随机初始化的特征的基线，这些特征在语义相关或有趣的情况下不会被激活。
	![](D:\workplace\note\数字水印论文\笔记\12月份\20号-26号\UNSUPERVISED REPRESENTATION LEARNING WITH DEEP CONVOLUTIONAL_img\image_4.png)
- FORGETTING TO DRAW CERTAIN OBJECTS
	- 通过标注窗口，并判断激活神经元是否在窗口内的方式来找出影响窗户形成的神经元，将这些神经元的权重设置为0，那么就可以导致生成的图像中没有窗户。从下图可以看到，上面一行图片都是有窗户的，下面一行通过语义遮罩的方式拿掉了窗户，但是空缺的位置依然是平滑连续的，使整幅图像的语义没有发生太大的变化。
- VECTOR ARITHMETIC ON FACE SAMPLES
	- 在向量算法中有一个很经典的例子就是【vector("King") - vector("Man") + vector("Woman") = vector("Queue")】，作者将该思想引入到图像生成当中并得到了以下实验结果：【smiling woman - neutral woman + neutral man = smiling man】
	![](D:\workplace\note\数字水印论文\笔记\12月份\20号-26号\UNSUPERVISED REPRESENTATION LEARNING WITH DEEP CONVOLUTIONAL_img\image_6.png)



