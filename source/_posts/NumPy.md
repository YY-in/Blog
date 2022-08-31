---
title : NumPy
date : 2022-01-18 00:18:00
tag : [python]
categories : 深度学习基础
description : NumPy是一个在Python中做科学计算的基础库，重在数值计算，也是大部分Python科学计算库的基础库，多用于在大型、多维数组上执行数值运算。  
---
# 科学计算库NumPy

1. 了解 NumPy 数组和矩阵的使用方法；

2. 了解 NumPy 随机模块的使用方法；

3. 了解 NumPy 常用的数学函数。

NumPy是一个在Python中做科学计算的基础库，重在数值计算，也是大部分Python科学计算库的基础库，多用于在大型、多维数组上执行数值运算。  
NumPy 通常与 SciPy（Scientific Python）和 Matplotlib（绘图库）一起使用，这种组合广泛用于替代 MatLab，是一个强大的科学计算环境，有助于我们通过 Python 学习数据科学或者机器学习。

SciPy 是一个开源的 Python 算法库和数学工具包，包含的模块有最优化、线性代数、积分、插值、特殊函数、快速傅里叶变换、信号处理和图像处理、常微分方程求解和其他科学与工程中常用的计算。

Matplotlib 是 Python 编程语言及其数值数学扩展包 NumPy 的可视化操作界面。

### 1. 数组

**步骤 1	导入numpy模块**


```python
import numpy as np
```

**步骤 2	创建数组**

| **数组生成方法（numpy简写为np）**                            | **使用详情**                                                 |
| :-----------------------------------------------------------: | :------------------------------------------------------------: |
| np.array(object, dtype=None, copy=True,)                     | 创建一个数组对象。                                           |
| np.zeros(shape, dtype=float, order='C')                      | 创建一个形状为shape的全零数组。dtype为数据类型。order=C代表与c语言类似，行优先;order=F代表列优先 |
| np.ones(shape, dtype=None, order='C')                        | 创建一个全1数组，和np.zeros()类似。                          |
| np.eye(N, M=None, k=0, dtype=float, order='C')               | 生成一个类似于对角矩阵的数组，N为行数；M为列数，默认和N一样；k为对角线的索引，0代表主对角线。 |
| np.empty(shape, dtype=float, order='C')                      | 生成一个未初始化的数组                                       |
| np.asarray(a, dtype = None, order = None)                    | 将数据转化为数组。a:输入数据,dtype：数据类型，一般默认为数据本身类型，可选（区别于array，asarray会直接修改源数据） |
| np.arange(start, stop, step, dtype)                          | 生成一个等间隔的数组。  start：起始范围，默认为0，；stop：终止范围  step：前后间隔，默认为1；dtype：数据类型。 |
| np.linspace(start, stop, num=50, endpoint=True,  retstep=False, dtype=None, axis=0) | 生成一个等间隔的数组，start起始值，stop终止值，num数量，endpoint=True表示stop为最后一个值。 |


```python
a = np.array([1,2,3,4,5,6])  # 一维数组
print("一维数组：", a)

b = np.array([[1,2,3],[4,5,6]])  # 多维数组
print("多维数组：" ,b)

c = np.zeros([2,2])  # 创建一个2*2的全0数组
print("全零数组：",c)

d = np.ones([2,2])  # 创建一个2*2的全1数组
print("全一数组：",d)

print(np.eye(2))

e = np.asarray([1,2,3,4,5,6])  # 一维数组
print("一维数组：", e)

f = np.arange(10,20,2)  # 一维等间隔数组
print("等间隔数组", f)

g = np.linspace(0,100,10)  # 一维等间隔数组
print(g)
```

    一维数组： [1 2 3 4 5 6]
    多维数组： [[1 2 3]
     [4 5 6]]
    全零数组： [[0. 0.]
     [0. 0.]]
    全一数组： [[1. 1.]
     [1. 1.]]
    [[1. 0.]
     [0. 1.]]
    一维数组： [1 2 3 4 5 6]
    等间隔数组 [10 12 14 16 18]
    [  0.          11.11111111  22.22222222  33.33333333  44.44444444
      55.55555556  66.66666667  77.77777778  88.88888889 100.        ]
    

