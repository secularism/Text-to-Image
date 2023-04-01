# Semantics Disentangling for Text-to-Image Generation

## 作者：Guojun Yin，Bin Liu，Lu Sheng，Nenghai Yu，Xiaogang Wang，Jing Shao 时间：2019

## 会议：CVPR

## 为什么提出SD-GAN：

* 对于T2I来说，以往文本-图像生成工作主要关注于通过从粗到细的叠加生成器结构或注意引导生成程序来提高生成图像的视觉质量和分辨率，但是这些方法忽略了一个重要的现象，即人类对同一幅图像的描述在表达上具有高度的主观性和多样性（也就是每个人对于同一幅图像进行描述会带着自己的主观意识，虽然描述的是同一个东西，但是文字的描述会有一定偏差），这意味着天真地使用这些文本作为独特的描述来生成图像，往往回产生不稳定的外观模式，与groundtruth图像相去甚远。换句话说说即使描述的是同一事物，不同的语言表达也给提取一致的语义带来了挑战。不同的描述可能回导致图像产生偏差。
* ![image-20220322223646905](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220322223646905.png)
* 上图表示为不同的语言表达但相同的语义产生了不同的图像，甚至是不相同的鸟类。

## SD-GAN的介绍和贡献：

#### *为了解决上述的问题，作者们提出了一种新的真实感文本到图像的生成方法，这个方法在生成过程中有效地利用了输入文本中的语义，即为SD-GAN*

### 1.介绍：

* SD-GAN可以从文本中提取语义共性，以保证图像生成的一致性，同时保留细粒度图像生成的语义多样性和细节，受Siamese结构在不同任务中的启发，可以找到一对序列之间的相似性，作者将鉴别器作为图像比较器，在图像描述全面且指向相同语义内容的前提下，保持图像之前的语义一致性。
* ![image-20220322225014075](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220322225014075.png)
* 也就是说SD-GAN使用一个暹罗方案，以一对文本作为输入，并使用上图所示的对比损耗进行训练。intra-class表示为具有不同描述的**相同ground truth image**，inter-class表示为具有不同描述的**不同ground truth image**，而通过SD-GAN，具有相似语言与义的intra-class在鉴别器特征空间内的距离要小得多（因为描述的是同一幅图像），而inter-class在鉴别器特征空间内的距离要大得多（因为描述的是不同的图像）。因此通过这样的结构，可以使得文本到图像生成器从多种语言表达中提炼出固有的语义共性。（可以理解为，通过一对文本的输入，通过loss，会使得同一幅图像的不同文字生成相似的图片，而不同的图像，则会生成区别较大的图片）。
* 上述提出的连体结构确实能够从文本中提炼出语义共性，但是即使是同一幅图像，因为描述人的主观性，我们在关注语义的共性时，还要保持文本和文本之间的语义差异（也就是每个文本的多样性），因此需要在视觉生成中嵌入详细的语义线索（linguistic cues），作者们在生成器中重新制定批处理规范层——SCBN，它能使详细和细粒度的语言嵌入能够操纵生成网络中的视觉特征地图。

### 2.贡献：

* 所提出的SD-GAN能从语言描述中提取语义公域，在此基础上生成的图像在表达变量下保持生成的一致性。（按作者说，这是第一次将暹罗机制引入到跨模态生成中）。
* 为了弥补暹罗机制可能失去的独特语义多样性，设计了一种增强的视觉语义嵌入方法，利用实例语言线索重新构造批处理规范层——SCBN，可以保留文本的语义多样性和细节。
* 所提出的SD-GAN在CUB-200 bird数据集和MS-COCO数据据上实现了最先进的文本-图像生成性能。

## 相关介绍：

### 1.暹罗结构（Siamese Network）：

