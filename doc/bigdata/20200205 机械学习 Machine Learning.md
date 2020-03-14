---
title: 20200205 Machine Learning 
date: 2020-02-05
---

## 线性回归算法

### 回归

> 从大量的函数结果和自变量反推回函数表达式的过程就是回归。线性回归是利用数理统计中回归分析来确定两种或两种以上变量间相互依赖的定量关系的一种统计分析方法。

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