**步骤 3	数组属性**


```python
a = np.array([[1,2], [3,4], [5,6]])

print(a.ndim)  # 数组维度
print('shape =', a.shape)  # 数组的形状
print('shape_row =', a.shape[0])  # 数组第一维度上的大小,即行数
print('size =', a.size)  #数组元素的个数
print('size_column =', np.size(a,0))  # 行数
print('len =', len(a))  # 数组第一维度的大小，即行数

b = a.reshape(2,3)
print(b.ndim)
```

    2
    shape = (3, 2)
    shape_row = 3
    size = 6
    size_column = 3
    len = 3
    2
    

**步骤 4	数组常用操作**


```python
a = np.arange(10)  # 创建数组
print(a)
print(a[2:8:2])    # 切片下标为2开始到8结束（不取8），步长为2
print(a[2:8])
print(a[2])
print(a[2:])
```

    [0 1 2 3 4 5 6 7 8 9]
    [2 4 6]
    [2 3 4 5 6 7]
    2
    [2 3 4 5 6 7 8 9]
    


```python
a = np.array([[1, 2], [3, 4]])
b
print(a > 2)
print (a[a > 2])
```

    [[False False]
     [ True  True]]
    [3 4]
    

reshape操作：改变数组维度。


```python
a = np.array([[1,2], [3,4], [5,6]])
print("a:\n", a)

b = a.reshape(2, 3)
print("reshape:\n", b)
```

    a:
     [[1 2]
     [3 4]
     [5 6]]
    reshape:
     [[1 2 3]
     [4 5 6]]
    

squeeze 操作：从数组的形状中删除单维度条目，即把shape中为1的维度去掉

用法：numpy.squeeze(a,axis = None)


```python
import numpy as np

a = np.arange(10)
print("a:\n", a, "\n", a.shape)

b = a.reshape(1,1,10)
print("b:\n", b, "\n", b.shape)

c = np.squeeze(b)
print("c:\n", c, "\n", c.shape)
```

    a:
     [0 1 2 3 4 5 6 7 8 9] 
     (10,)
    b:
     [[[0 1 2 3 4 5 6 7 8 9]]] 
     (1, 1, 10)
    c:
     [0 1 2 3 4 5 6 7 8 9] 
     (10,)
    

广播

广播(Boardcasting)是NumPy中用于在不同大小的阵列（包括标量与向量，标量与二维数组，向量与二维数组，二维数组与高维数组等）之间进行逐元素运算（例如，逐元素 加法，减法，乘法，赋值等）的一组规则。
两个array的shape长度与shape的每个对应值都相等的时候，那么结果就是对应元素逐元素运算，运算的结果shape不变。



```python
a = np.array([1,2,3,4])
b = np.array([1,2,3,4])
c = a * b

print(c)
```

    [ 1  4  9 16]
    

shape长度不相等时，先把短的shape前面一直补1，直到与长的shape长度相等时，此时，两个array的shape对应位置上的值 ：1、相等 或 2、其中一个为1，这样才能进行广播。


```python
a = np.arange(27).reshape((3,3,3))
print("a:\n", a)

b = np.array([[1,2,3]])
c = a + b
print("广播之后：\n", c)
```

    a:
     [[[ 0  1  2]
      [ 3  4  5]
      [ 6  7  8]]
    
     [[ 9 10 11]
      [12 13 14]
      [15 16 17]]
    
     [[18 19 20]
      [21 22 23]
      [24 25 26]]]
    广播之后：
     [[[ 1  3  5]
      [ 4  6  8]
      [ 7  9 11]]
    
     [[10 12 14]
      [13 15 17]
      [16 18 20]]
    
     [[19 21 23]
      [22 24 26]
      [25 27 29]]]
    

**步骤 5	常用函数**

**flat函数**：按照下标取元素，在多维数组中也可以直接取到对应的元素。


```python
a = np.arange(6).reshape(2, 3)

print("a\n", a)
print("a[1]\n", a[1])
print("a.flat[1]\n", a.flat[1])
```

    a
     [[0 1 2]
     [3 4 5]]
    a[1]
     [3 4 5]
    a.flat[1]
     1
    