* 简单来说Siamese Network就是“连体的神经网络”，神经网络的“连体”是通过共享权值来实现的。![img](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/v2-5070e28622a2f3ee9e3cb5d2259fae86_720w.jpg)
* 左右两两边的网络可以是同一个网络，比如CNN，也可以不一样，一边是LSTM，一边是CNN。当两个网络不共享权值叫做pseudo-siamese network（伪孪生神经网络）。
* 用途：简单说，是为了衡量两个输入的相似程度，将两个输入feed进入两个神经网络，然后两个神经网络分别将输入映射到新的空间，形成输入在新的空间中的表示，并通过Loss的计算，评价两个输入的相似度。
* Siamese Network用于处理两个输入**"比较类似"**的情况。Pseudo-Siamese Network适用于处理两个输入**"有一定差别"**的情况。比如，我们要计算两个句子或者词汇的语义相似度，使用siamese network比较适合；如果验证标题与正文的描述是否一致（标题和正文长度差别很大），或者文字是否描述了一幅图片（一个是图片，一个是文字），就应该使用pseudo-siamese network。也就是说，要根据具体的应用，判断应该使用哪一种结构，哪一种Loss。

### 2.CBN（Conditional Batch Normalization）：

* BN（批量归一化）是对每一个小批量进行归一化激活，该方法可以通过减少网络中协变量的移位俩加速训练和提高泛化。而CBN一种条件归一化层，其利用条件线索学习调制参数。我们知道传统BN的公式为：

  > $BN(x)=\frac{x-\mu(x)}{\sqrt{\sigma(x)+\epsilon}}*\gamma+\beta$

  其中$\gamma$和$\beta$都是网络层的参数，需要通过损失函数反向传播来学习。

  而CBN公式为

  > $BN(x|c)=\frac{x-\mu(x)}{\sqrt{\sigma(x)+\epsilon}}*(\gamma+\gamma_c)+(\beta+\beta_c)$

  这里的$\gamma_c$和$\beta_c$是把特征feature输入到一个MLP，前向传播得到的网络输出（$\gamma_c$和$\beta_c$取决于输入的feature，这样就可以通过调节输入的feature来达到影响BN），由于$\gamma_c$和$\beta_c$依赖于输入的feature这个condition，因此这个改进版的BN叫做CBN。

* CBN最先来自Modulating early visual processing by language这篇文章，他改进了一个基于图片的问答系统VQA。![image-20220323105454259](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220323105454259.png)
* 系统的输入为一张图片和一个针对图片的问题，系统则输出问题的答案。左边是通常的做法，而右边是该文作者们的做法。我们发现，LSTM提取文本的特征只在ResNet的顶层才和图片特征结合起来（因为神经网络的底层提取的是最基础的几何特征，顶层是具有含义的语义特征，因此应该把语言模型提取的句子特征在网络顶层和图片相结合）。但是作者认为，底层的图片特征也应该结合语言特征。**理由是，神经科学证明：语言会帮助图片识别。例如，如果事先告诉一个人关于图片的内容，然后再让他看图片，那么这个人识别图片的速度会大大加快。**因此，作者首创了将图片底层信息和语言信息结合的模型，如上图右侧所示。
* 而结合的方法则是想通过修改$\gamma$和$\beta$的方式，达到有针对性地提取图片部分信息的目的，将文本经过MLP得到新的$\gamma_{pred}$和$\beta_{pred}$作为一个小的增量加在原来BN层的参数上，也就得到了CBN的公式。这样的想法，在图像的底层feature中结合了自然语言信息，取得了很好的表现。

## 网络结构;

* ![image-20220323110242270](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220323110242270.png)
* 该结构实在AttnGAN基础上进行修改的，其中主干结构使用的是顺序堆叠的生成器-鉴别器模块，它是由Text Encoder和Hierarchical Generative Adversarial Networks两部分组成。

### 1. Text Encoder：

* 每个分支的输入都是一个自然语言描述的句子。文本呢编码器的目标是从自然语言描述中学习特征表示，这里采用的是BiLSTM（双向长短期记忆），使用$w_t$表示第$t^{th}$个单词的特征向量，$\overline{s}$表示句子特征向量。

### 2. Hierarchical Generative Adversarial Networks：

* ![image-20220323111223200](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220323111223200.png)
* 给定$\overline{s}$和噪音$z$进行图像的生成，(a)图代表第一阶段的初始图像，(b)图代表下一阶段利用前一阶段的输出和sentence feature生成分辨率更高的图像，这里要注意的是**SCBN在每个上采样的末端工作**。

## Contrastive Loss：

