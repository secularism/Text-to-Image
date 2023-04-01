# AttnGAN: Fine-Grained Text to Image Generation with Attentional Generative Adversarial Networks

## 作者：Tao Xu，Pengchuan Zhang，Qiuyuan Huang，Han Zhang，Zhe Gan，Xiaolei Huang，Xiaodong He 时间：2018

## 期刊/会议：2018

## 摘要：

* 之前基于gan的文本生成图像的模型通常都是将整句文本描述编码成单个向量作为生成图像的条件（比如StackGAN和Stack++），这样做缺乏细粒度的字级信息。

* 因此，在该文中，作者们主要提出了一个注意力生成对抗网络，它允许注意力驱动的、多阶段的细化文本到图像的生成。利用一种新的注意生成网络，AttnGAN可以通过关注自然语言描述中的相关词，在图像的不同子区域合成细粒度的细节。此外，提出了一种深度注意多模态相似度模型来计算细粒度的图像-文本匹配损失，用于训练生成器。提出的AttnGAN显著优于以前的技术水平。

## 核心思想：

* 该文的创新主要在两大组成部分：注意力生成网络和DAMSM
  * 注意力生成网络：生成网络中的引入的注意机制使AttnGAN能够在单词水平上实现单词与图片中的某个子区域的映射，自动选择字级条件以生成图像的不同子区域。
  * DAMSM：能够计算细粒度文本图像匹配损失。

## 网络架构：

* ![image-20220309125928871](D:\workplace\note\数字水印论文\笔记\3月份\3月7号-3月13号\AttnGAN Fine-Grained Text to Image Generation with Attentional Generative Adversarial Networks_img\image-20220309125928871.png)
* 图中黄色区域既是注意生成网络，它有m个生成器（G<sub>0</sub>、G<sub>1</sub>、G<sub>2</sub>、...、G<sub>m-1</sub>），它接受隐藏状态（${h}$<sub>0</sub>、${h}$<sub>1</sub>、${h}$<sub>2</sub>、...、${h}$<sub>m-1</sub>），生成小到大尺度的图像（${x}$<sub>0</sub>、${x}$<sub>1</sub>、${x}$<sub>2</sub>、...、${x}$<sub>m-1</sub>），公式为：
  * ![image-20220309130817716](D:\workplace\note\数字水印论文\笔记\3月份\3月7号-3月13号\AttnGAN Fine-Grained Text to Image Generation with Attentional Generative Adversarial Networks_img\image-20220309130817716.png)
  * ${z}$在公式中表示为一个噪声向量，通常从标准正态分布中采样。$\overline{e}$是全局的句子向量，${e}$是单词向量的矩阵，${F}$<sup>${ca}$</sup>表示条件扩充，它将句子向量${e}$转换为条件向量，${F}$<sub>${i}$</sub><sup>${attn}$</sup>是在AttnGAN第${i}$阶段提出的注意模型。
  * ${F}$<sup>${attn}$</sup>(${e,h}$)有两个输入：单词特征${e}$和来自前一个隐含层的图像特征${h}$。首先通过增加一个新的感知器层，将单词特征转化为图像特征的公共语义空间，然后，根据图像的隐藏特征${h}$为图像的每个子区域计算单词的上下文向量。${h}$的每一列是图像中的一个子区域的特征向量。对于第${j}$个子区域，其词上下文向量是与${h}$<sub>${j}$</sub>相关的词向量的动态表示。
* 左边的Text Encoder（LSTM）和右边的Image Encoder组合成为DAMSM，它将句子的图像和单词的子区域映射到一个共同的语义空间，从而在单词水平上度量图像-文本的相似度，计算出用于生成图像的细粒度损失。
  * Text Encoder利用注意力机制对文本进行编码，输出sentence feature和word features。在这里sentence feature取LSTM最后一个状态的输出，作用是当作生成器的控制信息；word feature是取中间隐藏状态的输出，用来确定图片与句子的一致性。
  * 图像编码器采用CNN将图像映射到语义向量。
* 下方的判别器（带有D的），输入的是sentence feature和该阶段生成器生成的图片，判断图片与句子的相符性。