**transpose函数**：根据数组的行列索引值对数据进行转换（二维数组默认是转置）


```python
a = np.arange(12).reshape(2, 2, 3)
b=np.transpose(a, (2,0,1))

print('a=', a)
print('b=', b)
```

    a= [[[ 0  1  2]
      [ 3  4  5]]
    
     [[ 6  7  8]
      [ 9 10 11]]]
    b= [[[ 0  3]
      [ 6  9]]
    
     [[ 1  4]
      [ 7 10]]
    
     [[ 2  5]
      [ 8 11]]]
    

**concentrate（np.concatenate((a1,a2,a3,...), axis=0)）函数**：按照特定方向轴进行拼接，默认是列。help(print)


```python
a = np.arange(6).reshape(2, 3)
b = np.arange(7, 13).reshape(2, 3)
c = np.concatenate((a, b))
d = np.concatenate((a, b),axis = 1)  # axis=1：按行拼接

print('a=', a)
print('b=', b)
print('c=', c)
print('d=', d)
```

    a= [[0 1 2]
     [3 4 5]]
    b= [[ 7  8  9]
     [10 11 12]]
    c= [[ 0  1  2]
     [ 3  4  5]
     [ 7  8  9]
     [10 11 12]]
    d= [[ 0  1  2  7  8  9]
     [ 3  4  5 10 11 12]]
    

**stack函数（numpy.stack(arrays,axis=0)）**：用于堆叠数组，arrays是用于堆叠的数组，asix堆叠的方向。


```python
a = np.array([[1,2], [3,4]])
b = np.array([[5,6], [7,8]])
c = np.stack((a,b), 0)
d = np.stack((a,b), 1)

print('a=', a)
print('b=', b)
print('c=', c)
print('d=', d)
```

    a= [[1 2]
     [3 4]]
    b= [[5 6]
     [7 8]]
    c= [[[1 2]
      [3 4]]
    
     [[5 6]
      [7 8]]]
    d= [[[1 2]
      [5 6]]
    
     [[3 4]
      [7 8]]]
    

**delete函数（numpy.delete(arr, obj, axis=None)）**：返回删除obj处数据后的值，不会改变原来的数据。


```python
a = np.arange(6).reshape(2, 3)
b = np.delete(a, 2)
c = np.delete(a, 1, axis=1)
print('a=', a)
print('b=', b)
print('c=', c)
```

    a= [[0 1 2]
     [3 4 5]]
    b= [0 1 3 4 5]
    c= [[0 2]
     [3 5]]
    

### 2.矩阵

**步骤 1	导入numpy模块**


```python
import numpy.matlib
import numpy as np
```

**步骤 2	创建数组**


```python
a=np.matlib.zeros((2,2))  # 全零矩阵
b=np.matlib.ones((2,2))   # 全一矩阵

print('a=',a)
print('b=',b)
```

    a= [[0. 0.]
     [0. 0.]]
    b= [[1. 1.]
     [1. 1.]]
    

对角矩阵


```python
c=np.matlib.eye(3)

print('c=', c)
```

    c= [[1. 0. 0.]
     [0. 1. 0.]
     [0. 0. 1.]]
    

数组转化为矩阵


```python
d = np.array([[1,2], [3,4]])
d = np.mat(a)

print(d)
print(type(d))
```

    [[0. 0.]
     [0. 0.]]
    <class 'numpy.matrix'>
    

随机生成矩阵：

a=np.matlib.rand(3, 3)
print('a=',a)

numpy命名空间 mat/matrix/asmatrix函数：


```python
a=np.array([1,2,3])   
b=np.mat(a)       # 三个函数功能类似
d=np.asmatrix(a)  # matrix和asmatrix的区别在于asmatrix处理矩阵或数组时不复制
c=np.matrix(a)    # 类似array和asarray

print('array a=', a)
print('matrix b=', b)
print('matrix c=', c)
print('matrix d=', d)
```

    array a= [1 2 3]
    matrix b= [[1 2 3]]
    matrix c= [[1 2 3]]
    matrix d= [[1 2 3]]
    

