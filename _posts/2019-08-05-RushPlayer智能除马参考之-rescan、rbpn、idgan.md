---
layout: post
title: "RushPlayer智能除马参考之-rescan、rbpn、idgan"
date: 2019-08-05
author:     "run2rush"
header-img: "img/post-bg-rwd.jpg"
categories: 技术
tags: 
    - 算法
---
个人公号：RushPlayer  
个人网站：https://rushplayer.github.io/

&emsp;&emsp;因为做一个视频图像修复技术的关系，参考了一些论文，看到一个单张图像将RNN和CNN结合去雨技术，感觉比较有新意。  
&emsp;&emsp;RNN和CNN结合在视频领域比较常见，在单幅图像应用较古典是描述语句生成，CNN做特征提取后作为RNN的hidden state，输入start后RNN网络输出描述语句。  
&emsp;&emsp;网络架构如图，主体是正常的encoder-decoder 生成网络，其中的Dialated Conv现在已是常规操作。  
<img src="https://i.loli.net/2019/08/05/M3vz7ITLfBy6mwr.jpg" width="400px" />

&emsp;&emsp;我归纳了几点:  
&emsp;&emsp;1.操作很骚，这个channel weight SE，AdaptiveAvgPool2d直接求特征图中每channel内像素的均值，然后作为特征扔到一个DNN网络回归训练，预测出alpha值再相乘原channel像素，抑制特定雨滴形状和纹理输出。  
&emsp;&emsp;2.RNN模块的hidden state是上阶段相应位置的feature map（考虑到RNN长距离传递的缺陷，作者还实现了lstm、gru），两个阶段的feature map像素扔进RNN一顿tanh、sigmoid操作，计算惊心动魄、不分你我。  
&emsp;&emsp;3作者将雨景图像定义为背景和雨滴前景，生成网络输出的是一张雨滴前景图像，最后将雨景图像减去雨滴前景就是去雨图像。    
&emsp;&emsp;作者没有提供pretrained model,实际效果也无法测试。  
&emsp;&emsp;同样是单张图像去雨之前还看到过一个实现:[IDGAN](https://github.com/hezhangsprinter/ID-CGAN/tree/master/IDCGAN)，采用GAN Loss加上Perceptual Loss，借鉴艺术化风格网络， 效果号称也是state of the art。  
&emsp;&emsp;cvpr2019还有一个视频超分辨率方法:[RBPN](https://alterzero.github.io/projects/RBPN.html)，结构如图。当前帧和前面几帧，加上光流图，concat一起，一顿常规操作，结构看着比较清晰。因为和我研究的应用场景关注点不同，并没有去实际验证。
![RBPN](https://i.loli.net/2019/08/05/YrqpcjmCHVk8EKd.png)  

&emsp;&emsp;一个感想:做算法的可劲调包调差，可劲地造，不会注意到做AI芯片的如何让去部署并优化这些刷榜的。  

&emsp;&emsp;最后，说到视频中马赛克渐渐消失的技术:   
![smile](https://i.loli.net/2019/08/05/O3jCNYH4qwgSMIc.gif)