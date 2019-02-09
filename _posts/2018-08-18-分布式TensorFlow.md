---
layout:     post
title:      分布式TensorFlow -- DeepFM
subtitle:   个人整理
date:       2018-08-18
author:     Zach
header-img: img/post-bg-field.jpg
catalog: true
tags:
    - TensorFlow
    - 分布式
    - DeepFM
---

## 概要
- 分布式几种模式：
    + 参数更新：
        * 同步更新
        * 异步更新
    + 数据图构建模式：
        * In-graph，模型并行，每个worker计算模型的不同部分
        * Between-graph，数据并行，每台机器使用完全相同的计算图，使用数据的不同部分
    + 本例为——数据并行+同步更新
- 分布式结构:
    + **ps服务器**
        * 负责管理和更新参数，收集各个worker计算的梯度计算出平均梯度更新模型参数
    + **worker服务器**
        * 同步模式中```chief-worker```(即第一个worker)比较特殊，实践发现模型ckp文件最后会存储在它这里；并且所有worker都会等待```chief-worker```调起后再开始运行
        * 负责主要的模型计算
- DeepFM单机版模型基于[ChenglongChen的实现](https://github.com/ChenglongChen/tensorflow-DeepFM) 
    + 注意该版本的DeepFM实现中使用了 ```tf.Graph() ``` 自行新建了一个图，各个变量如果想从外部改变需要先取到他所新建的图
    + 该版本不支持稀疏向量、FM层Deep层都不支持特征变长的样本，自行改进做了一个新的单机版，还参考paddlepaddle使用python的geenrator输入训练数据，实现了单机训练大数据集
- 分布式代码文件：
    + [DeepFM_distributed_script.py](https://github.com/Zachary4biz/algrithm/blob/master/python/7-Tensorflow/Distributed/Flow/DeepFM_distributed_script.py)&nbsp;&nbsp;<font color="gray">*模型代码*</font> 
    + [send_file_to_all_nodes.py](https://github.com/Zachary4biz/algrithm/blob/master/python/7-Tensorflow/Distributed/Flow/send_file_to_all_nodes.py)&nbsp;&nbsp;<font color="gray">*更新模型代码到各个机器上*</font> 
    + [handler.py](https://github.com/Zachary4biz/algrithm/blob/master/python/7-Tensorflow/Distributed/Flow/handler.py)&nbsp;&nbsp;<font color="gray">*在本机通过ssh在服务器上执行任务，并进行监控、操控。*</font> 
    + [tensorflow_webservice](link)&nbsp;&nbsp;<font color="gray">*基于Flask实现在网页端监控各node的log日志、kill某个任务*</font>
- 服务器可以通过诸如 ``` ssh 10.xx.xx.xx ll -th ``` 从本地直接在服务器上执行命令 ```ll-th```



