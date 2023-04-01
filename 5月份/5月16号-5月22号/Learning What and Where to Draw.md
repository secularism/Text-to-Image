# Learning What and Where to Draw

## 作者：Scott Reed、Zeynep Akata、Santosh Mohan、Samuel Tenka、Bernt Schiele、Honglak Lee 时间：2016

## 期刊：NIPS 

## 介绍：

- 提出了一种新的模型，GAWWN，它根据描述在哪个位置绘制什么内容的指令，合成图像。通过分离“what”和“where”的问题，可以利用这一事实在计算的每一步修改图像。除了参数效率之外，这还带来了更多可解释的图像样本的好处，因为我们可以跟踪网络在每个位置的意图。
- ![image-20220616153139206](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220616153139206.png)

## 主要贡献：

- （1）一种用于文本和位置可控的图像合成的新体系结构，产生更真实和更高分辨率的CUB样本；
- （2）一种文本条件对象部分完成模型，能够简化指定部分位置的用户界面；
- （3）探索了新的结果和一种用于pose-conditional 文本到人类图像的数据集

## 模型介绍：

### 1.Bounding-box-conditional text-to-image model：

- ![image-20220616154345427](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220616154345427.png)
- 首先，在空间上文本embedding被复制嵌入以形成MxMxT的特征映射，然后再空间上扭曲以适应规范化的而边界框坐标。注意的是这里的边界框外面都是0。然后与噪声向量连接起来。
- 模型的中的生成器分为本地和全局处理阶段。全局路径只是一系列步长为2的反卷积，用以将特征的空间维度从$1\times1$增加到$M \times M$。在局部路径中，当维度达到$M \times M$时，将应用掩蔽操作（就是将对象边界框外的区域设置为0）。最后通过depth concat将两者连接起来，然后通过反卷积达到图片空间的尺寸。
- 在鉴别器中，文本先是在空间上被复制为维度$M \times M \times T$的张量。同时图像通过局部和全局途径进行处理。与生成器类似的是局部路径中采用步长为2的卷积。然后将得到的特征与文本进行拼接，并最后通过卷积后与全局路径得到的特征进行组合（全局路径简单地由下至向量的卷积组成，并加入了文本embedding）。在最后一层中，作者应用Tanh函数来约束输出[-1,1]，以生成标量鉴别器得分。

### 2.Keypoint-conditional text-to-image model:

- ![image-20220617195725724](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220617195725724.png)
- location keypoints被编码成$M \times M \times K$的空间图中，其中通道对应着编码的part，比如图像中的头部在通道1，图像中的左腿在通道2。keypoints tensor被feed进网络的几个阶段。

### 3.Conditional keypoint generation model：

- 作者提出从用户体验的角度来看，如果在文本生成图片这个过程中要求用户输入他们希望绘制的对象部分的每个keypoints（比如指出绘制鸟的喙的位置），这不是很好的选择。因此，如果在给定观察到的关键点子集和文本描述的情况下，能够访问到未观察到的关键点的所有条件分布将是非常有用的。

- 作者用通用GAN框架来解决这个问题，基本思想是使用观察到的或未观察到的每个对象部分的assignment来作为gating mechanism。将单个图像的keypoints表示未$k_i:=\{x_i,y_i,v_i\}$。其中$x$和$y$分别表示行和列的位置。如果part可见，则$v$位设置为1，否则为0，为0时，$x$和$y$也设置为0。然后让$K \in [0,1]^{K \times 3}$将keypoints编码成矩阵。将conditioning variable（比如例如用户指定的喙位置）编码到switch unit的向量中$s \in \{0,1\}^K$，如果第$i$部分时条件变量，则第$i$项设置为1.否则设置为0。因此可以在文本和keypoints的子集条件下，在keypoints$G_K$上建立生成器网络，如下所示：

  ![image-20220617204502624](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220617204502624.png)

- 这里的$f:R^{Z+T+3K} \rightarrow R^{3K}$是MLP。在代码中，作者将z，t和flatten k连接起来，并选择f作为三层全连接层。鉴别器$D_k$从合成图像中学习区分真实keypoints和文本。

## 实验：

### 1.Controlling bird location via bounding boxes：

- ![image-20220617205253621](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220617205253621.png)

- 可以看到噪声向量在每一组三帧中固定时，背景通常是相似的，但不是完全不变的。另外，当边界框坐标改变时，鸟面对的方向不会改变。这辨明该模型学习使用噪声分布来捕获背景的某些方面。以及“何处”的不可控方面，如方向。

### 2.Controlling individual part locations via keypoints：

- ![image-20220617205622054](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220617205622054.png)
- 其中蓝色点是固定的keypoints，每个样本采用不同的随机噪声。
- 可以观察到，鸟类姿势和keypoints关系密切，并且在整个样本中保持不变。背景和其他小细节（例如树枝的厚度或背景调色板）确实会随噪声而变化。
- ![image-20220618134253073](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220618134253073.png)
- 与边界框的情况不同，现在可以控制鸟指向的方向（这里是固定鸟喙和尾巴的位置，所有鸟都面向左侧），但是当使用边界框时，方向是随机的。另外，在可控位置之外，场景的元素也会进行调整，以便与每一帧中鸟的位置保持一致。

### 3.Generating both bird keypoints and images from text alone：

- ![image-20220618134555984](D:\workplace\note\5月份\5月16号-5月22号\Learning What and Where to Draw_img\image-20220618134555984.png)
- 前面提到，ground-truth keypoints会导致视觉上合理的结果，但是keypoints的获取成本很高。在上图中，作者们提供了使用**生成的keypoints**的示例，与ground-truth keypoints相比，图像的质量没有显著下降