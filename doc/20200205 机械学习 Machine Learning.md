---
title: 20200205 Machine Learning 
date: 2020-02-05
---

## NumPy 库学习

### numpy 库介绍

NumPy（Numerical Python的缩写）是一个开源的Python科学计算库。使用NumPy，就可以很自然地使用数组和矩阵。 NumPy包含很多实用的数学函数，涵盖线性代数运算、傅里叶变换和随机数生成等功能。

### numpy 库的安装与导入

```
pip install numpy 

import numpy as np
```

### numpy 常量

```python
	numpy.array().ndim  表示数组的轴，eg: 一维，二维，三维等...
    numpy.array().shape 表示数组的维度
    numpy.array().size  表示数组中元素总个数
    numpy.array().dtype 表示数组中元素的类型
```

### 创建数组的方式

```python
1. 给定一个序列（列表，元组等...） 
2. 使用 numpy 函数生成
    numpy.random.random().reshape()    
    numpy.random.randint().reshape()
    numpy.random.randn().reshape()
    numpy.random.rand().reshape()

    numpy.arange().reshape()   
    numpy.where() 
3. 特殊数组
    numpy.zeros()  # 全 零
    numpy.ones()   # 全 1   
```

### 数组切片

```python
a = np.arange(12).reshape(3, 4)
print(a)
print("-------------")
# 获取第一行和第三行
print(a[::2], ::)
print("-------------")
# 获取第一列和第三列
print(a[::, ::2])
print("-------------")
# 获取第一行和第三行与第一列和第三列相较的点
print(a[::2, ::2])
```

### 数组遍历

```python
a = np.arange(12).reshape(4, 3)
print(a)
# 获取数组的行数 a.shape[0]
# 获取数组的列数 a.shape[1]


for i in range(a.shape[0]):
	for j in range(a.shape[1]):
		print(a[i][j], end='\t')
	print()  
```

### 矩阵运算

 矩阵的加法、减法、数乘、乘法、矩阵的逆、矩阵的行列转置

```python
a = np.arange(12).reshape(3, 4)
b = np.random.randint(1, 5, 12).reshape(3, 4)
c = np.random.randint(0, 5, 12).reshape(4, 3)
print("a:\n", a)
print("b:\n", b)
print("c:\n", c)
print("矩阵相加:\n", a + b)
print("矩阵相减:\n", a - b)
print("矩阵数乘:\n", 2 * a)
print("矩阵乘法:\n", a.dot(c))
print("矩阵的逆:\n", np.invert(b))
print("矩阵的转置: \n", b.T)
print("b*b.T: \n", b.dot(b.T))
```

### 矩阵形状变换

```python
a = np.arange(12).reshape(3, 4)
one = np.ones((3, 4))
print(a)
print(one)
print("----------------")
# ravel 扁平化
print(np.ravel(a))
# 合并 vstack hstack
print(np.vstack((a, one)))
print(np.hstack((a, one)))
print(np.row_stack((a, one)))
print(np.column_stack((a, one)))
print("-----------------------")
# 拆分
# print(np.split(a, 3))  # indices 不能是 2
# print(np.vsplit(a, 3))
print(np.hsplit(a, 4))
```

### 常用函数

- tile 函数的使用

  tile 给定矩阵，按照行或者列复制

  ```
  >>> dataSet = p.zeros((3, 2))
  >>> dataSet
  array([[0., 0.],
         [0., 0.],
         [0., 0.]])
  >>> dataSet[0,:] = [1, 2]
  >>> dataSet[1,:] = [3, 4]
  >>> dataSet[2,:] = [5, 6]
  >>> dataSet
  array([[1., 2.],
         [3., 4.],
         [5., 6.]])
  >>> p.tile(dataSet, (2,1))
  array([[1., 2.],
         [3., 4.],
         [5., 6.],
         [1., 2.],
         [3., 4.],
         [5., 6.]])
  >>> p.tile(dataSet, (2,2))
  ```



## PySpark 学习

### PySpark 环境准备

准备：【Spark2.3+ python版本需要python3.5+ | spark1.6 对应的python版本是python3.5】

