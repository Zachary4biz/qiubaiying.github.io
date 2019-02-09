---
layout:     post
title:      使用Timeline工具 && tfdbg
subtitle:   改进
date:       2018-08-29
author:     Zach
header-img: img/post-bg-field.jpg
catalog: true
tags:
    - DeepFM
    - TensorFlow
    - 优化
    - How to
---



```python3
from tensorflow.python.client import timeline
self.options = tf.RunOptions(trace_level = tf.RunOptions.SOFTWARE_TRACE)
self.run_metadata = tf.RunMetadata()

```


```python3
/job:localhost/replica:0/task:0/gpu:0 compute
```
这个条目下面的实际是在cpu上运行，作用是把运算发送到gpu上进行计算
```python3
/gpu:0/stream:all compute
```
这个条目下面才是gpu计算的部分


## 报错及解决

- 报错：```libcublas.so.9.0: cannot open shared object file: No such file or directory```
    + 由CUDA版本和TensorFlow版本不匹配导致的问题（这里是TensorFlow 1.10 和 CUDA 8.0）
    + 使用```pip install --upgrade tensorflow-gpu==1.4 -i https://pypi.tuna.tsinghua.edu.cn/simple```使用 TensorFlow 1.4 版本


- 报错: timeline工具 ```Couldn't open CUDA library libcupti.so.8.0. LD_LIBRARY_PATH: ```
    + 这基本是所有初次使用timeline工具必定会碰上的问题
    + 尝试一： ```sudo apt-get install libcupti-dev```（Ubuntu才有apt-get, 使用```cat /etc/*release*```查看当前linux发行版)
    + 尝试二：CentOS使用yum ```sudo yum install libcupti-dev```（yum官方源没有libcupti-dev，但是发现对于CentOS来说，在安装cuda的时候就已经把cupti安装好了，查看目录```/usr/local/cuda-8.0/extras/CUPTI/```)
    + 解决方案：添加LD_LIBRARY_PATH环境变量```export LD_LIBRARY_PATH=/usr/local/cudnn/cuda/lib64:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH```