**步骤 3	矩阵运算**


```python
a=np.mat([[1], [2], [3]])
b=np.mat([1,2,3])
c=a * b          # 星乘(*)和点乘(dot)都是从ndarray类中继承而来的
d=np.dot(a,b)  # 星乘(*)对于数组和矩阵而言进行的操作不同     

print('matrix a=', a)
print('matrix b=', b)
print('matrix c=', c)
print('matrix d=', d)
```

    matrix a= [[1]
     [2]
     [3]]
    matrix b= [[1 2 3]]
    matrix c= [[1 2 3]
     [2 4 6]
     [3 6 9]]
    matrix d= [[1 2 3]
     [2 4 6]
     [3 6 9]]
    

对应元素相乘：


```python
a = np.mat([1,2,3])
b = a
c = np.multiply(a, b)
d = a * 2

print('matrix a=', a)
print('matrix b=', b)
print('matrix c=', c)
print('matrix d=', d)
```

    matrix a= [[1 2 3]]
    matrix b= [[1 2 3]]
    matrix c= [[1 4 9]]
    matrix d= [[2 4 6]]
    

矩阵求逆：


```python
a = np.matrix([[2,0,0],
             [0,1,0],
             [0,0,2]])
b = a.I

print('matrix a=', a)
print('matrix b=', b)
```

    matrix a= [[2 0 0]
     [0 1 0]
     [0 0 2]]
    matrix b= [[0.5 0.  0. ]
     [0.  1.  0. ]
     [0.  0.  0.5]]
    

行列式计算：


```python
a = np.matrix([[1,2],
             [3,4]])
b = np.linalg.det(a)

print('matrix a=', a)
print('b=', b)
```

    matrix a= [[1 2]
     [3 4]]
    b= -2.0000000000000004
    

求和与最大最小：


```python
a = np.matrix([[1,2],
              [3,4]])
b = a.sum(axis=0)       #对列求和
c = a.sum(axis=1)       #对行求和
d = a.max()
e = a.min()

print('matrix a=', a)
print(' b=', b)
print(' c=', c)
print(' d=', d)
print(' e=', e)
```

    matrix a= [[1 2]
     [3 4]]
     b= [[4 6]]
     c= [[3]
     [7]]
     d= 4
     e= 1
    

**步骤 4	矩阵转数组**


```python
a = np.mat([[1,2,3], [4,5,6]])   
b = a.getA()  # getA将矩阵类转化为数组类

print('matrix a=', a)
print(type(a))
print('array b=', b)
print(type(b))
```

    matrix a= [[1 2 3]
     [4 5 6]]
    <class 'numpy.matrix'>
    array b= [[1 2 3]
     [4 5 6]]
    <class 'numpy.ndarray'>
    

### 3. 随机模块
**步骤 1	导入numpy模块**


```python
import numpy as np
```

**步骤 2 简单随机数据**



```python
r1 = np.random.rand(2,2)      # rand函数 范围在[0,1)
r2 = np.random.randn(2,2)     # randn函数 具有标准正态分布
r3 = np.random.randint(0,5)   # randint(low,high,size)函数 返回在[low,high)范围的整数
r4 = np.random.random((2,2))  # random函数，和random.rand函数输出相同，输入有区别

print('r1=','\n', r1)
print('r2=','\n', r2)
print('r3=','\n', r3)
print('r4=','\n', r4)
```

    r1= 
     [[0.28606482 0.24542071]
     [0.57506778 0.90029061]]
    r2= 
     [[0.17325    2.27482223]
     [0.05676126 0.02682367]]
    r3= 
     4
    r4= 
     [[0.56227428 0.98723769]
     [0.75565192 0.88940153]]
    

choice函数（choice(a[, size, replace, p])）：