```
1. 在 window 环境下解压 spark 程序，并配置 SPARK_HOME 环境变量 
	
2. 安装 anaconda 并配置环境变量 
	ANACONDA_HOME
	ANACONDA_HOME\Scripts
	ANACONDA_HOME\Libary\bin

3. 安装模块 py4j 和 pyspark 
	第一种方式： 
		pip install 
	第二种方式： 
		从 %SPARK_HOME%\python\lib 拷贝 py4j 和 pyspark 
		到 ANACONDA_HOME\lib\site-packages		
```

特别注意：<font color='red'>配置好环境变量，一定要重启 IDE。</font>

**为 Anaconda 添加清华源**

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
# 以上两条是Anaconda官方库的镜像

conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
# 以上是Anaconda第三方库 Conda Forge的镜像

# for linux
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
# for legacy win-64
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/peterjc123/
以上两条是Pytorch的Anaconda第三方镜像

conda config --set show_channel_urls yes
```

参考： [为 Anaconda 添加清华源](https://zhuanlan.zhihu.com/p/47663391)

### WordCount

```python
"""
    Python 版本 Spark WordCount
"""
from pyspark import SparkConf, SparkContext
import os

if __name__ == "__main__":
    # os.environ['SPARK_HOME'] = "F:\\usr\\spark-2.3.1-bin-hadoop2.6"
    # 创建 SparkConf
    conf = SparkConf()
    conf.setMaster("local[1]")
    conf.setAppName("Python_Spark_WordCount")

    sc = SparkContext(conf=conf)
    file = sc.textFile("./words.txt")
    file.foreach(print)
    flap_words = file.flatMap(lambda line: line.split(" "))
    map_words = flap_words.map(lambda word: (word, 1))
    result_words = map_words.reduceByKey(lambda v1, v2: v1 + v2)
    result = result_words.sortBy(lambda t: t[1])
    result.foreach(print)
```

### PVUV

> 数据格式：2020-02-05  192.168.143.199    uid98105   shanghai   www.jd.com logout

**计算每个网址访问量最高的 top2 地区**

![](http://img.zwer.xyz/blog/20200206101824.png)

```python
def every_site_visit_top2(file):
    """
    计算每个网址访问量最高的 top2 地区
    :param file:
    :return:
    """
    file.map(lambda one: (one.split("\t")[4], one.split("\t")[3])) \
        .groupByKey() \
        .map(lambda one: get_site_local_Top2(one)) \
        .foreach(print)
        
def get_site_local_Top2(one):
    """
    获取单个网站访问量最高的地区
    :param one:
    :return:
    """
    site = one[0]
    local_iterable = one[1]
    # 创建字典，用于统计地区树
    local_dict = {}
    # 遍历地区
    for local in local_iterable:
        if local in local_dict:
            local_dict[local] += 1
        else:
            local_dict[local] = 1
    # 对字典进行排序
    new_list = sorted(local_dict.items(), key=lambda pair: pair[1], reverse=True)
    # 存放结果的列表
    result_list = []
    if len(new_list) > 2:
        for i in range(2):
            result_list.append(new_list[i])
    else:
        result_list = new_list
    return site, result_list
```

**计算每个网址最活跃的 top3 用户**

![](http://img.zwer.xyz/blog/20200206102315.png)

```python
def every_site_user_top3(file):
    """
    计算每个网址最活跃的 top3 用户
    :param file:
    :return:
    """
    file.map(lambda line: (line.split("\t")[2], line.split("\t")[4])) \
        .groupByKey() \
        .flatMap(lambda line: get_site2user_count(line)) \
        .groupByKey() \
        .map(lambda one: get_site2user_count_top3(one)) \
        .foreach(print)
        
def get_site2user_count(line):
    """
    数据格式转化：user,[site, site, site] => site,(user,count)
    :param line:
    :return:
    """
    user = line[0]
    sites = line[1]
    site_dict = {}
    for site in sites:
        if site in site_dict:
            site_dict[site] += 1
        else:
            site_dict[site] = 1
    result_list = []
    for site, count in site_dict.items():
        result_list.append((site, (user, count)))
    return result_list


