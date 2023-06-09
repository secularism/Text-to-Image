﻿# DR-GAN Distribution Regularization for Text-to-Image Generation

# 一、摘要

> 这篇 paper 提出了一个新的文本生成图像模型-分布正则化生成网络（DR-GAN）。在 DR-GAN 中，包含了两个新的模块：语义解开模块（SDM）和分布正则化模块（DNM）。SDM 包含空间自注意机制和一个新的语义解开损失（SDL），SDL 能帮助生成器在图片生成阶段提取关键的语义信息。DNM 使用变分自动编码（VAE）对图像潜在分布正则化和去噪，则还能帮助生成器更好的区分生成图像和真实图像。DNM 也采用了分布生成损失（DAL）去指导生成器通过正则化去对齐潜在空间中真实图像的分布。

---

# 二、为什么提出 DR-GAN

- 许多先进的 T2I 方法一般是先提取文本特征，然后使用 GAN 网络去生成对应的图像。他们的本质是将文本特征的分布映射到图像分布上，但是有两个因素阻止了基于 GAN 的 T2I 方法去捕获真实图像的分布
  - 文本描述的抽象性和多样性使得生成器从图像生成中捕获关键的语义信息是很困难的
  - 视觉信息的多样性使得图像分布复杂化以至于基于 GAN 的 T2I 方法难以从文本特征分布中捕获真实图像分布

# 三、对应解决方法

- 针对第一个问题，作者第一个策略是在模型中间部分的特征中设计一个信息解离机制，这会更好的在执行跨模态分部学习之前提取关键信息
- 针对第二个问题，作者设计一个有效的分布正则化策略去正则化图像的潜在分布。这个机制目标在于帮助判别器更好的学习生成图像和真实图像之间的分布决策边界

# 四、所做的贡献

总的来说、主要的贡献有三点

- 拟定了一个语义解离模块（SDM）去帮助生成器从图像和文本特征中提取关键信息（并且过滤掉非关键信息）
- 设计了一个新的分布正则化模块（DNM），它在基于 GAN 的 T2I 模型中引入了 VAE 以至于它能对图像潜在的分布进行更有效的正则化和去噪
- 通过四个指标，广泛的实验和分析展示了 DR-GAN 在两个基准上（COCO 和 CUB）的功效

# 五、模型架构

![在这里插入图片描述](https://img-blog.csdnimg.cn/e5626fcefad34e9d89c09b6621c4ad11.png)

- 和 AttnGAN 类似，文本通过编码器得到句子特征和单词特征，句子特征通过条件增强模块后结合噪声进入 m 个生成器得到最终生成图像，生成阶段的信息流表述为：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/d56bfe62dbe64df1832359ec70a343bd.png)
- 需要注意的是这里的$SDM_0$仅仅包含一系列的卷积层和上采样模块。所拟定的语义解离模块（SDM）在$SDM_i$中被采用，$i=1,...,m-1$

## 5.1SDM

