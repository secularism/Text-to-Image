# CPGAN: Content-Parsing Generative Adversarial Networks for Text-to-Image Synthesis
## 作者：Jiadong Liang，Wenjie Pei，Feng Lu 时间：2019年

## 期刊：CVPR

## 为什么提出CPGAN：

### 1.以往方法的不足之处

* 以往的方法在GANs网络的基础上虽然已经取得了实质性的进展，但一个潜在的限制是，这些方法试图再生成过程中直接对文本到图像的映射进行建模，这对于这种跨模态翻译来说相当困难。比如AttnGAN和StackGAN都很难将单词“sheep”对应于绵羊的完整视觉图片。如下图：
* ![image-20220511151148597](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511151148597.png)
* 比如，在上图中，“sheep”这样的单词，以往的模型并不会将其完整的对应在生成图片的部分，也许sheep这个单词还对应着图片其他部分，如草地天空这样的像素点上，并不是完整对应，这样一来生成出来的图片质量以及与文本的一致性就比较差。
* 因此在语义层面更明确地建模文本到图像的一致性是可行的，但这需要对文本和图像模式有透彻的理解。然而，之前的方法很少关注输入文本或生成图像的语义分析内容。而19年提出的SDGAN研究了这一限制，它利用鉴别器中的暹罗结构来学习两个文本描述之间的语义一致性。但是，对于输入文本和生成的图像，在语义层面上直接面向内容的解析并没有深入执行。因此提出CPGAN用于解决这一问题。

### 2.CPGAN的提出有哪些创新点：

* 在该文中，作者们致力于对输入文本和合成图像的内容进行彻底解析，从而对它们之间的语义对应关系进行建模。其中共有三个方面的创新点：
  * 在文本模态方面，设计了一种记忆机制，通过捕捉词汇表中每个单词的训练数据中相关图像的各种视觉上下文信息来解析文本内容。
  * 在图像形态方面，作者们建议以对象感知的方式对生成的图像进行编码，以提取视觉语义。然后利用得到的文本embedding和图像embedding来度量文本图像在语义空间中的一致性。
  * 此外，还设计了一个条件鉴别器，通过对单词和图像子区域之间的细粒度相关性进行局部建模来推动语义文本图像对齐。
* 正因如此，通过生成的模型执行全局内容解析，以更好地在语义上对齐输入文本和生成的图像，从而提高文本到图像合成的性能。

## CPGAN：

* ![image-20220511161700578](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511161700578.png)

* 分为三个结构：

  1）Memory-Attended Text Encoder：利用记忆结构来探索单词与其各种视觉语境之间的语义对应关系

  2）Object-Aware Image Encoder用于在语义层解析生成的图像

  3）Fine-grained Conditional Discriminator用于度量输入文本和生成图像之间的一致性，以指导整个模型的优化。

* 在结构图中，和AttnGAN模型一样的是采用从粗到细的框架来生成高质量的图像，区别在于引入了额外的剩余连接从$C_0$到$C_1$和$C_2$（通过向上采样），以缓解generator之间的信息传播。CPGAN使用两种不同类型的损失函数对整个模型进行联合优化：1）生成性对抗损失$D_i^{uc}$，通过训练对抗性鉴别器使生成的图像更真实，同时匹配描述性文本；2）文本-图像语义一致性损失$D_i^c$，鼓励文本-图像在语义层面上对齐。

  ![image-20220511163156798](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511163156798.png)

### 1.Memory-Attended Text Encoder：

#### 1.1 介绍

*  

  > 这里视觉上下文信息可以理解为与图像相关的部分，比如要搜索到自行车这个图像，我们可以联系自行车上下文相关的信息，比如轮胎，马路，进而通过这个去搜寻自行车在图像中的所处位置，而一个单词对应训练数据中的多个相关图像和有多个视觉上下文信息**个人理解是一个单词在不同的语境上有多个意思，比如spring既有春天的意思也有温泉的含义**

* 典型的文本编码方法在训练过程中对文本进行在线编码，**只能关注当前训练对的文本-图像对应关系**（也就是说忽略了其他和当前训练对的单词有相关的图像）。该文基于记忆的文本编码器**旨在从训练数据中的所有相关图像中捕获单词与各种视觉上下文之间的完整语义对应关系**。因此，模型可以对词汇表中的每个单词进行更全面的理解，并合成质量更高、更多样的图像。 

#### 1.2 结构：

