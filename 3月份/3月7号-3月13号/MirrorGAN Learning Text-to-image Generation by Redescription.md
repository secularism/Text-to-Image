# MirrorGAN: Learning Text-to-image Generation by Redescription

## 作者：Tingting Qiao，Jing Zhang，Duanqing Xu1，and Dacheng Tao 时间：2019

## 期刊/会议：ICPR

## 摘要：

* 文中指出根据给定的文本描述生成图像有两个目标：视觉真实感和语义一致性。在利用GAN生成高质量、视觉逼真的图像方面已经取得了很大的进展，但要保证文本描述和视觉内容之间的语义一致性仍然非常具有挑战性。
* 作者提出了一种新的全局局部关注和语义保持的文本-图像-文本框架MirrorGAN来解决这个问题。MirrorGAN利用了通过重描述学习文本到图像生成的思想，有三个模块组成：
  * 语义文本嵌入模块（STEM）- 生成word feature和sentence feature
  * 用于级联图像生成的全局-本地协同注意模块（GLAM）- 具有从粗到细尺度生成目标图像的级联结构，利用局部单词注意力和全局句子注意力，逐步增强生成图像的多样性和语义一致性
  * 语义文本再生和对齐模块（STREAM）- 主要是寻求从生成的图像中重新生成文本描述，它在语义上与给定的文本描述一致。

##  为什么要提出MirrorGAN：

* 文中指出虽然使用GAN在该方面已经取得了重大进展，但保证生成图像与输入文本的语义对齐仍然具有挑战性，对于最近提出的AttnGAN来说，虽然它引导生成器在生成不同图像区域时聚焦于不同的单词，但是作者认为因为文本和图像模式之间的差异，仅使用单词级别的注意并不能确保全局语义的一致性。
* T2I其实也可以认为是I2T的问题，（I2T表示在给定的图像的情况下生成文本描述），因此作者考虑到处理每个任务都需要建模和对齐两个领域的底层语义，所以很自然的想到在一个统一的框架中建模两个任务——也就是最后提出了MirrorGAN的新型文本-图像-文本的框架来改进T2I的生成，其利用了通过重新描述来学习T2I生成的思想。

##  主要贡献：

* 提出了一个新的统一框架——MirrorGAN，其用于T2I和I2T的建模，具体针对T2I生成的思想。
* 提出了一个全局-局部协作注意模型，该模型无缝嵌入级联生成器中，以保持跨领域语义的一致性，并平滑生成过程。
* 提出了一种基于ce的文本语义重建损失，以监督生成器生成视觉逼真和语义一致的图像。

## 网络结构：

* ![image-20220312151355648](D:\workplace\note\数字水印论文\笔记\3月份\3月7号-3月13号\MirrorGAN Learning Text-to-image Generation by Redescription_img\image-20220312151355648.png)

* 如上图所示，从左到右依次是STEM、GLAM和STREAM。

  * 在STEM中，将给定的文本描述嵌入到局部词级特征和全局句子特征中；使用RNN网络从给定的文本描述T中提取语义嵌入，包含由词向量word feature和句子向量sentence feature。

    > ${w,s}$ = ${RNN(T)}$
    >
    > 这里的${T=\{T_l|l =0,...,L-1\}}$，${L}$表示句子长度。${w=\{w^l|l=0,...,L-1\}\in R^{D \times L}}$是每个单词的隐藏状态${w^l}$的拼接，${s \in R^D}$是最后一个隐藏状态，${D}$是${w^l}$和${s}$的维数。由于文本域2的多样性，排列较少的文本可能具有相似的语义。因此作者提出使用条件增强的方法来增强文本描述的做法，这产生了更多的图像-文本对，从而鼓励了对沿着条件文本流形的小扰动的鲁棒性。  

  * 在GLAM（Global-Local collaborative Attentive Module in Cascaded Image Generators）通过将三个图像生成网络按顺序叠加，构造多级级联生成器，这里采用的是AttnGAN描述的基本结构，因为它在生成真实感图像方面有很好的性能，注意在图中的这部分实质上是有多个图像生成器${G}$，也就是该生成了多个图像，下图这部分重复了多次。

    ![image-20220312154003886](D:\workplace\note\数字水印论文\笔记\3月份\3月7号-3月13号\MirrorGAN Learning Text-to-image Generation by Redescription_img\image-20220312154003886.png)
    * 在数学上，使用${F_0,F_1,...,F_{m-1}}$表示${m}$个视觉特征变压器，${G_0,G_1,...,G_{m-1}}$表示${m}$个图像生成器。每个阶段的视觉特征${f_i}$和生成的图像${I_i}$可以表示为：

      > ${f_0=F_0(z,s_{ca})},$
      >
      > $f_i=F_i(f_i,F_{att_i}(f_{i-1},w,s_{ca})),i\in \{1,2,...,m-1\},$
      >
      > $I_i=G_i(f_i),i\in \{1,2,...,m-1\},$

    

    * $F_{att_i}$是提出的全局-局部协同注意模型，该模型包含两个分量$Att_{i-1}^w$和$Att_{i-1}^s$

      > $F_{att_i}(f_{i-1},w,s_{ca})=concat(Att_{i-1}^w,Att_{i-1}^s)$

      

  * STREAM（Semantic Text REgeneration and Alignment Module），其用于从生成的图像中重新生成文本描述，该描述在语义上与给定的文本描述对齐。具体来说，作者们采用了一个广泛使用的基于编码器解码器的图像标题框架作为基本的STREAM架构。**在这里，作者指出还可以使用更高级的image captioning model，这也许会产生更好的结果。**图像编码器实在ImageNet上与训练的卷积神经网络（CNN），解码器是RNN。由末级发生器生成的图像$I_{m-1}$送入CNN编码器和RNN解码器。

    > $x_{-1}=CNN(I_{m-1})$
    >
    > $x_t=W_eT_t,t \in \{0,...,L-1\}$
    >
    > $p_{t+1}=RNN(x_t),t \in\{0,...,L-1\}$
    

    * 这里$x_{-1} \in R^{M_{m-1}}$是一个视觉特征，用作开始时的输入，告知RNN图像内容。$W_e \in R^{M_{m-1} \times D}$表示一个词嵌入矩阵，该矩阵将词特征映射到视觉特征空间。$p_{t+1}$时单词的预测概率分布

## 实验结果：

* ![image-20220312200032067](D:\workplace\note\数字水印论文\笔记\3月份\3月7号-3月13号\MirrorGAN Learning Text-to-image Generation by Redescription_img\image-20220312200032067.png)

* 从上图可以看出MirrorGAN所生成的图像比起AttnGAN来说更接近真实图像

* 在CUB和COCO数据集上：![image-20220312200156526](D:\workplace\note\数字水印论文\笔记\3月份\3月7号-3月13号\MirrorGAN Learning Text-to-image Generation by Redescription_img\image-20220312200156526.png)

  