def get_site2user_count_top3(one):
    """
        site,[(user,count), (user,count), (user,count), ...]
        求 top3
        3 5 1 8 6 4 7
        [3] [5, 3] [5, 3 ,1]    2 0   2 1  2 2
        [8, 5, 3]  [8,]
    :param one:
    :return:
    """
    site = one[0]
    user_count_iter = one[1]
    top3_list = ['', '', '']
    for user_count in user_count_iter:
        count = user_count[1]
        for i in range(len(top3_list)):
            if top3_list[i] == '':
                top3_list[i] = user_count
                break
            elif top3_list[i][1] < count:
                for j in range(2, i, -1):
                    top3_list[j] = top3_list[j-1]
                top3_list[i] = user_count
                break
    return site, top3_list        
```

**完整代码**

```python
"""
    pv: page view 网站访问量
    uv: unique view  单个用户访问网站的次数
"""
from pyspark import SparkConf, SparkContext


def pv(file):
    """
    计算 pv
    :param file:
    :return:
    """
    file.map(lambda line: (line.split("\t")[4], 1)) \
        .reduceByKey(lambda v1, v2: v1 + v2) \
        .sortBy(lambda t: t[1], ascending=False) \
        .foreach(print)


def uv(file):
    """
    计算 uv
    :param file:
    :return:
    """
    file.map(lambda line: (line.split("\t")[1] + "#" + line.split("\t")[4], 1)) \
        .distinct() \
        .map(lambda pair: (pair[0].split("#")[1], 1)) \
        .reduceByKey(lambda v1, v2: v1 + v2) \
        .sortBy(lambda t: t[1], ascending=False) \
        .foreach(print)


def bj_pv(file, location):
    """
    计算北京地区的 pv
    :param file:
    :param location:
    :return:
    """
    file.filter(lambda one: one.split("\t")[3] == location) \
        .map(lambda one: (one.split("\t")[4], 1)) \
        .reduceByKey(lambda v1, v2: v1 + v2) \
        .sortBy(lambda t: t[1], ascending=False) \
        .foreach(print)


def every_site_user_top3(file):
    """
    计算每个网址最活跃的 top3 用户
    :param file:
    :return:
    """
    file.map(lambda line: (line.split("\t")[2], line.split("\t")[4])) \
        .groupByKey() \
        .flatMap(lambda line: get_site2user_count(line)) \
        .groupByKey() \
        .map(lambda one: get_site2user_count_top3(one)) \
        .foreach(print)


def every_site_visit_top2(file):
    """
    计算每个网址访问量最高的 top2 地区
    :param file:
    :return:
    """
    file.map(lambda one: (one.split("\t")[4], one.split("\t")[3])) \
        .groupByKey() \
        .map(lambda one: get_site_local_Top2(one)) \
        .foreach(print)


def get_site_local_Top2(one):
    """
    获取单个网站访问量最高的地区
    :param one:
    :return:
    """
    site = one[0]
    local_iterable = one[1]
    # 创建字典，用于统计地区树
    local_dict = {}
    # 遍历地区
    for local in local_iterable:
        if local in local_dict:
            local_dict[local] += 1
        else:
            local_dict[local] = 1
    # 对字典进行排序
    new_list = sorted(local_dict.items(), key=lambda pair: pair[1], reverse=True)
    # 存放结果的列表
    result_list = []
    if len(new_list) > 2:
        for i in range(2):
            result_list.append(new_list[i])
    else:
        result_list = new_list
    return site, result_list


def get_site2user_count(line):
    """
    数据格式转化：user,[site, site, site] => site,(user,count)
    :param line:
    :return:
    """
    user = line[0]
    sites = line[1]
    site_dict = {}
    for site in sites:
        if site in site_dict:
            site_dict[site] += 1
        else:
            site_dict[site] = 1
    result_list = []
    for site, count in site_dict.items():
        result_list.append((site, (user, count)))
    return result_list


