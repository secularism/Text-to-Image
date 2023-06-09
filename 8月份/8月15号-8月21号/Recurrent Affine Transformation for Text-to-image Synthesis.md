# Recurrent Affine Transformation for Text-to-image Synthesis

## 作者 Senmao Ye ，Fei Liu，Minkui Tan 时间2022

## 期刊：CVPR

## 为什么提出RAT-GAN：

- 现有方法通常使用多个独立的融合块（例如，条件批量归一化和实例归一化）将合适的文本信息自适应第融合到合成过程中。然而，孤立的融合块不仅相互冲突，而且增加了训练的难度，因为它们彼此不相互作用。

## 解决方法：

- 为此，提出了RAT-GAN，使用一种递归仿射变换（RAT），可以一致地控制所有融合块。
- RAT使用相同形状的标准上下文向量表达不同层的输出，以实现对不同层的统一控制。然后使用递归神经网络（RNN）连接上下文向量，以检测长期相关性。通过在RNN中跳过连接，融合块不仅在相邻块之间保持一致，而且降低了训练难度。
- 另外，为了提高文本和图像之间的语义一致性，作者在鉴别器中加入了空间注意力模型，由于知道匹配的图像区域，文本描述监督生成器合成更多相关的图像内容。**但是基于广泛的实验，作者发现，使用softmax函数的空间注意力会导致模型崩溃，因为softmax函数会抑制大多数概率接近零，这反过来会导致GAN的不稳定性增加**。为了克服这一问题，我们使用软阈值函数代替softmax函数，从而防止小的注意力概率接近零。通过空间注意力，鉴别器可以将注意力集中在与文本描述相关的图像区域，从而更有效地监督生成器。

## 贡献点：

- 提出了一种循环仿射变换，将所有融合块连接起来，以便在合成过程中全局分配文本信息。
- 将空间注意力纳入鉴别器，以关注相关图像区域，因此生成的图像与文本描述更相关
- RAT-GAN改善了CUB、Oxford-102和COCO数据集的视觉质量和评估指标。

## RAT-GAN：

- ![image-20220901191803650](./Recurrent%20Affine%20Transformation%20for%20Text-to-image%20Synthesis_img/image-20220901191803650.png)