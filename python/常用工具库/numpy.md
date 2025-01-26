# numpy

> NumPy(Numerical Python) 是 Python 语言的一个扩展程序库，支持大量的维度数组与矩阵运算，此外也针对数组运算提供大量的数学函数库

NumPy 数组（ndarray）和普通的 Python 列表（list）之间有很多区别，这些区别使得 NumPy 数组在数值计算和科学计算领域中更具优势。以下是一些主要区别：

1. **性能和效率**：
   - NumPy 数组是经过优化的，使用了底层的 C 或 Fortran 代码，因此在数值计算上非常高效。普通的 Python 列表是动态类型，执行起来较慢。
2. **类型一致性**：
   - NumPy 数组要求所有元素的类型一致，这使得数组内的操作更加快速。而 Python 列表可以容纳不同类型的元素。
3. **内存占用**：
   - NumPy 数组通常比 Python 列表更紧凑，占用更少的内存，因为它们存储的是同一类型的数据。
4. **广播功能**：
   - NumPy 数组支持广播，这意味着它们可以在不同形状的数组之间进行数学运算，NumPy 会自动处理维度匹配。
5. **向量化操作**：
   - NumPy 提供了许多向量化操作，允许你在数组上执行复杂的操作而无需显式编写循环。这使得代码更加简洁和高效。
6. **多维操作**：
   - NumPy 数组可以是多维的，而 Python 列表只能是一维的。
7. **丰富的函数库**：
   - NumPy 提供了丰富的数学函数和线性代数操作，可以方便地进行科学计算和数据分析。

总的来说，NumPy 数组是在数值计算和科学计算中更为强大和高效的数据结构，适用于处理大规模的数据集和复杂的数学运算。普通的 Python 列表在更一般的编程任务中更灵活，但在数值计算方面的性能和功能方面相对较弱。



## Numpy 常用方法

#### astype

转换数据类型

```python
import numpy as np

# 创建一个整数数组
int_array = np.array([1, 2, 3, 4, 5])

# 将整数数组转换为浮点数数组
float_array = int_array.astype(float)

print(float_array)  # 输出: [1. 2. 3. 4. 5.]
```



#### reshape 

调整数组形状

```python
import numpy as np

# 创建一个 1D 数组，包含 12 个元素
array_1d = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12])

# 将 1D 数组转换为 2D 数组，形状为 (3, 4)
array_2d = array_1d.reshape(3, 4)

print(array_2d)
# 输出:
# [[ 1  2  3  4]
#  [ 5  6  7  8]
#  [ 9 10 11 12]]
```



## 参考文章

菜鸟教程-numpy：https://www.runoob.com/numpy/numpy-tutorial.html