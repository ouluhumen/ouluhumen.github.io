---
layout:     post   				    
title:      【邱锡鹏】chap1_warmup-numpy 练习题				
subtitle:   
date:       2022-02-07 				
author:     慕念 						
header-img: img/post-bg-computer-ml.jpg	
catalog: true 						
tags:								
    - Machine Learning
---

### numpy的array操作
1. 导入numpy库

```python
import numpy as np
```

2. 建立一个一维数组 a 初始化为[4,5,6]，

- (1)输出 a 的类型（type）
- (2)输出 a 的各维度的大小（shape）
- (3)输出 a 的第一个元素（值为4）

```python
a = np.array([4, 5, 6])
print(a.dtype)
print(a.shape)
print(a[0])
```

3. 建立一个二维数组 b ，初始化为 [ [4, 5, 6],[1, 2, 3]] 

- (1)输出各维度的大小（shape）
- (2)输出 b(0,0)，b(0,1)，b(1,1) 这三个元素（对应值分别为4,5,2）

```python
b = np.array([[4, 5, 6],
              [1, 2, 3]])
print(b.shape)
print(b[0, 0], b[0, 1], b[1, 1])
```

4. (1)建立一个全0矩阵 a ，大小为3x3，类型为整型（提示: dtype = int）

   (2)建立一个全1矩阵 b ，大小为4x5;  

   (3)建立一个单位矩阵 c ，大小为4x4; 

   (4)生成一个随机数矩阵 d ，大小为3x2.

```python
a = np.zeros((3, 3), dtype=int)
b = np.ones((4, 5), dtype=int)
# c = np.eye(4, 4, dtype=int)
c = np.identity(4, dtype=int)
d = np.random.randint(1, 10, dtype=int, size=(3, 2))
print(a)
print(b)
print(c)
print(d)
```

5. 建立一个数组a，(值为[[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]] ) 

- (1)打印a
- (2)输出下标为(2,3)，(0,0) 这两个数组元素的值

```python
a = np.array([[1, 2, 3, 4],
              [5, 6, 7, 8],
              [9, 10, 11, 12]])
print(a)
print(a[2, 3])
print(a[0, 0])
```

6. 把上一题的a数组的 0到1行 2到3列，放到b里面去，（此处不需要从新建立a,直接调用即可）

- (1)输出b
- (2)输出b 的（0,0）这个元素的值

```python
b = a[0:2, 1:3]
print(b)
print(b[0, 0])
```

7. 把第5题中数组a的最后两行所有元素放到 c中，（提示：`a[1:2, :]`）

- (1)输出 c 
- (2)输出 c 中第一行的最后一个元素（提示，使用 -1 表示最后一个元素）

```python
c = a[1:3]
print(c)
print(c[-1][-1])
```

8. 建立数组a,初始化a为[[1, 2], [3, 4], [5, 6]]，输出 （0,0）（1,1）（2,0）这三个元素（提示：使用 `print(a[[0, 1, 2], [0, 1, 0]]) `）

```python
a = np.array([[1, 2], [3, 4], [5, 6]])
print(a[[0, 1, 2], [0, 1, 0]])     # [[想输出的行数下标], [想输出的列数下标]]
```

