# DM-GAN: Dynamic Memory Generative Adversarial Networks for Text-to-Image Synthesis

## 作者：Minfeng Zhu，Pingbo Pan，Wei Chen，Yi Yang 时间：2019

## 会议/期刊：19年被CVPR（IEEE Conference on Computer Vision and Pattern Recognition）会议收录

## 以往的方法存在什么问题：

* 该文提出在以往的方法中，很大程度上依赖初始图像的质量（这里初始图像的质量自己理解为与文本对应的程度）。如果初始图像初始化不好，后续处理很难将图像细化到令人满意的质量
* 在描述不同的图像内容时，每个词的重要性程度不同，但现有的图像精华过程中使用的是不变的文本表示。也就是说文本描述中的每个单词对图片的内容的作用都不一样，重要性也不一样，他们都有不同程度的信息描述图像内容。但是目前的模型在不同的图像细化过程中使用相同的词表示，这使得细化过程无效。要考虑图像信息，确定每个词的重要性进行细化。

## DM-GAN提出的方法以及贡献：

### 1.提出的方法：

* 对于第一个问题：该文建议添加一个内存机制来处理生成糟糕的初始图像，（受Memory Networks. In ICLR, 2015的启发）作者提出了在GAN框架中增加键值存储结构。将初始图像的模糊图像特征作为查询，从存储模块中读取特征。对内存的读取用于优化初始图像。
* 对于第二个问题：引入了一个内存写入们来动态地选择与生成图像相关的单词。这使得生成的图像以文本描述为条件。因此，在每个图像细化过程中，根据初始图像和文本信息动态地写入和读取存储组件。此外，该算法不直接连接图像和内存，而是使用响应门自适应地接受图像和内存中的信息。

### 2. 贡献：

* 提出了一种新颖的GAN模型与动态记忆组件相结合，以生成高质量的图像。即使初始图像没有很好地生成。
* 介绍了一种记忆写入门，它可以根据初始图像选择相关的单词
* 提出了一种自适应融合图像和记忆信息的响应门

## Memory Networks：

* Memoty Networks提供了一种新的架构，可以使用显示存储和注意力概念更有效地从记忆中推理答案。Memoty Networks首先将信息写入外部内存，然后根据相关概率从内存插槽读取内容了，在此基础上，16年在ACL提出的Key-value memory Networks通过对键存储和值存储使用不同的编码进行推理，在预测最终答案时，键内存用于推断对应值内存的权重。受此网络的启发，作者们提出了DM-GAN，通过键和值记忆之间的非平凡转换生成高质量的图像。
* Key-value memory Networks的结构图为：![网络结构](D:\workplace\note\数字水印论文\笔记\3月份\3月14号-3月20号\DM-GAN Dynamic Memory Generative Adversarial Networks for Text-to-Image Synthesis_img\7ae697b045a56df4814da3bfbc993266.png)

## 网络模型：

* ![image-20220316225555954](D:\workplace\note\数字水印论文\笔记\3月份\3月14号-3月20号\DM-GAN Dynamic Memory Generative Adversarial Networks for Text-to-Image Synthesis_img\image-20220316225555954.png)
* 该模型结构由两个阶段组成：初始图像生成和基于动态内存的图像细化。
  * 在图像生成的初始阶段，首先通过文本编码器将输入的文本描述转换为sentence feature和word feature，然后通过CA模块（流形插值+加入随机噪声）和生成器部分（包括有全连接、上采样），生成图像特征$R_0$，最后由卷积生成初始图像$x_0$，可以看到初始图像的形状较为粗糙，细节较少。
  * 在图像细化阶段：主要是针对初始图像添加更多的细粒度视觉内容，生成真实感的图像特征。这里的细化阶段可以重复多次，图中的$R_{i-1}$为最后阶段的图像特征，并由此生成更细粒度细节的高分辨图像。

## Dynamic Memory:

* 基于动态内存的图像细化阶段由四个部分组成：内存写入、键寻址值读取和响应。

  * 内存写入操作将文本信息存储到一个键值结构的内存中，以便进一步检索。
  * 在上一步之后，采用键寻址和值读取操作从记忆模块中读取特征，对低质量图像的视觉特征进行细化。
  * 最后，采用Response操作来控制图像特征的融合和内存的读取

* 在Dynamic memory中先将上一步得到的初始图像特征$R_0$和word feature进行输入。其中由;

  > $W=\{w_1,w_2,...,w_T\},w_i \in R^{N_w}$
  >
  > $R_i=\{r_1,r_2,...,r_N\},r_i \in R^{N_r}$

  这里的$T$是words的数量，$N_w$为单词特征的维数，$N$为图像像素数，图像像素特征为$N_r$维向量

### 1.Memory Writing：

* 记忆写入就是编码先验知识，记忆写入将文本特征经过一次$1*1$卷积运算嵌入到$n$维的记忆特征空间中，这里$M(\cdot)$代表$1*1$卷积运算。

  > $m_i=M(w_i),m_i \in R^{N_m}$

### 2.Key Addressing：

* 这一步中，使用键存储器检索相关的存储器。我们计算每个内存插槽的权重作为内存插槽$m_i$和图像特征$r_i$

  > $\alpha_{i,j}=\frac{exp(\phi_K(m_i)^Tr_j)}{\sum_{l=1}^Texp(\phi_k(m_l)^Tr_j)}$

