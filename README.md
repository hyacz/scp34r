# 1. Scientific computing with python 3 for R users

> Author: hyacz\<hyacz@foxmail.com>  
> Date: 2018-08-06  
> Version: 0.1

## 1.1. 前言

本文旨在帮助有需要且有一定编程基础的 R 语言用户迁移到 Python3，亦可作为两种语言的速查表使用。本文档受到[参考资料[1]](http://mathesaurus.sourceforge.net/r-numpy.html)的启发，在其基础上更新了一些过时的内容，并补充了许多新版本的内容与个人实践的结果。R 和 Python 都是很棒的科学计算工具，我在学习和使用这两种语言上都经历了许多痛苦并快乐着的时光，本文不想去讨论语言优劣，更无意引战。从个人学习历程上而言，我先接触了 Python Web 开发，而后接触到 R 的科学计算，所以本文的会更多的从 Python 出发与 R 做对比。

> [1] http://mathesaurus.sourceforge.net/r-numpy.html
Time-stamp: "2007-11-09T16:46:36 vidar"
©2006 Vidar Bronken Gundersen, /mathesaurus.sf.net
Permission is granted to copy, distribute and/or modify this document as long as the above attribution is retained.

## 1.2. 算术操作

| 描述     | R              | Python            |
| -------- | -------------- | ----------------- |
| 赋值     | `a <- 1`       | `a = 1`           |
| 加法     | `a + b`        | `a + b`           |
| 减法     | `a - b`        | `a - b`           |
| 乘法     | `a * b`        | `a * b`           |
| 除法     | `a / b`        | `a / b`           |
| 幂运算   | `a ^ b`        | `a ** b`          |
| 取余     | `a %% b`       | `a % b`           |
| 整数除法 | `a %/% b`      | `a // b`          |
| 阶乘     | `factorial(a)` | `np.factorial(a)` |

注： python 中 numpy.math.factorial 和 scipy.math.factorial 实际上都是 math.factorial 为了保持引用简洁，在引用 numpy 之后不再显式的调用 math.factorial。

```python
>>> import scipy, numpy, math
>>> scipy.math.factorial.__self__, numpy.math.factorial.__self__, math.factorial.__self__

(<module 'math' from '/usr/.../math.cpython-37m-darwin.so'>,
<module 'math' from '/usr/.../math.cpython-37m-darwin.so'>,
<module 'math' from '/usr/.../math.cpython-37m-darwin.so'>)

```

## 1.3. 逻辑操作

| 描述     | R        | Python   |
| -------- | -------- | -------- |
| 等于     | `a == b` | `a == b` |
| 不等     | `a != b` | `a != b` |
| 大于     | `a > b`  | `a > b`  |
| 小于     | `a < b`  | `a < b`  |
| 大于等于 | `a >= b` | `a >= b` |
| 小于等于 | `a <= b` | `a <= b` |

| 逻辑操作   | a,b类型       | 返回值类型    | R           | Python                         |
| ---------- | ------------- | ------------- | ----------- | ------------------------------ |
| 与         | 布尔变量      | 布尔变量      | `a && b`    | `a and b`                      |
| 或         | 布尔变量      | 布尔变量      | `a || b`    | `a or b`                       |
| 非         | 布尔变量      | 布尔变量      | `!a`        | `not a`                        |
| 与(短路)   | 布尔变量/向量 | 布尔变量      | `a && b`    | `np.all(np.logical_and(a, b))` |
| 或(短路)   | 布尔变量/向量 | 布尔变量      | `a || b`    | `np.any(np.logical_or(a, b))`  |
| 与(逐个)   | 布尔变量/向量 | 布尔变量/向量 | `a & b`     | `np.logical_and(a, b)`         |
| 或(逐个)   | 布尔变量/向量 | 布尔变量/向量 | `a | b`     | `np.logical_or(a, b)`          |
| 异或(逐个) | 布尔变量/向量 | 布尔变量/向量 | `xor(a, b)` | `np.logical_xor(a,b)`          |
| 非(逐个)   | 布尔变量/向量 | 布尔变量/向量 | `!a`        | `~a` or `np.logical_not(a)`    |

注：Python 中没有与 R 中短路操作（ `&&、||` ）直接等价的操作，需通过组合函数。**从上表可知，Python 在处理逻辑变量列表时不要使用原生的逻辑关键字，需使用numpy中的函数。**

## 1.4. 缺失值

| 描述          | R   | Python |
| ------------- | --- | ------ |
| Not a Number  | NaN | np.nan |
| Not Available | NA  | None   |

注：NaN 和 NA 各有各的用法，需要注意如果将 None 混入数字型向量中，这会使得 sum 之类的函数直接报错崩溃;而当向量中存在 np.nan 的时候 sum 之类的操作会得到同为 nan 的结果(和 R 行为相同)。

## 1.5. 一些个人实践

### 1.5.1. 判断矩阵是否可逆

利用 `det() != 0` 来判断矩阵是否可逆是不严谨的，有一些矩阵可能 `det()` 的值不等于 0，但是非常非常小，在理论上该矩阵是可以求逆的，但是由于计算机的精度是有限的，导致该矩阵是 “computationally singular” 的。要计算矩阵的二范数，当矩阵的二范数小于 `1 / 计算精度` 的时候矩阵才是可利用计算机求逆的。

``` R
# R
if (kappa(A) < 1 / .Machine$double.eps) {
    Ai <- solve(A)
}
```

``` python
# Python
import numpy as np
from numpy import linalg as LA

if LA.cond(A) < 1 / np.finfo(A).eps:
    Ai = LA.inv(A)
```

但是直接去计算二范数是比较慢的，考虑到速度问题更为常见的作法是利用 `try ... catch` 的方法来捕获求逆的错误，转而求伪逆。对于可逆的矩阵，直接求逆会比求伪逆在速度上有一定的优势。

``` R
# R
library(MASS)

Ai <- try(solve(A), silent=TRUE)
if (inherits(Ai, "try-error")) {
    Ai <- ginv(Ai)
}
```

``` python
# python
from numpy import linalg as LA

try:
    Ai = LA.inv(A)
except LA.LinAlgError:
    Ai = LA.pinv(A)
```


### 利用 Label 对图像着色

```python
# python
x = df["label"].astype('category').cat.codes    # str to category to int
 
y = df["y"]
t = x % 9
fig, ax = plt.subplots(1, 1)
ax.scatter(x, y, c=t, cmap='Set1')
```

### 进度条

在 python 中推荐使用 https://github.com/tqdm/tqdm 来产生进度条。