9. 建立矩阵a，初始化为[[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]，输出(0,0),(1,2),(2,0),(3,1) 

   （提示：使用`b = np.array([0, 2, 0, 1])	print(a[np.arange(4), b])`）

```python
a = np.array([[1, 2, 3], 
            [4, 5, 6], 
            [7, 8, 9], 
            [10, 11, 12]])
b = np.array([0, 2, 0, 1])  #列
print(a[np.arange(4), b])
```

10. 对9中输出的那四个元素，每个都加上10，然后重新输出矩阵a.（提示： `a[np.arange(4), b] += 10` ）

```python
a[np.arange(4), b] += 10
print(a)
```

### array的数学运算
11.  执行 `x = np.array([1, 2])`，然后输出 x 的数据类型

```python
x = np.array([1, 2])
print(x.dtype)
```

12. 执行 `x = np.array([1.0, 2.0])` ，然后输出 x 的数据类类型

```python
x = np.array([1.0, 2.0])
print(x.dtype)
```

13. 执行 `x = np.array([[1, 2], [3, 4]], dtype=np.float64)` ，`y = np.array([[5, 6], [7, 8]], dtype=np.float64)`，然后输出` x+y` 和 `np.add(x,y)`

```python
x = np.array([[1, 2], [3, 4]], dtype=np.float64)
y = np.array([[5, 6], [7, 8]], dtype=np.float64)
print(x + y)
print(np.add(x, y))
```

14. 利用13题目中的 x，y 输出 `x-y` 和 `np.subtract(x,y)`

```python
print(x - y)
print(np.subtract(x, y))
```

15. 利用13题目中的 x，y 输出 `x*y` 和 `np.multiply(x, y)` 还有 `np.dot(x,y)`，比较差异。然后自己换一个不是方阵的试试。

```python
print(x * y)
print(np.multiply(x, y))

# 矩阵乘法
print(np.dot(x, y))

x1 = np.array([[1, 1, 1], [2, 2, 2]])
y1 = np.array([[1,1], [2, 2], [3, 3]])
print(np.dot(x1, y1))
# print(x * y)  不是方阵，所以不能元素相乘
```

16. 利用13题目中的 x，y 输出 `x / y` 。（提示：使用函数 `np.divide()`）

```python
print(x / y)
print(np.divide(x, y))
```

17. 利用13题目中的 x 输出 x 的开方。（提示：使用函数 `np.sqrt()`）

```python
print(np.sqrt(x))
```

18. 利用13题目中的 x，y 执行 `print(x.dot(y))` 和 `print(np.dot(x,y))`

```python
print(x.dot(y))
print(np.dot(x, y))
```

19. 利用13题目中的 x 进行求和。提示：输出三种求和 

- (1)`print(np.sum(x))`
- (2)`print(np.sum(x,axis = 0 ))`
- (3)`print(np.sum(x,axis = 1))`

```python
print( np.sum(x) )
print( np.sum(x, axis = 0))  # 求列和
print( np.sum(x, axis = 1))  # 求行和
```

20. 利用13题目中的 x 进行求平均数，提示：输出三种平均数

- (1)`print(np.mean(x)) `
- (2)`print(np.mean(x,axis = 0))`
- (3)`print(np.mean(x,axis = 1))`

```python
print(np.mean(x))
print(np.mean(x, axis=0))   # 求列平均
print(np.mean(x, axis=1))   # 求行平均
```

21. 利用13题目中的 x ，对 x 进行矩阵转置，然后输出转置后的结果，（提示：`x.T` 表示对 x 的转置）

```python
print(x.T)
```

22. 利用13题目中的 x 求e的指数（提示：函数`np.exp()`）

```python
print(np.exp(x))
```

23. 利用13题目中的 x 求值最大的下标，提示：

- (1)`print(np.argmax(x))`
- (2)`print(np.argmax(x, axis = 0))`
- (3)`print(np.argmax(x, axis = 1))`

```python
print(np.argmax(x))
print(np.argmax(x, axis=0))    # 求列最大值索引
print(np.argmax(x, axis=1))    # 求行最大值索引
```

24. 画图，y=x*x 其中 `x = np.arange(0, 100, 0.1)` （提示：这里用到`matplotlib.pyplot`库）

```python
import matplotlib.pyplot as plt

x = np.arange(0, 100, 0.1)
y = x * x
plt.plot(x, y)
plt.show()
```

25. 画图。画正弦函数和余弦函数，`x = np.arange(0, 3 * np.pi, 0.1)`（提示：这里用到 `np.sin()` `np.cos()` 函数和 `matplotlib.pyplot` 库）

```python
x = np.arange(0, 3*np.pi, 0.1)
y = np.sin(x)
plt.plot(x, y)
z = np.cos(x)
plt.plot(x, z)
plt.show()
```