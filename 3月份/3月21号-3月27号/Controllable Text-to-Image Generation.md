# Controllable Text-to-Image Generation

## 作者：Bowen Li，Xiaojuan Qi，Thomas Lukasiewicz，Philip H. S. Torr 时间：2019

## 会议/期刊：NeurIPS

## 为什么提出ControlGAN：

* 由于GAN在生成真实感图像方面的成功，文本到图像的生成在实现条件生成网络（Conditional GAN）的基础上取得了显著的进展。然而，**目前的生成网络一般是不可控的**，这意味着如果用户改变句子中的某些词，合成的图像就会与原始文本生成的图像有很大的不同，如下图所示;
* ![image-20220326172012635](./Controllable%20Text-to-Image%20Generation_img/image-20220326172012635.png)
* 在这里。当给定的文本描述(如颜色)发生变化时，鸟的相应视觉属性会被修改，但其他不相关的属性(如姿势和位置)也会发生变化。而在真实的应用程序中，当用户希望进一步修改合成图像以满足自己的偏好时，通常不希望出现这种情况。
* 因此该文的目标是**从文本生成图像，并允许用户在一个框架中使用自然语言描述来操作合成图像。特别是，我们专注于通过改变给定的文本描述来修改生成图像中对象的视觉属性(例如，类别、纹理和颜色)。**为此，提出了一种新型的可控文本-图像生成网络——ControlGAN。该网络既可以合成高质量的图像，又允许用户在不影响其他内容生成的情况下操纵对象的属性。

## ControlGAN的组成模块和结构：

### 1.组成模块：

* 所提出的ControlGAN既能有效地合成高质量的图像，又能根据自然语言描述控制部分图像生成。该网络包含三个新组件：word-level spatial and channel-wise attention-driven generator（字级空间和通道级注意力驱动生成器）、word-level discriminator（词级鉴别器）以及在生成器中采用的（perceptual loss）感知损失。为此，作者们进行了广泛的分析，表明了该方法可以有效地分离不同的属性，并在不丧失多样性的情况下准确地操作合成图像的部分。

#### 1.1 word-level spatial and channel-wise attention-driven generator：

* 利用注意力机制，让生成器合成与最相关的单词对应的子区域，该生成器基于AttnGAN，遵循多阶段架构，从粗到细合成图像，并逐步提高质量。

#### 1.2 word-level discriminator：

* 该模块通过研究词与图像子区域之间的相关性来分解不同的视觉属性，为生成器提供与视觉属性相关的细粒度训练信号。

#### 1.3 perceptual loss：

* 可以减少生成过程中的随机性，并强制生成器保留未修改文本的相关的视觉外观。

### 2.结构：

* ![image-20220326174320382](./Controllable%20Text-to-Image%20Generation_img/image-20220326174320382.png)
* 该网络的目标是合成一个在语义上与$S$一致的真实图像$I$，并且使得这个生成过程可控。也就是说如果将$S$修改为$S_m$，则合成的结果在语义上与$S_m$匹配，同时保留$I$中存在的不相关内容。
* 可以看出，在输入端和之前的网络类似，给出一个$S$，通过Text Encoder——一个预先训练的双向RNN，将句子编码成sentence feature（ $s \in R^D$，$D$维度是描述整个句子）和word feature（$w \in R^{D \times L}$，$L$是单词的数量），然后将$s$通过CA模块得到$\tilde{s}$，然后与随机向量$z$连接，作为第一阶段的输入。整体框架在多个阶段生成从粗到细的图像，在每个阶段，**网络生成一个隐藏的视觉特征$v_i$，它是输入到相应的生成器$G_i$，以产生合成图像。可以看到spatial attention（空间注意力机制）和channel-wiseattention（通道式注意模块）以$w$和$v_i$为输入，输出的是注意的词-上下文特征。然后这些值得注意的特征将进一步与隐藏的特征$v_i$连接在一起，作为下一阶段的输入。
* 值得一说的是，在AttnGAN中也采用了spatial attention，但是在没有channel-wise attention的情况下，spatial attention只能将单词与单个空间位置关联起来。因此引入channel-wise attention来开发单词和通道之间的联系。这样的做法发现，channel方向的注意模块将语义意义部分与相应的词高度相关，而spatial方向的注意模块则将注意力集中在颜色描述上。因此，二者一起可以帮助生成器解开不同的视觉属性，并允许它只关注最相关的子区域和通道。