![在这里插入图片描述](https://img-blog.csdnimg.cn/960228e4d8de48d89f14d70aeafffe55.png)SDM 是基于 AttnGAN 中的级联注意生成模型（CAGM）所构建的，因为 CAGM 的词级注意机制（WAM）可以有效地增强生成图像的语义细节。 **最初的 SDM 是被设计为直接从文本特征中提取关键信息和非关键信息**。但因为文本和图像属于异构域，在特征空间中很难实现合理的语义匹配。与文本特征相比，**WAM 的词级上下文特征包含更多的结构和语义信息，在语义上更好地匹配图像特征**。此外，单词级上下文特征和图像特征的语义来源于输入的单词特征 W 和句子特征 s。因此，基于 CAGM 的单词级上下文特征和图像特征，**设计 SDM 来提取关键信息和过滤非关键信息。**

### 5.1.1 获得$Q^{'}_i$

首先单词向量$W$和前一层的隐藏特征$H_{i-1}$被输入到 WAM 中得到包含上下文信息的词级特征$Q^{'}_i$，此时的$Q_i$是图像和文本特征的加权组合，它可以有效地丰富图像细节的语义。但由于文本描述的抽象性和模糊性，生成器容易解析错误或不准确的语义，这会导致$Q_i$和$H$中的语义和结构不正确。因此需要从这两者中提取关键信息。

### 5.1.2 获得关键和非关键信息

作者使用自注意机制来表示两者中关键信息和非关键信息。为了基于$Q^{'}_i$必要的空间结构信息，使用 ResBlock 细化该特征得到$Q_i$。然后将$Q_i$和$H_{i-1}$分别送入 self-attention 中得到相对应的空间掩码信息$Mask^Q_i$和$Mask^H_i$，再使用$Q_i$和$H_{i-1}$与掩码信息做$\odot$，得到$Q^+_i$和$H^+_i$，这两者代表关键信息。做相减得到$Q^-_i$和$H^-_i$代表非关键信息。

### 5.1.3 SDL 损失

为了驱动 SDM 更好地区分$Q_i$和$H_i$中关键信息和非关键信息，使用语义解离损失 SDL 去解决这个问题。**在生成任务中，假设生成的图像分布与真实图像分布为相同类型的分布。如果两个分布的均值和方差相同，则两个分布是相同的。**因此作者使用均值和方差来约束关键信息和非关键信息。其中 SDL 损失表示为：![在这里插入图片描述](https://img-blog.csdnimg.cn/9805ad4830b34bf9ac47bfdd403088f7.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/99c49977092d40c6bb5fd481fdb5b443.png)
这里${\mu}({\cdot})$和${\sigma}({\cdot})$分别是获取特征的均值和方差，$SP(x)=ln(1+e^x)$，另外这里的$H^*_i$代表对应实像$I^*_i$的特征映射。最后将关键特进行拼接、融合上采样得到下一阶段的特征映射$H_i$

## 5.2、DNM

![在这里插入图片描述](https://img-blog.csdnimg.cn/9e05b63ffb284bea9fb3348d077c8c2e.png)

上述的 SDM 能帮助生成器获得生成所需要的关键语义信息。但是在鉴别器方面，面对复杂多样的图像分布使得区分真实图像和生成图像更加困难。**我们知道数据的归一化能够降低数据的噪声和内部协变量移位，这会进一步提高深度学习社区的流形学习效率**。因此归一化是一种有效的去噪和降低复杂性的策略。

### 5.2.1 VAE

VAE 可以有效地去噪潜在分布，降低它的复杂性。基于 VAE 重建图像的优势，归一化嵌入向量可以保留关键语义视觉信息。因此，作者在 DNM 中构建了一个 VAE 模块来归一化图像的潜在分布，以帮助鉴别器更好地区分“真”图像和“假”图像。

### 5.2.2 过程

- 整个 DNM 包含两个子模块：鉴别器和 VAE 模块，鉴别器由编码器和逻辑分类器组成，用于判别图像是真还是假。但是基于上述理论，判别器直接判别带有其他非关键信息的图像会使最终的判断变得困难和复杂。整个 VAE 由编码器、变分采样模块和解码器组成。
- $x_i$通过编码器得到向量$v_i$，然后$v_i$通过变分采样模块得到向量的均值和方差，并使用 KL 散度对其做正则化得到$z^*_i$。$z^*_i$和句子向量结合后送入解码器中

### DAL 损失

根据 VAE 的下变分解界，VAE 在 DNM 的损失为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d5334171b00c465485830cfdedd2cbad.png)
另外(1)生成器的训练步骤中，对 VAE 模块中生成的图像分布进行归一化。(2)将归一化生成分布与归一化实像分布对齐。为了实现上述两个目标，分布一致性损失被定义为：![在这里插入图片描述](https://img-blog.csdnimg.cn/937cca9ec36e4b1ea2c54ebb991d4b38.png)

## 六、实验

数据集：COCO 和 CUB
评价指标： 使用 IS 来测试图像的多样性、使用 FID 和 MS 来测试分布一致性、使用 R 分数和 H.P.来测试语义一致性
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b42ef61a8954a83a4c9c53db83f42ea.png)

## ![在这里插入图片描述](https://img-blog.csdnimg.cn/6e297fe135ab44bfb8b5d8c23461cd89.png)

# 总结

- 作者提出了一个新的模型、使用 SDM 和 DNM 模块可以高效的生成高质量图像
- 使用 VAE 来正则化生成图像分布、并采用 DAL 损失使生成图像分布接近真实图像分布
- 作者使用自注意机制获取特征信息中的关键信息，并将非关键信息和关键信息计算损失，使生成器能更好的捕捉关键信息去生成图像
- 但是使用 DR-GAN 也难以避免生成的图像存在纹理不合理等问题，这可能是在特征信息中缺乏了布局和结构信息，导致某些特征在构建时，所对应的对象被随意放置
