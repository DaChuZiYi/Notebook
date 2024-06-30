---
title: 第二套模拟题目解题笔记
date: 2024-03-09 19:57:49 +0800
categories: [Python,算法]
tags: [Python,算法]
---

# 题目

[题目链接🔗](http://oj.kechuangweilai.com/#/examination?contest_id=1039)

## 1.密码验证

创建一个密码验证功能。

验证的条件是：

- 长度应大于*6*；
- 应至少包含一个数字。

**输入**
一个字符串(str)

**输出**
一个逻辑值 (bool)

**样例输入**
样例1：`Short`

样例2：`1234856`

**样例输出**
样例1：`False`

样例2：`True`

### 我的思路

根据题目,有2个条件

1. 长度大于6
2. 包含数字

根据[is开头的函数表](https://dachuziyi.github.io/Notebook/posts/%E7%AC%AC%E4%B8%80%E5%A5%97%E9%A2%98%E7%9B%AE%E7%AC%94%E8%AE%B0/#2.%E5%AF%86%E7%A0%81%E9%AA%8C%E8%AF%81)

长度可以用`len()`

循环字符串的每个字符,如果有任何一个为数字就满足第二点

### 我的解法

```python
# 输入
pwd = input("输入")
# 默认判断不包含数字
IFhasN = False
# 遍历字符串
for i in pwd:
    # 如果为数字，则满足第二点
    if i.isdigit():
        IFhasN = True
# 输出布尔值,用and连接2个条件
print(len(pwd) >= 6 and IFhasN)
```

---

## 2.阿姆斯特朗数

如果一个n位正整数等于其各位数字的n次方之和，则称该数为阿姆斯特朗数。

例如`13 + 53 + 33 = 153`。

1000 以内的阿姆斯特朗数：`1, 2, 3, 4, 5, 6, 7, 8, 9, 153, 370, 371, 407`。

输入一个数，判断这个数是否是阿姆斯特朗数，是输出`True`，否输出`False`。

**样例输入**

样例1：`9`

样例2：`98`

**样例输出**

样例1：`True`

样例2：`False`

### 我的思路

首先获取输入数字的位数，使用`len()`

再循环取出各个位数的数字，`**位数`

判断结果是否与原数字相等，输出布尔值

### 我的解法

```python
# 输入
num = int(input("输入"))
# 获取长度
n = len(str(num))
# 设置默认结果
end = 0
# 循环(长度)次，获取每一位数
for i in range(n):
    # 第一次i为0，单独处理
    if i == 0:
        temp = (num % 10)** n
        end += temp
    else:
        # 获取每一位数并**位数
        temp = ((num // (10 ** i)) % 10)** n
        end += temp
# 输出判断结果的布尔值
print(end == num)
```

---

## 3.杨辉三角

杨辉三角又称*Pascal三角形*，它的第`i+1`行是`(a+b)*i`的展开式的系数。

它的一个重要性质是：*三角形中的每个数字等于它两肩上的数字相加。*

以下是杨辉三角形的前4行：

```
1
1 1
1 2 1
1 3 3 1
```

输入一个n，输出它的前n行。

**样例输入**

样例1：`4`

样例2：`6`

**样例输出**

样例1：

```
 1
 1 1
 1 2 1
 1 3 3 1
```

样例2：

```
 1
 1 1
 1 2 1
 1 3 3 1
 1 4 6 4 1
 1 5 10 10 5 1
```

### 我的思路

首先输入行数，第一行总是为1

其它行则按照每一行的需要的数字字数的多少与上一行进行计算

判断是否为一行中的第一个或是最后一个，是的话则为一

将所有数字添加到一个列表中，用`join()`输出

### 我的解法

```python
# 初始化
last = []
Temp_last = []
# 输入
n = int(input("输入"))
# 设置需要输出的行数
for time in range(n):
    # 第一行输出 1
    if time == 0:
        print("1")
    # 其它行按照规律
    else:
        # 设置本行需要输出几位数字
        for i in range(time+1):
            # 开头或者结尾为 1
            if i == 0 or i == time:
                Temp_last.append("1")
            # 否则为前一行的两位数字之和
            else:
                Temp_last.append(str(int(last[i-1])+int(last[i])))
        # 将当前行设置为上一行
        last = Temp_last
        # 重置当前行
        Temp_last = []
        # 输出
        print(" ".join(last))
```

### 延伸

若需要以等腰三角的形式输出，则可以用以下方式

```python
# 初始化
last = []
Temp_last = []
end = []
# 输入
n = int(input('输入'))
# 设置需要输出的行数
for time in range(n):
    # 第一行为 1
    if (time == 0):
        end.append("1")
        # 其它行按照规律
    else:
        # 设置本行需要几位数字
        for i in range((time + 1)):
            # 开头或者结尾为 1
            if (i == 0 or i == time):
                Temp_last.append('1')
                # 否则为前一行的两位数字之和
            else:
                Temp_last.append(str((int(last[(i - 1)]) + int(last[i]))))
        # 将当前行设置为上一行
        last = Temp_last
        # 重置当前行
        Temp_last = []
        # 存储当前行
        end.append(" ".join(last))

# 若有结果，则输出
if len(end) > 0:
    # 循环设置需要的空格数
    for i in range(len(end),0,-1):
        # 输出空格
        for i2 in range(i-1):
            print(" ",end="")
        # 空格后添加相应的一行
        print(end[0 - i])
```

> 请注意，此方法仅适用于每个数字都是个位数的情况，若为多位数，则会出现以下情况：
>
> ```
>          1
>         1 1
>        1 2 1
>       1 3 3 1
>      1 4 6 4 1
>     1 5 10 10 5 1
>    1 6 15 20 15 6 1
>   1 7 21 35 35 21 7 1
>  1 8 28 56 70 56 28 8 1
> 1 9 36 84 126 126 84 36 9 1
> ```