* ![image-20220511164452484](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511164452484.png)
* Memory 结构被构造成一个映射结构，其中每个项目将一个单词映射到其视觉上下文表示。但是需要先找到给定单词的每个相关图像，再从中学习到有意义的视觉特征，比如上图中单词为bicycle，找到所有自行车图片后，需要从中学习关于自行车的视觉特征。因此需要在每个图像中检测单词的显著区域，并从中提取特征。该文通过预训练模型Yolo-V3对图像进行对象检测，然后选择顶部36个子区域（Image N这一列），但实际上，作者们只保留了每个相关图像中最显著的子区域的视觉特征（在Image N中被虚线圈起来的，其Attention weight为红色圆圈的）。拿到该单词在每个图像中最显著的视觉特征后，对他们进行加权平均和。
* ![image-20220511171023208](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511171023208.png)
* 这里的N是训练数据中与第r个单词相关的图像的数量。$a_{i,n}$是第n个相关图像的第i个子区域上的注意权重，$q_n$则是取到权重最大的一个。通过这种记忆机制解析视觉特征的好处有两个：**1）从每个单词的相关图像的最显著区域提取精确的语义特征；2） 捕捉单词与其各种视觉语境之间的完整语义对应关系。**
* **Text Encoding with Memory**，除了从视觉上下文信息中解析文本的学习记忆外，CPGAN还通过学习词汇中每个单词的潜在嵌入来对文本进行编码，以表征所有单词之间的语义距离。具体来说，目标就是学习embedding矩阵E，其由K个单词的d维embedding 向量组成。然后，词汇表中第i个单词学习单词嵌入$e_i=E[:,i]$，通过连接与学习记忆$m_i$融合$f_i=[e_i;p(m_i)]$，这里的$p(m_i)$是一个非线性投影函数，用于平衡$m_i$和$e_i$之间的特征维度。如图中右边部分所示，在该图中通过两个全连接来执行p，中间有一个LeakReLU层。最后给定包含T个单词的文本描述X，使用BiLSTM结构来获得每个时间步的最终单词嵌入，它包含单词之间的时间依赖关系：$W,s=BiLSTM(f_1,f_2,...,f_T)$

### 2.Object-Aware Image Encoder

* 该模块在语义层面上对生成的图像进行解析。通过最小化输入文本和合成图像之间的语义差异，为提出的TISCL（Text-Image Semantic
  Consistency Loss）准备获得的图像编码特征，以指导整个模型的优化。

  ![image-20220511185528591](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511185528591.png)

* 因此，解析图像特征的质量对我们模型的图像合成性能至关重要。除了学习整个图像的全局特征外，处理局部图像特征的典型方法是从等分图像子区域中提取特征，作者建议在对象级别解析图像，以提取更多具有物理意义的特征，这里同样使用Yolo-V3来检测具有最高目标检测置信度的显著边界盒。另外，作者还结合了从等分子区域中提取的局部特征，然后，通过线性变换将两种提取的特征$V_o$和$V_e$投影到具有相同维度的潜在空间中，并将它们连接在一起，以导出最终的图像编码特征$V_c$： 

  ![image-20220511190042170](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511190042170.png)

  得到的$V_c$进一步输入到DAMSM中，通过注意力机制测量输入文本的每个单词与图像不同子区域的最大语义一致性，计算TISCL具体表现为模型图的最右边部分。

  ![image-20220511190235627](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511190235627.png)

### 3.Fine-grained Conditional Discriminator：

* 该鉴别器用于区分文本标题是否与图像成对匹配，从而通过相应的对抗性丢失来推动合成图像和输入文本之间的语义对齐。设计条件鉴别器的**典型方法是分别从文本和图像中提取嵌入的特征，然后直接在聚集的特征上训练鉴别器。**这种方法的一个**潜在限制是，只考虑文本和图像之间的全局兼容性，而不探讨文本中的单词和图像子区域之间的局部相关性。**然而，图像和标题之间最显著的相关性总是在局部反映出来。因此该鉴别器侧重于对图像和标题之间的局部相关性进行建模。
* 收到PatchGAN的启发，CPGAN将图像划分为NxN个patches，并且为每个patches提取视觉特征。然后通过关注文本中的每个单词，从每个补丁的文本中学习上下文特征，如下图。
* ![image-20220511192059930](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511192059930.png)
* ![image-20220511192205470](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511192205470.png)
* 这里的$a_n$是文本中第n个单词的注意权重。将获得的上下文特征$p_{i，j}$以及句子embedding连接在一起，以区分真假

## 实验：

* ![image-20220511192548080](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511192548080.png)
* ![image-20220511192659832](D:\workplace\note\5月份\5月2号-5月8号\CPGAN Content-Parsing Generative Adversarial Networks for Text-to-ImageSynthesis_img\image-20220511192659832.png)
* 可以看到在Inception score上CPGAN有巨大的效果，并且其他指标也有不错的效果，参数量比起AttnGAN也少了很多