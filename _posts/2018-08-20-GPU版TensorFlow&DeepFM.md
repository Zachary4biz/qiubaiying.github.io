---
layout:     post
title:      GPU版TensorFlow&DeepFM
subtitle:   个人整理
date:       2018-08-20
author:     Zach
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - TensorFlow
    - GPU
    - CUDA
    - DeepFM
    - 已完成
---
## 概要
- 查看GPU使用情况 ```nvidia-smi```
- TensorFlow分为“CPU版”和“GPU版”，使用清华的源来进行pip安装
    + “CPU版”：```pip install tensorflow -i https://pypi.tuna.tsinghua.edu.cn/simple```
    + “GPU版”：```pip install tensorflow-gpu -i https://pypi.tuna.tsinghua.edu.cn/simple```
- CUDA的版本和TensroFlow的版本是有一定对应关系的
    + 查看CUDA版本 ```cat /usr/local/cuda/version.txt```
    + 参考自 [@Lfxxx的博客](https://blog.csdn.net/lifuxian1994/article/details/81103530)
>tensorflow-gpu v1.9.0 | cuda9.0 <br/>
 tensorflow-gpu v1.8.0 | cuda9.0 <br/>
 tensorflow-gpu v1.7.0 | cuda9.0 <br/>
 tensorflow-gpu v1.6.0 | cuda9.0 <br/>
 tensorflow-gpu v1.5.0 | cuda9.0 <br/>
 tensorflow-gpu v1.4.0 | cuda8.0 <br/>
 tensorflow-gpu v1.3.0 | cuda8.0 <br/>
 tensorflow-gpu v1.2.0 | cuda8.0 <br/>
 tensorflow-gpu v1.1.0 | cuda8.0 <br/>

- 安装完毕后的简单验证
    + 可以使用如下代码查看当前机器上可用的GPU，[输出log见此](https://github.com/Zachary4biz/Tahiti_any_data/blob/master/log_tf_check_avaliable_gpu_of_device.txt)
    ```python3
    rom tensorflow.python.client import device_lib
    device_lib.list_local_devices()
    ```
    + 使用如下代码尝试指定GPU， [输出log见此](https://github.com/Zachary4biz/Tahiti_any_data/blob/master/log_tf_example_of_specify_device.txt)
    ```python3
    import tensorflow as tf
    with tf.device('/gpu:0'):
        a = tf.constant([1, 2, 3, 4, 5, 6], shape=[2, 3], name='a')
        b = tf.constant([1, 2, 3, 4, 5, 6], shape=[3, 2], name='b')
        c = tf.matmul(a, b)

    with tf.Session(config=tf.ConfigProto(allow_soft_placement=True,log_device_placement=True)) as sess:
        print(sess.run(c))
    ```

## DeepFM
- 测试效果：
    + 使用cpu版本运行时，1000个batch耗时 [91.7s] 左右
    + 使用如下方式配置cpu、gpu后， 1000个batch耗时 [37.7s]
    + gpu占用在 18%~22%
    <br/>![gpu信息](https://zachblog-1256781535.cos.ap-shanghai.myqcloud.com/gpu%E4%BD%BF%E7%94%A8%E6%83%85%E5%86%B5.jpg "Optional title"){:height="50%" width="50%"}
- 配置方式：Deep层在gpu上进行，FM层在cpu上进行，最后合并也在cpu上进行
    + 直接把FM层的实现放在```with tf.device("/cpu:0"):```里面，Deep层的实现放到```with df.device("/gpu:0"):```里面
    + FM层:
    ```python3
    with tf.device("/cpu:0"):
        # ---------- first order term ----------
        self.y_first_order = tf.nn.embedding_lookup(self.weights["feature_bias"], self.feat_index) # None * F * 1
        self.y_first_order = tf.reduce_sum(tf.multiply(self.y_first_order, feat_value), 2)  # None * F
        self.y_first_order = tf.nn.dropout(self.y_first_order, self.dropout_keep_fm[0]) # None * F
        # ---------- second order term ---------------
        # sum_square part
        self.summed_features_emb = tf.reduce_sum(self.embeddings, 1)  # None * K
        self.summed_features_emb_square = tf.square(self.summed_features_emb)  # None * K
        # square_sum part
        self.squared_features_emb = tf.square(self.embeddings)
        self.squared_sum_features_emb = tf.reduce_sum(self.squared_features_emb, 1)  # None * K
        # second order
        self.y_second_order = 0.5 * tf.subtract(self.summed_features_emb_square, self.squared_sum_features_emb)  # None * K
        self.y_second_order = tf.nn.dropout(self.y_second_order, self.dropout_keep_fm[1])  # None * K
    ```
    + Deep层：
    ```python3
    with tf.device("/gpu:0"):
        # ---------- Deep component ----------
        self.y_deep = tf.reshape(self.embeddings, shape=[-1, self.field_size * self.embedding_size]) # None * (F*K)
        self.y_deep = tf.nn.dropout(self.y_deep, self.dropout_keep_deep[0])
        for i in range(0, len(self.deep_layers)):
            self.y_deep = tf.add(tf.matmul(self.y_deep, self.weights["layer_%d" %i]), self.weights["bias_%d"%i]) # None * layer[i] * 1
            if self.batch_norm:
                self.y_deep = self.batch_norm_layer(self.y_deep, train_phase=self.train_phase, scope_bn="bn_%d" %i) # None * layer[i] * 1
            self.y_deep = self.deep_layers_activation(self.y_deep)
            self.y_deep = tf.nn.dropout(self.y_deep, self.dropout_keep_deep[1+i]) # dropout at each Deep layer
    ```


## 报错及解决

- 报错：使用```tf.device('/gpu:0')```指定gpu时报错，```Cannot assign a device for operation 'MatMul_1': Could not satisfy explicit device specification '/device:GPU:0' because no supported kernel for GPU devices is available```
    + [官方issue](https://github.com/tensorflow/tensorflow/issues/2285)
    + 需要给sess添加config参数
    ```python3
    config = tf.ConfigProto(allow_soft_placement=True)
    with tf.Session(config=config) as sess:
    ```
    
- 报警：```RuntimeWarning: compiletime version 3.5 of module 'tensorflow.python.framework.fast_tensor_util' does not match runtime version 3.6```
    + 大意是所安装的tensorflow是由Python3.5编译的，而当前环境用的Python3.6。
    + 暂时未发现这会导致什么问题









