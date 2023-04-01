# SegAttnGAN: Text to Image Generation with Segmentation Attention

## 作者：Yuchuan Gou，Qiancheng Wu，Minghao Li，Bo Gong，Mei Han 时间：2020

## 期刊/会议：CVPR

## 为什么提出SegAttnGAN：

* 文本生成图像任务的许多模型使用GAN可以保持图像-文本一致性，但是它们除了特定的关键字限制对象的形状外，对生成的图像的布局几乎没有控制。以往的模型通常会生成具有变形形状的物体或具有不显示布局的图像。如下图所示：
* ![image-20220422160739339](./SegAttnGAN%20Text%20to%20Image%20Generation%20with%20Segmentation%20Attention_img/image-20220422160739339.png)
* 但是当利用分割数据的空间注意力来知道图像生成时，可以在图像合成任务中获得良好的结果，因此为了解决形状形变和布局不现实的问题，作者设计了SegAttnGAN，它利用分割来增加文本输入之外的全局空间关注，并且实验表明，当附加分割信息用于指导图像生成时，有良好的效果。

## SegAttnGAN的贡献：

* 提出一种新颖的网络，利用文本和空间注意力来生成逼真的图像。
* 验证了在GAN中加入空间注意机制可以通过调节物体形状和图像布局显著提高视觉真实感。
* 建立了self-attention network来生成segmentation mask，然后用它来生成图像。基于定性结果，自我注意模型也能很好地约束物体的形状。

## 模型：

* ![image-20220422162745513](./SegAttnGAN%20Text%20to%20Image%20Generation%20with%20Segmentation%20Attention_img/image-20220422162745513.png)
*  和AttnGAN一样，SegAttnGAN使用词注意模型，并且使用LSTM文本编码器来提取单词特征和句子特则会那个，句子特征用随机潜在向量连接，词特征作为词级注意。

### 分割注意力模块：

* 该模块通过保持输入语义图的空间约束来增强图像合成，其被定义为：

  > $F^` = BN(F)*Conv(S)+Conv(S)$

  这里的BN为批处理归一化函数，Conv()为卷积函数，F为来自前一层的特征，S为输入分割映射。该函数的核心是保留分割掩码的空间信息。它与超分任务中的注意模块非常相似。该模型通过将语义映射注意力从粗到细的策略引入到每个上采样层中，有望避免纯上采样层所消除的语义。

### 分割掩模策略：

- 对于分割掩膜在应用中有不同的策略，也即是两个不个不同的模型：
  - SegAttnGAN，使用**数据集中已有的掩码**作为注意力输入。
  - self attention SegAttnGAN使用自我注意发生器生成的掩码。

## 实验结果：

* ![image-20220422171146126](./SegAttnGAN%20Text%20to%20Image%20Generation%20with%20Segmentation%20Attention_img/image-20220422171146126.png)
* ![image-20220422171237847](./SegAttnGAN%20Text%20to%20Image%20Generation%20with%20Segmentation%20Attention_img/image-20220422171237847.png)
* 可以看到与基线模型AttnGAN相比，SegAttnGAN生成的结果具有更好的对象形状

## 结论：

* 就实验结果来说，与AttnGAN的效果其实相差不大，甚至比起MirrorGAN来说要略低一些。个人认为，总的来说，这篇论文在针对生成图像中图像的形状确实有一些提升，能够更好的解决形状形变的问题。
* 但是其效果并不是很显著，只不过在T2I任务中的切入点多了一个。

