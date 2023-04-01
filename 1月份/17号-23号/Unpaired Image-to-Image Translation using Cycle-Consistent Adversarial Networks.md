# Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks

## 作者：Jun-Yan Zhu、Taesung Park、Phillip Isola、Alexei A. Efros 时间：2018年 

## 期刊/会议：ICCV17

## 摘要：

* CycleGAN的主要贡献是在不使用配对的图像对（paired images）数据时，用 GAN 实现了图像到图像的转换（image-to-image translation），即学习从源域图像到目标域图像之间的映射关系。此外除了传统的对抗损失（adversarial loss），还加入了循环一致性损失（cycle consistency loss）来保证转换后的图像保留原图像的细节信息。CycleGAN的另一个一个重要领域是Domain Adaptation（域迁移：可以通俗的理解为画风迁移），比如可以把一张普通的风景照变化成梵高化作，或者将游戏画面变化成真实世界画面等等。以下是原论文中给出的一些应用：

![image-20220123145917811](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/image-20220123145917811.png)

## CycleGAN优势：

+ 在读到这篇论文时，其实发现其与之前阅读的Pix2Pix有类似的地方，都是有风格转换等效果，但是在阅读过程中发现Pix2Pix要求训练数据必须是成对的，而现实生活中，要找到两个域(画风)中成对出现的图片是相当困难的。因此CycleGAN的出现就是解决了这样的一个问题它只需要两种域的数据，而不需要他们有严格对应关系，这使得CycleGAN的应用更为广泛。关于成对数据，原论文中是这样解释的：

![image-20220123145954319](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/image-20220123145954319.png)

## 核心思想：

* 那么对于没有成对的图片要如何进行和完成训练，作者在这里做了一个假设：在源域和目标域存在一种潜在的关系。其实就是两者从分布上而言，是存在相似性的。也就是可以每种场景中的每幅图片在另一个场景中都有它对应的图像
* 由此一来，我们可以联想到GAN的思想，就可以让机器去学习这个转换关系尽管缺乏成对的监督学习样本，我们仍然可以在集合层面使用监督学习：我们在数据域X中给出一组图像，在数据域Y 中给出另外一组图像。我们可以训练一种映射G : X → Y 使得 输出 ![image-20220123155006738](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/image-20220123155006738.png)
* 但是文中提出了单纯使用GAN又出现了一些问题：
  * 无法保证对于一个输入x，可以得到有意义的输出y，因为使用了不成对的训练集，可以学到无数种的映射G，而这些G都可以使生成的分布y^逼近与目标域。
  * 单独优化对抗损失非常困难，导致了一些问题，如模型崩溃。模型崩溃：例如训练集有很多种类别(如猫狗牛羊)，但是我们只能生成狗(或猫或牛或羊)，虽然生成的狗的图片质量特别好，但是整个G就只能生成狗，根本没法生成猫牛羊，陷入一种训练结果不好的状态。
  * 针对这个作者提出了**Cycle-Consistent Adversarial Networks(CycleGan)**实现了非配对的图像到图像转换，换个例子就是将中文用英文翻译出来，再将英文翻译成中文，和原文进行对照，这两个翻译之间要有一致性，双射互逆循环。

## 模型结构：

* ![image-20220123160441199](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/image-20220123160441199.png)
* **G和F应该是互逆的，即两者是一个双向映射。**于是可以同时训练G和F来确保这个性质，在这其中增加一个循环一致性损失，使得**F(G(x))≈x以及G(F(x))≈y**。**组合该损失和对抗损失**，得到了整体的非正对的图像到图像迁移的优化目标。此外，我们还引入了两个对抗性判别器Dx 和 Dy, **Dx的目的是区别图像x和翻译后的图像F(y)，Dy的目的是区分y和G(x)。**

![img](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/14512145-722683a931a80682.png)



## 损失函数：

* 损失函数分为两个，一个是普通的GAN网络的损失函数，一个是CycleGAN网络的损失函数。
  * ![image-20220123161955600](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/image-20220123161955600.png)
  * ![image-20220123162021170](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/image-20220123162021170.png)
  * 其中CycleGAN的损失函数其实就是在双循环映射中降低其损失，G(x)得到Y'，F(Y')得到x'，x'是循环生成之后的图像，再与原图像进行对比(图像中每个像素进行对比)，要使循环得到的x'与原图像x尽量一致，对于G(F(y))得到的y'也是如此。

## 局限：

* CycleGAN在涉及纹理或颜色的之类的图像转换效果好一点，但是存在一定的局限性：
  * 1.在涉及几何转换上有失败，例如下图（狗转变成猫或相反）：

![img](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/14512145-9360b86d9d81b7fa.png)

* 2.由训练及分布特征导致的，比如训练野马和斑马转换时，并没有考虑到马上可能骑着一个人,个人猜测有可能只提取了轮廓部分。

![image-20220123163400015](./Unpaired%20Image-to-Image%20Translation%20using%20Cycle-Consistent%20Adversarial%20Networks_img/image-20220123163400015.png)

* 在匹配的数据集上训练和不匹配的数据集上训练仍然会有差距，作者提出要解决这种问题需要融合一些弱语义监督，这样仍然只会带来比全监督系统低得多的标注成本。

