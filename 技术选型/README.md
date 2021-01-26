# 技术选型调研

* 大方向任务
  * 分布式平台
  * 选出几个可行的方案
  * 分析优缺点
* 任务细分：
  * 数据源存储的问题
  * 支持分布式的深度学习组件
  * 业内端到端的解决方案有哪些——可借鉴的架构方案

# 方案路线

1. hdfs -> mapreduce -> hive(on spark/Tez) ->  提取小批量数据 -> 预建模预分析：sklearn/Tensorflow
2. hdfs -> yarn -> spark -> spark mllib/TensorFlowonSpark/BigDL

# 数据存储

## 分布式文件系统--HDFS

## 分布式关系型数据库--Hive

**优点**：

1. 将sql转化为MapReduce，适用于离线批处理环境
2. Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合
3. Hive 优势在于处理大数据
4. Hive 支持用户自定义函数，用户可以根据自己的需求来实现自己的函数

**缺点**：

1. 基于MapReduce，速度慢 
2. Hive调优比较困难，粒度较粗
3. 迭代式算法无法表达
4. 由于 MapReduce 数据处理流程的限制，效率更高的算法却无法实现

## 分布式非关系型数据库--HBase

**优点**：

1. 容量大：Hbase单表可以有百亿行、百万列，数据矩阵横向和纵向两个维度所支持的数据量级都非常具有弹性
2. 列存储：其数据在表中是按照某列存储的，这样在查询只需要少数几个字段的时候，能大大减少读取的数量，可以动态增加列
3. 高可用，依赖于Zookeeper
4. 写入速度快，适用于读少写多的场景
5. 稀疏性，为空的列并不占用存储空间，表可以设计的非常稀疏。不需要null填充

**缺点**：

1. 不能支持条件查询，只支持按照row key来查询
2. 只能在主键上索引和排序
3. 对join以及多表合并数据的查询性能不好
4. 更新过程中有大量的写入和删除操作，需要频繁合并和分裂，降低存储效率

## 优化：Hive on Tez / Spark

**使用Tez和Spark替代MapReduce，达到提高Hive执行效率的目的**

# 计算引擎

## Spark

目前Spark支持三种分布式部署方式，分别是

* standalone
* spark on MESOS
* spark on YARN（较流行）

**优点**：（与MapReduce相比）

1. 能处理循环迭代式数据流处理
2. 适用于多并行的数据可复用场景（如：机器学习、图挖掘算法、交互式数据挖掘算法）
3. RDD提供了比MapReduce 丰富的模型，可以快速在内存中对数据集进行多次迭代，来支持复杂的数据挖掘算法和图形计算算法
4. Spark 多个作业之间数据通信是基于内存，效率更高

**缺点**：

1. Spark 是基于内存的，由于内存的限制，可能会由于内存资源不够导致 Job 执行失败

# 算法层

## SparkMLlib

* 支持的语言：python，scala，java
* 支持的文件系统：HDFS
* 支持的数据库：Hive，HBase
* 支持的算法：分类，聚类，回归，降维，协同过滤

**优点**：

1. Spark善于处理机器学习中迭代式运算，基于内存，因此迭代结果不会落在磁盘中
2. 可以使用Spark其他的衍生工具

**缺点**：

1. 缺少深度学习算法框架

## Mahout

* 支持的语言：java，scala
* 支持的文件系统：HDFS
* 支持的数据库：Hive，HBase
* 支持的算法：分类，聚类，回归，降维，协同过滤

**优点**：

1. 基于hadoop实现
2. 利用MapReduce计算引擎，提升了算法可处理的数据量和处理性能。

**缺点**：

1. 由于实现算法需要MR化，所以实现的算法相对较少
2. 学习资料较少，官网提供的相关文档并没有很详细的关于每个算法的使用教程。
3. 不支持深度学习

## TensorFlowOnSpark

使Spark能够利用TensorFlow拥有深度学习和GPU加速计算的能力，目前被用于雅虎私有云的Hadoop集群中

* 支持的语言：python
* 支持的文件系统：HDFS
* 支持的计算引擎：Spark

![image-20210124224553335](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210124224553335.png)

**优点**：

1. 可用于生产环境
2. 支持所有TensorFlow功能
3. 轻松移植现有TensorFlow程序到Spark集群上
4. 轻松整合现有的数据处理流程和机器学习算法
5. 更新频率高，且支持最新版本的TensorFlow和Spark
6. 代码量少便于二次开发

**缺点**：

1. ·缺乏许多预先训练的模型。
2. 存在学习成本
3. 开源时间短

## BigDL

支持的语言：Python，Scala

支持的文件系统：HDFS

支持的计算引擎：Spark

**优点**：

1. 具有类似kerasAPI接口
2. 可将TensorFlow，Caffe，Keras模型加载到BigDL中
3. 可以使用TensorBoard可视化

**缺点**：

1. 小众，相关资料较少

https://github.com/intel-analytics/BigDL

## H2O

* 支持的语言：Python/Java/Scala/R
* 支持的文件系统：HDFS
* 支持的计算引擎：Spark，MapReduce
* 支持的算法：AutoML，深度学习，分类，聚类

**优点**：

1. AutoML实现自动化机器学习流程

**缺点**：

1. 不支持YARN
2. 社区资料较少
3. 无HA

# 算法选择

[可参考kaggle-Telco Customer Churn](https://www.kaggle.com/blastchar/telco-customer-churn)

# 方案路线

## 方案一

hdfs -> mapreduce -> hive(on spark/Tez) ->  提取小批量数据 -> 预建模预分析：sklearn/Tensorflow

**优点**：

1. 易上手，易操作
2. 可用于预训练，预分析
3. 可以作为Baseline

**缺点**：

1. MR执行效率低
2. 属于单机处理分析

## 方案二

hdfs -> yarn -> spark -> spark mllib/TensorFlowonSpark/BigDL

**优点**：

1. 在分布式集群中进行完整的数据挖掘操作
2. 使用Spark替代MR，提高执行效率
3. 既支持机器学习模型也支持深度学习模型
4. 各个组件均可使用python语言操作

**缺点**：

1. 存在学习成本和环境部署的问题
2. 部分组件模块的学习资料较少

## 对比选型

1. 建议先采用方案一作为Baseline，快速上手，对数据进行简单处理挖掘

2. 根据Baseline得到的结论，再采用方案二，并在分布式集群上进行深入分析挖掘

# 项目呈现方式

1. 对用户进行分类，即哪类用户的流失可能性更大
2. 输入一个用户特征，对该用户进行分类评定/评分，给出用户流失的可能性以及相应方案

# 参考

https://www.secrss.com/articles/9258

https://www.jianshu.com/p/d3c68829a9de

https://www.kaggle.com/blastchar/telco-customer-churn

https://tf.wiki/zh_hans/appendix/distributed.html

https://zhuanlan.zhihu.com/p/142726510

https://www.slidestalk.com/u4656/TensorFlow_On_Spark

https://bigdl-project.github.io/master/#

https://github.com/yahoo/TensorFlowOnSpark

https://github.com/intel-analytics/BigDL