def get_site2user_count_top3(one):
    """
        site,[(user,count), (user,count), (user,count), ...]
        求 top3
        3 5 1 8 6 4 7
        [3] [5, 3] [5, 3 ,1]    2 0   2 1  2 2
        [8, 5, 3]  [8,]
    :param one:
    :return:
    """
    site = one[0]
    user_count_iter = one[1]
    top3_list = ['', '', '']
    for user_count in user_count_iter:
        count = user_count[1]
        for i in range(len(top3_list)):
            if top3_list[i] == '':
                top3_list[i] = user_count
                break
            elif top3_list[i][1] < count:
                for j in range(2, i, -1):
                    top3_list[j] = top3_list[j-1]
                top3_list[i] = user_count
                break
    return site, top3_list


if __name__ == "__main__":
    #     0               1           2            3             4             5
    # 2020-02-05	192.168.118.234	uid39474	shanghai	www.baidu.com	comment
    conf = SparkConf().setAppName("puvu").setMaster("local")
    sc = SparkContext(conf=conf)
    file = sc.textFile("./pvuvdata")
    # 计算 pv
    # pv(file)
    # 计算 uv
    # uv(file)
    # 计算北京地区的 pv
    # bj_pv(file, "beijing")
    # 计算每个网址访问量最高的 top2 地区
    # every_site_visit_top2(file)
    # 计算每个网址最活跃的 top3 用户
    every_site_user_top3(file)
```

## 线性回归算法

### 回归

从大量的函数结果和自变量反推回函数表达式的过程就是回归。线性回归是利用数理统计中回归分析来确定两种或两种以上变量间相互依赖的定量关系的一种统计分析方法。

### 一元线性回归

包括一个自变量（$x_1$） 和 一个因变量（$y$）,且二者的关系可用一条直线近似表示，这种回归分析称为一元线性回归分析。公式：$y = w_0 + w_1x_1$

这里是 $w_0$ 叫做一元线性回归公式的**截距**，$w_1$ 叫做一元线性回归公式的**斜率**

### 多元线性回归

如果回归分析中包括两个或两个以上的自变量，且因变量和自变量之间是线性关系，则称为多元线性回归分析。公式：
$$
y = w_0 + w_1x_1 + w_2x_2 + w_3x_3 + ... + w_nx_n + w_nx_n
$$


### 最小二乘法误差公式

![](http://img.zwer.xyz/blog/20200206180618.png)

### 确定权重

![](http://img.zwer.xyz/blog/20200206181807.png)

### 梯度下降

[梯度下降](https://www.cnblogs.com/pinard/p/5970503.html)

### 线性回归代码

```python
package com.bjsxt.lr

