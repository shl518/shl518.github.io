---
title: Stanford_CS231n
date: 2022-05-23 20:24:14
tags: DeepLearning
---

### KNN(K-nearest-Neighbor) K近邻算法

K近邻（K-Nearest Neighbor, KNN）是一种最经典和最简单的*有监督学习*方法之一。K-近邻算法是最简单的分类器，*没有显式的学习过程或训练过程*，是*懒惰学习*（Lazy Learning）。当对数据的分布只有很少或者没有任何先验知识时，K 近邻算法是一个不错的选择。

- 核心就是类似一种投票机制，测试样本附近最近的k个样本中，哪个类型的点多就归为哪一类

- K近邻的难点在于衡量距离？比如两段文本，如何衡量其距离？

  

#### 1. 如何衡量两个对象距离？

- L1：Manhattan曼哈顿距离，二维图像来看是一个正方体，正方形所有点距离远点曼哈顿相同。（视频中提到如果你的特征向量有一些具体的含义，那么maybe L1 is a better choice）.Besides, 坐标轴变换对Manhattan Distance 有影响 
- L2: Euclidean欧氏距离，如果特征向量没有更多的含义普遍采用L2，转动坐标系对L2没有影响

![image-20220523210259085](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220523210259085.png)

- 注：Hyperparameters 无法从数据中学习到的参数，k的选取，距离度量的选取

#### 2. 训练方法

`train + validation + test`

choose best hyperparameters on val ,and evaluate it on test,train is just used as labels 

![image-20220523214107087](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220523214107087.png)

数据较少的时候更好的训练方法is交叉验证法，继续划分train，train中的每一个部分挨个当作val

![image-20220523214203066](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220523214203066.png)

总结

- KNN不适用于图像分类领域
  - KNN导致维度灾难，要实现准确分类，需要的训练数据指数级增长
  - 反正就是不适合图像！看着就不适合！
