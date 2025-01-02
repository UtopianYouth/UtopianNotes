## 基于ML和实时反馈控制辅助半导体表面脱氧

##### 专业术语

1. ###### 什么是epi-ready surface？		
2. ###### epilayers？

##### 问题

1. ###### 监测Si(111)脱氧过程，并且通过Si与**特定表面重构相似性**进行分类？
2. ###### 模型训练数据：大量的GaAs脱氧过程中的RHEED视频流序列 + 少量的Ge和InAs脱氧视频流序列
3. ###### 模型主要预测的一件事情：通过衬底脱氧状态来预测衬底此时的温度。
4. ###### 衬底的多样性和温度的不确定性，对脱氧的过程都会造成影响，脱氧的过程会对衬底的性能有影响，所以较为考验工艺工程师的经验。

##### 实验部分

###### 1、Riber 32P MBE system

###### 2、RHEED 12, from STAIB

###### 3、对RHEED视频序列图像处理的过程：设置曝光时间和帧采样速率 → 将采集到的RHEED图像按时间顺序整理好 → 进行亮度的归一化处理（灰度化）→ 将一系列二维图像构造成三维序列图像 → 将三维序列图像转换为一维图像序列 → 输入模型进行训练

###### 4、

##### 深度学习部分

###### 1、Interpolation-ViT模型

###### 2、Attention heatmap

&emsp;&emsp;注意力机制热图，在具有注意力机制的模型中，承担了对模型输入数据提取更加重要的部分，提高模型的性能和效果

###### 3、t-distributed Stochastic Neighbor Embedding（t-SNE）算法

&emsp;&emsp;和主成分分析（PCA）算法类似，都是对数据进行降维的方法。

###### 4、the activation value of feature map

&emsp;&emsp;特征图是张量（图片高度，宽度和颜色通道）

###### 5、convolutional kernel and convolutional layer



##### 讨论部分

###### 1、Riber C21 MBE System （30rpm/8fps）

###### 2、