* 所提出的Siamese结构目的是在训练过程中，无论输入描述的表达式的变化，增强生成的一致性，如果从两个分支生成的视觉特征是文本语义感知的，那么生成的两个图像应该是相似的(即有较小的距离)——intra class。否则，两个生成的图像应该是不同的(即有很大的距离)——inter class。为此，我们采用对比损耗法从输入的描述对中提取语义信息：

  > $L_c=\frac{1}{2N}\sum_{n=1}^{N}y\cdot d^2+(1-y)max(\epsilon-d,0)^2$

  其中$d=||v_1-v_2||_2$为视觉特征向量$v_1$和$v_2$到两个Siamese分支的距离，$y$为标记输入描述是否来自同一幅图像的标识，1表示相同，0表示不同。$N$为特征向量的长度$\epsilon$用来平衡$y=0$时的距离。

* 在对比损失的情况下，通过使生成的图像于同一图像描述之间的距离最小化和使不同图像描述之间的距离最大化来优化暹罗结构。但由于输入噪声，即使描述完全一致，生成的图像可能在外观上不同，比如姿势，背景等等。因此为了避免两个语义一致的文本在生成图像上使得两幅图像完全一样（要使上面的$L_c$变小，$d$会越来越小，很有可能趋近于0，这就代表$v_1$和$v_2$一致，两幅图像一样，我们需要避免这样无意义的模式，可以理解为对应上面的保留语义多样性），因此作者将上述公式修改为：

  >$L_c=\frac{1}{2N}\sum_{n=1}^{N}y\cdot max(d,\alpha)^2+(1-y)max(\epsilon-d,0)^2$

  这里的$\alpha$是一个超参数，主要目的是防止$d$变得太小，以至于两个图像一样，在实验中$\alpha$设置为0.1

## SCBN:

* SCBN的目的是加强生成网络特征映射中的视觉语义嵌入。它使语言嵌入能够通过方法或缩小、否定或关闭视觉特征图等方式（CBN的作用，将语义作为condition，称为SCBN）来操纵视觉特征图。其关注文本中独特的语义差异。SCBN的语义线索可以从两个方面获得：句子层面和词层面。
* ![image-20220323194534048](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220323194534048.png)

### 1.Sentence-level Cues（句子层面的线索）：

* 如上图（a）所示，采用一隐层的MLP从输入描述的sentence feature中踢球调制参数$\gamma_c$和$\beta_c$（这里只给出$\gamma_c$的事例，$\beta_c$一样）：

  > $\gamma_c=f_{\gamma}(\overline{s}),\beta_c=f_{\beta}(\overline{s})$

  上式$f_{\gamma}(\cdot)$和$f_{\beta}(\cdot)$分别表示$\gamma_c$和$\beta_c$的单隐层MLPs。然后将$f_{\gamma}(\overline{s})$和$f_{\beta}(\overline{s})$的维数扩展到与$x$相同的大小，以便利使用CBN的公式嵌入语言线索和视觉特征。

### 2.Word-level Cues（单词层面的线索）：

* 如上图（b）所示，$W=\{w_t\}_{t=1}^T\in R^{D \times T}$表示为词组的特征，图中的$X\in R^{C \times L}$是视觉特征，$C$是通道尺寸并且$L=W \times L$。这个VSE模块采用相互融合的特征和视觉特征，其公式为：

  > vse$_j=\sum_{t=0}^{T-1}\sigma(v_j^T \cdot f(w_t))f(w_t)$

  先使用一个感知层也就是$f(w_t)$来匹配文本特征和视觉特征的维度，然后根据图像的嵌入特征$v_j$，对图像的每一个子区域$j$计算VSE向量vse$_j$，它是与视觉特征$v_j$相关的词向量$\{w_t\}_{t=1}^T$的动态表示。$\sigma(v_j^T \cdot f(w_t)$)表示为视觉特征映射的第$j^{th}$个子区域$v_j$和第$t^{th}$个词向量$w_t$的**视觉语义嵌入权值**，这类似于互相关的点积相似度。

## 实验结果：

* ![image-20220323202525239](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220323202525239.png)
* ![image-20220323202538187](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220323202538187.png)
* ![image-20220323202543266](./Semantics%20Disentangling%20for%20Text-to-Image%20Generation_img/image-20220323202543266.png)
* 可以看出SD-GAN在相同的参数下，性能更好。