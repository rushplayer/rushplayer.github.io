---
layout: mypost
title: RushPlayer“一键下马”系列之-JavPlayer
categories: [技术]
---
个人公号：RushPlayer  
个人网站：https://rushplayer.github.io/

为了RushPlayer的“一键下马”功能，我做了不少研究，其中很多是用GAN（Generative Adversarial Networks）来消除视频中的马赛克。
在后续介绍RushPlayer算法文章里，我会对具体细节做讲解。先简单介绍一些国外同行们的研究成果。
 
也许只有在日本成熟的商业场景，会产生一个完整的打码除码产业链，算法研究者才有变现动机去实现。首先介绍的JAVplayer便来自日本，它有一个完整的播放器界面，所有运算都在本地完成。因为直接打包了tensorflow、caffe深度学习框架，加上没有轻量化的AI图像生成网络，软件比较臃肿。但好处是AI计算很方便地调用Cuda加速。
 
![0.png](https://ftp.bmp.ovh/imgs/2020/04/c4029c8b0acaf74c.png)
<center>原图</center>

![0-mosaic.png](https://ftp.bmp.ovh/imgs/2020/04/e67ba1aef9ee9746.png)
<center>有码</center>

顺手拿手头一部测试片子，把码上移到脸上（爱学习的同学已经认出水野老师了吗？）这么做有考虑：1，为方便审核分享，而且人脸打马赛克也是常见场景。2，人脸焦距、角度、光照、环境都有变化，配合厚薄适中的马赛克，更好测试算法实际性能。

### 马赛克检测
去马赛克先是要找到马赛克，找到马赛克先要知道什么是马赛克，这是一个很哲学的逻辑环。先介绍马赛克，马赛克就将图像内S x S 方块内所有位置的像素值用块内第一个像素值代替，很绕的解释。检测马赛克，可以用传统CV算法：sobel算子、canny边缘检测、膨胀等算法，找出水平和垂直的直线边缘，算出S 的大小，精确找出马赛克的位置。后续的不管是模糊和锐化算法、AI图像生成网络的MASK、后处理泊松融合算法的边缘等都依赖这一步，其重要性甚至专门训练一个AI检测网络来检测马赛克都不为过。

![cut75_005.png](https://i.loli.net/2020/04/06/vFdwmpRCnT6sISl.png)
<center>马赛克检测</center>
JavPlayer的马赛克检测性能一般，Rushplayer的算法会要求马赛克检测更准一点。


### 马赛克消除
JavPlayer默认提供了一种简单的去马赛克实现：先模糊再锐化。效果一般，仅适用于薄码。
![0-normal.png](https://ftp.bmp.ovh/imgs/2020/04/55b7c5c0a98f1201.png)
<center>简单去马赛克</center>

JavPlayer接入了deepcreampy工程。这是对于Nvidia论文《Inpainting for Irregular Holes Using Partial Convolutions》的复现，它需要用户指定的Mask范围，区别于其他算法的正方形，mask可以是不规则的形状。它提出了部分卷积层（Partial Convolutional Layer），包括 mask 和重新标准化卷积操作以及后续的 mask 更新（mask-update）。可自动为下一层生成更新的 mask作为前向传递的一部分。个人不喜欢这种花式卷积操作，不好优化。效果如下：

![0-deepcreamppy.png](https://ftp.bmp.ovh/imgs/2020/04/d051eed1b34ffa79.png)
<center>DeepCreampy去马赛克</center>

JavPlayer接入Waifu工程，引入了额外一个大网络的中间特征来perceptual loss，架构几年前也属sota的。

JavPlayer主要推介的是tecogan。它有几个特点：1，提出首个时空判别器，以获得逼真和连贯的视频超分辨率；2，提出新型 Ping-Pong 损失，以解决循环伪影；3，从空间细节和时间连贯度方面进行详细的评估；4，提出新型评估指标，基于动态估计和感知距离来量化时间连贯度。简单说就是参考到了视频前后几帧的相关信息，生成网络的输入需要连续5帧图像。

![0-tg-normal.png](https://ftp.bmp.ovh/imgs/2020/04/f36b5e89d1cefe21.png)
<center>TecoGan去马赛克</center>

![0-tg-waifu.png](https://ftp.bmp.ovh/imgs/2020/04/7f5cf03394b5c0c7.png)
<center>TecoGan+waifu去马赛克</center>

貌似tecogan + waifu细节丰富但噪点略多。各种算法打码后的效果完整视频显示：
https://youtu.be/nL71CgE5bzw

[TOC]