```python
r1 = np.random.choice(5, 3)                           # 从np.arange(5)数组中抽取3个数
r2 = np.random.choice(5, 3, p=[0.1, 0, 0.3, 0.6, 0])  # 被抽取数组中的元素具有不同概率
r3 = np.random.choice(5, 3, replace=False)            # 不放回抽样

print('r1=','\n',r1)
print('r2=','\n',r2)
print('r3=','\n',r3)
```

    r1= 
     [1 3 2]
    r2= 
     [0 3 2]
    r3= 
     [1 0 4]
    

**步骤 3	随机排列**


```python
sample = np.arange(5)
np.random.shuffle(sample)                 #改变自身内容
r2 = np.random.permutation([1, 2, 3, 4])  #返回随机序列

print('sample=','\n', sample)
print('r2=','\n', r2)
```

    sample= 
     [4 0 2 3 1]
    r2= 
     [3 4 2 1]
    

**步骤 4	常用分布**


```python
r1 = np.random.normal(0, 0.1, 5)  # 正态分布，参数1为均值，参数2为标准差，参数3为返回值的形状，默认是None,只输出一个值，也可以为多维，如（2,3）
r2 = np.random.uniform(0, 5, 2)   # 均匀分布
r3 = np.random.poisson(5, 2)      # 泊松分布

print('r1=','\n', r1)
print('r2=','\n', r2)
print('r3=','\n', r3)
```

    r1= 
     [-0.08815004  0.13306175  0.10571012  0.15864561  0.22488548]
    r2= 
     [0.52911513 3.43475153]
    r3= 
     [4 6]
    

### 4. 常用数学函数
**步骤 1	三角函数**


```python
a = np.array([0, 30, 45, 60, 90])
b = np.sin(a * np.pi / 180)
c = np.cos(a * np.pi / 180)

print('b=', b)
print('c=', c)
```

    b= [0.         0.5        0.70710678 0.8660254  1.        ]
    c= [1.00000000e+00 8.66025404e-01 7.07106781e-01 5.00000000e-01
     6.12323400e-17]
    

**步骤 2	around函数-四舍五入：**


```python
a = np.array([1.0, 1.5, 2.0, 2.55])
b = np.around(a)
c = np.around(a, decimals=1)

print('b=', b)
print('c=', c)
```

    b= [1. 2. 2. 3.]
    c= [1.  1.5 2.  2.6]
    

**步骤 3	floor/ceil函数：**


```python
a = np.array([1.0, 1.5, 2.0, 2.55])
b = np.floor(a)  # 向下取整
c = np.ceil(a)   # 向上取整

print('b=', b)
print('c=', c)
```

    b= [1. 1. 2. 2.]
    c= [1. 2. 2. 3.]
    

**步骤 4	算数运算**


```python
a = np.array([1, 2, 3, 4])
b = np.array([4, 3, 2, 1])
c = np.add(a, b)       # 加
d = np.subtract(a, b)  # 减
e = np.multiply(a, b)  # 乘
f = np.divide(a, b)    # 除
g = np.mod(a, b)       # 取余
h = np.power(a, b)     # 乘方

print('c=', c)
print('d=', d)
print('e=', e)
print('f=', f)
print('g=', g)
print('h=', h)
```

    c= [5 5 5 5]
    d= [-3 -1  1  3]
    e= [4 6 6 4]
    f= [0.25       0.66666667 1.5        4.        ]
    g= [1 2 1 0]
    h= [1 8 9 4]
    

**步骤 5	统计函数**


```python
a = np.arange(6).reshape(2, 3)
b = np.amin(a, 0)  # 第0维度上最小值
c = np.amax(a,1)   # 第1维度上最大值
d = np.median(a)   # 中位数
e = np.mean(a)     # 平均数

print('a=', a)
print('b=', b)
print('c=', c)
print('d=', d)
print('e=', e)
```

    a= [[0 1 2]
     [3 4 5]]
    b= [0 1 2]
    c= [2 5]
    d= 2.5
    e= 2.5
    


```python
a = np.array([[3,5,1], [2,8,7]])
b = np.sort(a)
c = np.sort(a, axis=0)

print("b:\n", b)
print("c:\n", c)
```

    b:
     [[1 3 5]
     [2 7 8]]
    c:
     [[2 5 1]
     [3 8 7]]
    


```python

```