* 这里$\alpha_{i,j}$维第$i$个记忆体与第$j$个图像特征之间的相似概率，$\phi_K()$为将记内存特征特征映射到维数$N_r$的关键记忆体访问过程，$\phi_k()$实现为$1\times1$卷积。

### 3.Value Reading：

* 输出存储器表示被定义为根据相似概率对值存储器进行加权求和：

  > $o_j=\sum_{i=1}^T \alpha_{i,j}\phi_V(m_i)$

* 这里$\phi_V()$是将内存特征映射到维度$N_r$的值内存访问过程，$\phi_V()$实现为$1 \times 1$的卷积

### 4.Response：

* 在接收到输出存储器后，将当前图像和输出表示结合起来，以提供一个新的图像特征。一种简单的方法是将图像特征和输出表示连接起来。方法为：

  > $r_i^{new}=[o_i,r_i]$

* 这里的$[\cdot,\cdot]$为串联操作。然后，我们可以利用一个上采样块和几个剩余块将新的图像特征升级为高分辨率图像。上采样块由最近邻居上采样层和$3 \times 3$卷积组成。最后，利用$3 \times 3$卷积从新的图像特征中得到精制图像。

##  Gated Memory Wiriting（内存写入门）;

* ![image-20220317131229354](D:\workplace\note\数字水印论文\笔记\3月份\3月14号-3月20号\DM-GAN Dynamic Memory Generative Adversarial Networks for Text-to-Image Synthesis_img\image-20220317131229354.png)

* 内存写入门允许DM-GAN模型选择相关的单词来细化初始图像，它将最后阶段的图像特征$R_i$与单词特征$W$相结合，计算出单词的重要性

  > $g_i^w(R,w_i)=\sigma(A*w_i+B*\frac{1}{N}\sum_{i=1}^Nr_i)$

* 这里$\sigma$代表sigmoid函数，$A$是一个$1\times N_w$矩阵，$B$是一个$1 \times N_r$矩阵，然后结合图像和文字特征编写内存插槽$m_i \in R^{N_m}$

  > $$m_i=M_w(w_i)*g_i^w+M_r(\frac{1}{N}\sum_{i=1}^Nr_i)*(1-g_i^w)$$

* 其中$M_w(\cdot)$和$M_r(\cdot)$代表$1 \times 1$卷积运算。它们将图像特征和单词特征嵌入到相同的$N_m$尺度的特征空间上。

* **也就是说，内存写入门最终要做的事情是将img feature和word feature拼接起来进行输入，既是这里的$$m$$实际上包含着上一次的图像特征和词特征，上面公式中$i$代表的是第几次做这样的运算，第一次的图像特征是第一步的初始图像。计算出来的$g^w$是一个在0-1之间的值**

## Key-Value：

* ![image-20220317133232288](D:\workplace\note\数字水印论文\笔记\3月份\3月14号-3月20号\DM-GAN Dynamic Memory Generative Adversarial Networks for Text-to-Image Synthesis_img\image-20220317133232288.png)

* 在理解响应门之前，我们先对这部分做一个解释，当上一步中的内存写入门得到$m$后，其通过dynamic memory得到$\phi_V(m)$和$\phi_K(m)$，从上面我们知道这两个运算都是将内存特征映射到维度$N_r$的值内存访问过程，其中先通过Key Addressing操作，得到$\alpha_{i,j}$得到记忆体$m$和图像$R$之间的相似概率（也就是weight），**这里的相似概率也可以理解为相关性，得到的$\alpha$是多个，其代表每个记忆体和每个图像特征之间的相关性，概率越大，相关性也越强。**再通过Value Reading操作进行加权求和，注意此时得到是多个$o_j$，其中$j$代表的是第j个图像特征。最后再将输出的向量$o$送入到响应门。**这一步的操作其实是参考了Key-value memory Networks论文的做法， 只不过Key-value memory Networks在这一步就完成了一个hop**，在本文中，作者将其改进为继续输出，在最后得到新的图像特征后，才算作是一个hop。

## Gated Response（响应门）：

* ![image-20220317135917409](D:\workplace\note\数字水印论文\笔记\3月份\3月14号-3月20号\DM-GAN Dynamic Memory Generative Adversarial Networks for Text-to-Image Synthesis_img\image-20220317135917409.png)

* 这一步操作的目的就显而易见了——更新图像

  > $g_i^r=\sigma(W[o_i,r_i]+b)$
  >
  > $r_i^{new}=o_i*g_i^r+r_i*(1-g_i^r)$

* 根据响应门最后输出$R^{new}$新的图像特征，它可以自适应融合从记忆中读取的信息和图像特征，使得能够从文本描述中准确地生成图像。在此之后，通过上采样和残差块处理，得到图像特征（文中提到因为GPU的原因，只精华了两次），最终通过卷积层将图像特征生成高质量的图像。

## 实验：

* ![image-20220317141155684](D:\workplace\note\数字水印论文\笔记\3月份\3月14号-3月20号\DM-GAN Dynamic Memory Generative Adversarial Networks for Text-to-Image Synthesis_img\image-20220317141155684.png)
* 值得一提的是，在同一年提出的MirrorGAN，DM-GAN在coco数据集上的效果优于MirrorGAN。