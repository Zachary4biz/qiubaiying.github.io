---
layout:     post
title:      DBSCAN在Spark上的应用
subtitle:   使用开源库
date:       2018-06-07
author:     Zach
header-img: img/post-bg-field.jpg
catalog: true
tags:
    - DBSCAN
    - Spark
    - Scala
    - Algorithm
---

### 开源库
- **总览**

1. [scalanlp/nak](https://github.com/scalanlp/nak)
2. [irvingc/dbscan-on-spark](https://github.com/irvingc/dbscan-on-spark)
3. [alitouka/spark_dbscan](https://github.com/alitouka/spark_dbscan)
4. [ELKI](https://elki-project.github.io/)

- **相关文章或讨论**

1. [关于scalanlp/nak的一篇博客](https://blog.csdn.net/a3301/article/details/53229100)
2. [SO上关于后三个开源库的讨论](https://stackoverflow.com/questions/36090906/dbscan-on-spark-which-implementation)
3. [Breeze库的API总结](https://blog.csdn.net/u012102306/article/details/53463388)

### 实际使用经验
- **结论**
    + scalanlp/nak <br/>
        [感觉不能用](#nak)，基于RDD做的，使用的是breeze.linalg.DenseMatrix,比较老旧了测试代码跑了半个小时也没出结果<br/>
    + irvingc/dbscan-on-spark <br/> 
        [不能用](#dbscan-on-spark)，这个只支持二维的，git上有个人提交了commit看起来是让它支持多维，但是从源码上粗略看了下，感觉还是不支持多维的，具体后续有时间再看。<br/>
    + alitouka/spark_dbscan  <br/>
        [更新到spark2.2很麻烦，有几十个文件](#spark_dbscan)


- <p id="nak">scalanlp/nak</p>

    + <p> 使用mvn项目管理，所以先导入mvn包（[mvn官方关于这个包的信息](http://mvnrepository.com/artifact/org.scalanlp/nak/1.1.0)）</p>
    ``` html
        <!-- https://mvnrepository.com/artifact/org.scalanlp/nak -->
        <dependency>
            <groupId>org.scalanlp</groupId>
            <artifactId>nak_2.10</artifactId>
            <version>1.3</version>
        </dependency>
    ```
    + <p> 发现只适用 *RDD* + *breeze.linalg.DenseMatrix*<br/> 
    Spark已经2.0以后都偏向于DataFrame了，很多地方实际用不上RDD了。<br/>
    而Breeze类似于python的numpy[是一个做向量、矩阵运算的库](https://blog.csdn.net/u012102306/article/details/53463388)，没有用到Spark自己的 *ml.linalg.DenseMatrix* 怀疑这里是不是能做到分布式并行运行DBSCAN。 </p>
    ```scala
    def dbscan(v : breeze.linalg.DenseMatrix[Double]) = {
      val gdbscan = new nak.cluster.GDBSCAN(
        nak.cluster.DBSCAN.getNeighbours(epsilon = 0.001, distance = nak.cluster.Kmeans.euclideanDistance),
        nak.cluster.DBSCAN.isCorePoint(minPoints = 3)
      )
      val clusters = gdbscan cluster v
    }

    val rdd_1 = {
      spark.read.parquet("apus_ai.db/ac/ac_ar_day/kMeans_featureDF/dt=2018-05-22").rdd
        .zipWithIndex()
        .map(x =>(x._1.getAs[String]("client_id_s"),x._1.getAs[SparseVector]("features").toDense,x._2))
    }
    val arr = rdd_1.map(_._2.toArray).collect()
    val matrix2 = breeze.linalg.DenseMatrix.create(arr.length,arr(1).length,arr.flatMap(x => x))
    // matirx : 90268 * 2169
    dbscan(matrix2) // 14:58开始 
    ```
    + <font color="#5151a2">直接使用MVN配置的包 nak_2.10，但是内部有编译错误，Kmeans.class 丢失了。nak的确是不能用</font>
    + 这里其实是由于Spark的版本不同，我使用的是2.2，mvn上的jar包用的2.0，所以直接从github把源码拷过来自己重新打个jar包就行了
    + 
    


- <p id="dbscan-on-spark"> dbscan-on-spark </p>

    + <p> 使用mvn导入的包有编译错误，所以直接从git上把包拿下来，自己改了下 </p>
    + 深入研究了下源码，大致思路如下：
        * 数据集按“个数”先分区，最优情况下是各个分区的数据点个数都相同
        * 各个分区内部进行DBSCAN分类（这里触发Spark的就是此分区，各分区并行计算）
        * 然后把接近
        * 
    + 

- <p id="spark_dbscan"> spark_dbscan </p>
    + 来源