import org.apache.log4j.{Level, Logger}
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.{LabeledPoint, LinearRegressionWithSGD}
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object LinearRegression {

  def main(args: Array[String]) {
    // 构建Spark对象
    val conf = new SparkConf().setAppName("LinearRegressionWithSGD").setMaster("local")
    val sc = new SparkContext(conf)
    Logger.getRootLogger.setLevel(Level.WARN)

    //读取样本数据 1.2669476,-0.929360463147704 -0.0578991819441687 0.152317365781542 -1.02470580167082 -0.522940888712441 -0.863171185425945 -1.04215728919298 -0.864466507337306
    val data = sc.textFile("lpsa.data")
    val examples :RDD[LabeledPoint]= data.map { line =>
      val parts = line.split(',')
      val y = parts(0)
      val xs = parts(1)
      LabeledPoint(parts(0).toDouble, Vectors.dense(parts(1).split(' ').map(_.toDouble)))
    }

    val train2TestData: Array[RDD[LabeledPoint]] = examples.randomSplit(Array(0.8, 0.2), 1L)

    /*
     *  迭代次数
     *  训练一个多元线性回归模型收敛（停止迭代）条件：
     *  	1、error值小于用户指定的error值
     *  	2、达到一定的迭代次数
     */
    val numIterations = 100

    //在每次迭代的过程中 梯度下降算法的下降步长大小    0.1 0.2 0.3 0.4
    val stepSize = 0.003

    val miniBatchFraction = 1

    val lrs = new LinearRegressionWithSGD()
    //让训练出来的模型有w0参数，就是有截距
    lrs.setIntercept(true)
    //设置步长
    lrs.optimizer.setStepSize(stepSize)
    //设置迭代次数
    lrs.optimizer.setNumIterations(numIterations)
    //每一次下山后，是否计算所有样本的误差值,1代表所有样本,默认就是1.0
    lrs.optimizer.setMiniBatchFraction(miniBatchFraction)

    val model = lrs.run(train2TestData(0))
    println("weights = "+  model.weights)
    println("intercept = "+ model.intercept)

    // 对样本进行测试
    val prediction = model.predict(train2TestData(1).map(_.features))
    val predictionAndLabel: RDD[(Double, Double)] = prediction.zip(train2TestData(1).map(_.label))

    val print_predict = predictionAndLabel.take(20)
    println("prediction" + "\t" + "label")
    for (i <- 0 to print_predict.length - 1) {
      println(print_predict(i)._1 + "\t" + print_predict(i)._2)
    }
    
    // 计算测试集平均误差
    val loss = predictionAndLabel.map {
      case (p, v) =>
        val err = p - v
        Math.abs(err)
    }.reduce(_ + _)
    val error = loss / train2TestData(1).count
    println("Test RMSE = " + error)
    // 模型保存
//    val ModelPath = "model"
//    model.save(sc, ModelPath)
//    val sameModel = LinearRegressionModel.load(sc, ModelPath)
    sc.stop()
  }

}
```



## 逻辑回归

#### 逻辑回归公式

$$
f(z) = \frac{1}{1+e^{-z}}
$$



## 模式评判

### 混淆矩阵

### ROC 曲线

### AUC 面积

## 推荐系统

### 协同过滤

- 基于用户的协同过滤（适用于网站初期建设）
- 基于商品的协同过滤

推荐系统使用协同过滤思想，逻辑回归

### 推荐系统架构

### 准备数据

在 Hive 中添加 Python 文件

```hive
add file file.py 
```



## 决策树

### 决策树介绍

> 决策树是一种非线性有监督分类模型 ，随机森林是一种非线性有监督分类模型


线性分类模型比如说逻辑回归，可能会存在不可分问题，但是非线性分类就不存在

### 认识决策树

![](http://img.zwer.xyz/blog/20200223155651.png)

### 决策树术语

- 术语

  根节点：最顶层的分类条件
  叶节点：代表每一个类别号
  中间节点：中间分类条件
  分枝：代表每一个条件的输出
  二叉树：每一个节点上有两个分枝
  多叉树：每一个节点上至少有两个分枝

<img src="http://img.zwer.xyz/blog/20200223155957.png" style="zoom:50%;" />



### 决策树分裂原则

> 决策树的生成：数据不断分裂的递归过程，每一次分裂，尽可能让**类别一样的数据在树的一边**，当树的叶子节点的数据都是一类的时候，则停止分类。(if else 语句)

![](http://img.zwer.xyz/blog/20200223165426.png)





### 决策树分裂纯粹度

**量化纯粹度**

- 度量信息混乱程度指标：
  信息熵H(X)：信息熵是香农在1948年提出来量化信息的信息量的。熵的定义如下
  $$
  H(X) = -\sum_{i=1}^np_ilog{p_i} 
  $$

- 基尼系统
  $$
  Gini(p) = \sum_{k=1}^kp_k(1-p_k) = 1- \sum_{k=1}^kp_k^2
  $$
  
- 条件熵：类似于条件概率，在知道 Y 的情况下， x 的不确定性
  $$
  H(X|Y) = H(X,Y) - H(Y) \\
         =  \sum_xp(x)H(Y|X=x)
  $$

- 信息增益：代表的熵的变化程度

  特征 Y 对训练集 D 的信息增益  $g(D,Y) = H(X) - H(X|Y)$

  在构建决策树的时候就是选择信息增益最大的属性作为分裂条件（ID3）

