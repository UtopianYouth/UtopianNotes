## 基于机器学习的QD外延生长反馈控制框架

###### 原文链接：https://mp.weixin.qq.com/s/YTVo45HU5SAI1z4ekzjkwQ

###### 论文1原文：https://doi.org/10.1038/s41467-024-47087-w

###### 论文2原文：https://doi.org/10.1021/acsami.4c01765

###### 论文标题1: Machine-learning-assisted and real-time-feedback-controlled growth of InAs/GaAs quantum dots

###### 论文标题2: Universal Deoxidation of Semiconductor Substrates Assisted by Machine Learning and Real-Time Feedback Control

##### 一、基本问题

###### 为什么说识别结果不会由于经验误差和设备稳定性等干扰而发生波动？

###### 量子点？

###### 机器学习残差模型？

###### 原子力显微镜获得的QDs(Quantum Dots)密度信息？

###### 上采样插值的视觉变换器模型？

###### 图1 讨论（基于机器学习的QD外延生长反馈控制框架）

![通过机器学习模型（RHEED视频序列信息）控制衬底温度，进一步控制QD外延生长](https://mmbiz.qpic.cn/mmbiz_png/Iu0iblfbxvlFia180Fppib8KNtTo4dY6BibVns7J5GeIsSnwZ4Bt3ElZbRQ8AXyRquFqokb7ODpDVIg2ibDw2MNibnDg/640?wx_fmt=png&from=appmsg&wxfrom=13&tp=wxpic)

###### 图2 讨论（机器学习辅助InAs/GaAs量子点原位生长过程**温度**的实时反馈控制，应用机器学习的残差模型）

![针对图1的更加细化表现](https://mmbiz.qpic.cn/mmbiz_png/Iu0iblfbxvlFia180Fppib8KNtTo4dY6BibV8TPQhO9xmNTznDEfSKZdsbicGU7V7mMcuXD2HdGWXff0f4AdnyicnP2g/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

&emsp;&emsp;结合**RHEED表面再构时序视频**与**原子力显微镜获得的QDs密度信息**相互关联，实现**对衬底温度的原位反馈控制**。

###### 图3 讨论（机器学习辅助衬底表面氧化物的去除，应用机器学习上采样插值的视觉变换器模型）

![机器学习辅助衬底脱氧数据的基本框架](https://mmbiz.qpic.cn/mmbiz_png/Iu0iblfbxvlFia180Fppib8KNtTo4dY6BibVIAECatyE6RWVkZGpujPTr2Hugwfj5fkFRm9djaCibcPuoV63LBicE4Ug/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)



&emsp;&emsp;利用机器学习监测**衬底温度**，进行实时反馈，来辅助半导体表面**脱氧的过程**。



# 文章大致结构

##### 前言

Stranski-Krastanow (SK)模型

量子点生长和普通的材料生长

原位表征

原位控制



挑战：

提取的RHEED信息里面有噪声和重叠的图像；

一个单一的RHEED图像和衬底会形成一个固定的角度；



##### 样品结构与数据标签



##### 程序框架和RHEED视频流处理

借助常规的图像处理技术

##### ML模型构建和评估

主要讲述了ResNet模型通过已经处理好的视频流图像作为训练集，将训练好的模型对RHEED图像进行预测，两种模型，量子点模型和密度模型，分别通过图像预测是否有量子点生长和量子点密度范围。

##### 低密度量子点控制生长
