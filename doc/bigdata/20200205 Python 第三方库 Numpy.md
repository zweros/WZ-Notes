## NumPy 库学习

### numpy 库介绍

> NumPy（Numerical Python的缩写）是一个开源的 Python 科学计算库。使用 NumPy，就可以很自然地使用数组和矩阵。 NumPy包含很多实用的数学函数，涵盖线性代数运算、傅里叶变换和随机数生成等功能。

### numpy 库的安装与导入

- numpy 库安装

  ```
  pip install numpy
  ```

- numpy 导入

  ```python
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

代码示例：



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

  ```python
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