### 2.1 Channel-Wise Attention：

* ![image-20220326193843956](./Controllable%20Text-to-Image%20Generation_img/image-20220326193843956.png)

* 在第$k$个阶段，将$w$和$v_k$进行输入（图中的$H_k$和$W_k$代表第$k$阶段map feature的高度和宽度），词特征$w$先通过感知层$F_k$映射到与$v_k$相同的语义空间中得到$\tilde{w_k}$，然后通过矩阵乘法（MatMul）将两者进行结合得到$m^k$，这样的一来，**$m^k$聚集了所有空间位置上的通道与字之间的相关值**，再将其通过softmax层后与词特征的转置进行相乘得到最后的结果$f_k^a$，$f_k^a$中的每个通道都是一个动态的表示，通过单词与视觉特征中对应通道之间的相关性加权得到。**这样一来，相关值高的通道对对应的词的响应就会高，这样既可以将词的属性分解到不同的通道中，又可以通过赋予较低的相关性来减少不相关通道的影响。**在图中$a^k$的公式为：

  > $a_{i,j}^k=\frac{exp(m_{i,k}^k)}{\sum_{l=0}^{L-1}exp(m_{i,l}^k)}$

  $a^k$代表注意力权值，它表示视觉特征$v_k$中的第$i$个通道与句子S中的第$j$个单词的相关性，值越大，相关性越强。

### 2.2 Word-Level Discriminator：

* ![image-20220326195204649](./Controllable%20Text-to-Image%20Generation_img/image-20220326195204649.png)

* Word-Level Discriminator也是有两个输入：1）word feature $w$，这里的$w^`$从Text Encoder编码得来，$w$和$w^`$分别表示从原始text $S$编码得到的单词特征和随机采样的不匹配文本。2）视觉特征$n_{real}$、$n_{fake}$，它们都是通过基于GoogleNet图像编码器从真实图像$I$和生成的图像$I^`$编码得来的。（文中为了简化，将$w$和$w^`$统一表示为$w$，$n_{real}$、$n_{fake}$统一表示为$n$）。图中的左侧操作与channel-wise attention的处理大致相同，其中$\beta_{i,j}$表示第$i$个单词与图像的第$j^{th}$子区域的相关值。最后得到的$b$称为图像的子区域感知的词特征，它集合了所有空间信息，其权重由词上下文相关矩阵$\beta$加权。

  > $b=\tilde{n}\beta^T$

  此外，为了进一步减少不太重要的词的负面影响，作者在这里还采用了Word-Level Self Attention（来自于Text-adaptive generative adversarial networks: manipulatingimages with natural language这篇文章，word-level discriminator的灵感也是来源这一篇论文）推导出一个长度为$L$的一维向量$\gamma$来反映每个词的相对重要性。然后通过$\tilde{b}=\bigodot\gamma^`$得到$\tilde{b}$，这里的${\bigodot}$代表element-wise multiplication。最后通过公式得到$r_i$。其评估第$i$个单词和图像之间的相关性。

  > $r_i=\sigma(\frac{(\tilde{b_i}^Tw_i)}{||\tilde{b_i}||\;||w_i||})$

  然后，图像$I$和句子$S$之间的最终相关值$L_{corre}$是通过求和所有的word-context相关来计算的，通过这样做，生成器可以从字级标识符接收每个视觉属性的细粒度反馈，这可以进一步帮助监督每个子区域的生成和操作。

### 2.3 Perceptual Loss:

* 如果不对与文本无关的区域（背景等）添加任何约束，生成的结果可能是高度随机的，并且可能在其他内容在语义上不一致。为了减少这种随机性，作者采用了**基于16层VGG网络的感知损失**，该网络用于从生成的图像$I^`$和真实图像$I$提取语义特征。

  > $L_{per}(I^`,I)=\frac{1}{C_iH_iW_i}||\phi_i(I^`)-\phi_i(I)||_2^2$

  其中$\phi_i(I)$为VGG网络中第$I$层的激活，通过匹配特征空间，可以减少图像生成过程中的随机性。

## 补充：

### spatial attention：

* ![image-20220326203744871](./Controllable%20Text-to-Image%20Generation_img/image-20220326203744871.png)
* 简而言之就是将所有的通道进行求和，然后reshape, softmax, reshape获得注意力矩阵。



