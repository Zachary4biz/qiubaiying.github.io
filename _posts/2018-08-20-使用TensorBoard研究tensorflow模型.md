---
layout:     post
title:      使用TensorBoard
subtitle:   How to
date:       2018-08-20
author:     Zach
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - TensorFlow
    - TensorBoard
    - How to
---

## 概要
- TensorBoard 可视化
- 参考 [Stanford CS20SI课程的课件](http://web.stanford.edu/class/cs20si/lectures/)
    + 勘误: 原课件 slides_02 中示例代码 ``` writer = tf.summary.FileWriter('./graphs, sess.graph)``` 漏掉了一个引号，应该为 ``` writer = tf.summary.FileWriter('./graphs', sess.graph)``` 
- 更详细的关于```tf.summary```的使用参考了 [@love小酒窝的博客](https://www.cnblogs.com/lyc-seu/p/8647792.html)
- 使用 ```tf.name_scope("input")```对TensorFlow的图中各个节点进行分组，看起来会更清楚

## 实例
- 代码中插入summary部分
```python3
"""
这里直接使用FileWriter只会保存计算图
加上 tf.summary.scalar() 可以增加一张图，用来展示变量 self.loss 在每个batch_cnt的变化
"""
# 初始化计算图的时候顺带初始化这两个变量
tf.summary.scalar('log_loss', self.loss)
self.merge_summary = tf.summary.merge_all()#调用sess.run运行图，生成一步的训练过程数据, 是一个option
self.writer = tf.summary.FileWriter("./graphs", self.sess.graph)

# 在fit_on_batch中(即调用sess.run() 的时候顺带运行 self.merge_summary)
loss, opt, train_summary = self.sess.run((self.loss, self.optimizer, self.merge_summary), feed_dict=feed_dict)
# 然后调用self.writer的add_summary方法
self.writer.add_summary(train_summary,batch_cnt)
```
- 代码运行后，terminal或者shell中输入
```python3
tensorboard --logdir="./graphs" --port 5006
```

## 报错及解决
- 报错 ```No module named '_sqlite3'```
    + 很多方法都要安装sqlite3然后再重新编译python3，实际上这台机器使用 ```yum install sqlite*``` 提示已经安装过了sqlite3，况且也不想重新编译python3
    + 解决：找一台正常的机器，找到其```_sqlite3```文件拷贝过来
        * ```sys.path```可以找到一个以```lib-dynload```结尾的路径，里面有文件```_sqlite3.cpython-36m-x86_64-linux-gnu.so```
        * 从正常机器拷贝到目标机器的```lib-dynload```目录即可
        * (linux就找linux，不同机器的_sqlite3文件名不同的，如linux是```_sqlite3.cpython-36m-x86_64-linux-gnu.so```，mac是```_sqlite3.cpython-36m-darwin.so```)






