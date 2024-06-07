---
title: 前缀和 | 差分
date: 2023-08-20 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---

## 前缀和

### 一维前缀和

> **用法**：快速求原数组中的**一段数的和**，时间复杂度：`O(1)`（不需要使用循环 `O(N)`）

<!--more-->

**一维前缀和定义**：

原数组：`a1, a2, a3, ..., an`（数组下标从`1`开始！）

前缀和数组：`S1, S2, S3, ..., Sn`，其中`Si = a1 + a2 + ... + ai`（下标也是从`1`开始，定义`S0 = 0`）

**求 Si**

`for(int i = 1; i <= n; i++)	Si = Si-1 + ai;`

**求`[l, r]`这一段数的和**：`sum = Sr - Sl-1`		

- 消除边界情况的差异，一般使前缀和从 1 开始，而数组也从下标 1 开始

```cpp
// 模板：求数组 a 中从 l 到 r 的和
int a[N];
int s[N];

// 数组 s 和 a 都是从下标 1 开始存放元素，即 1~n 存放元素
for(int i = 1; i <= n; i++)	
    s[i] - s[i-1] = a[i];	// 前缀和初始化

sum = s[r] - s[l-1];		// 区间和的计算
```

例题：[795. 前缀和](https://www.acwing.com/problem/content/797/)



### 二维前缀和

> **用法**：快速求出**区域矩阵内的元素和**

**求 `Sij`**

<img src="https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-01-11-113000.png" alt="image-20240111192934654" style="zoom: 25%;" />（方框内的点表示元素） 



```cpp
for(int i = 1; i < n; i++)
    for(int j = 1; j < m; j++)
        s[i][j] = s[i-1][j] + s[i][j-1] - s[i-1][j-1] + a[i][j];
```



**求区域和：**

- 即整个区域减去两个矩形区域，再补上重复减去的区域

​	<img src="https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-01-11-223305.png" alt="2024-01-11-223136" style="zoom:25%;" />

```cpp
sum = s[x2][y2] - s[x2][y1-1] - s[x1-1][y2] + s[x1-1][y1-1];
```

```cpp
// 模板：求(x1, y1)和(x2, y2)区域内元素的和
int a[N][N];
int s[N][N];

// 数组 s 和 a 都是从下标 1 开始存放元素，即 1~n 存放元素
// 二维前缀和初始化
for(int i = 1; i <= n; i++)
    for(int j = 1; j <= m; j++)
        s[i][j] = s[i][j-1] + s[i-1][j] - s[i-1][j-1] + a[i][j];

// 求区域元素和
sum = s[x2][y2] - s[x1-1][y2] - s[x2][y1-1] + s[x1-1][y1-1];
```



例题：[796. 子矩阵的和](https://www.acwing.com/problem/content/798/)



## 差分

> 差分是前缀和的逆运算

### 一维差分

#### 定义和用法

**差分定义**

> 这里的`a、b`数组都是从下标`1`开始

1. 原数组：`a1, a2, a3, ..., an`

2. 构造：`b1, b2, b3, ..., bn`

3. 使得：`ai = b1 + b2 + b3 + ... + bi`

- 则 a 是 b 的前缀和，**b 是 a 的差分**

**用法**：给区间`[l, r]`中的每个数加上`c`，求加后序列，使得时间复杂度 `O(1)`（无需遍历）	

- 已知数组`b[n]`，求新的数组`a[1], ..., a[l]+c, a[l+1]+c, ...a[r]+c, a[r+1], ...,a[n]`

**方法**：只需修改：`b[l]+c`和`b[r+1]-c`即可求和

![](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-01-26-111953.png)

1. 修改`b[l]`为`b[l]+c`：使得`a`数组变为`a[1], ..., a[l]+c, a[l+1]+c, ..., a[n]+c`
2. 修改`b[r+1]`为`b[r+1]-c`：使得`a`数组从`a[r+1]`开始删掉加上的`c`

```cpp
// 差分的插入操作
void insert(int l, int r, int c)
{
    b[l] += c;		
    b[r + 1] -= c;
}
```

#### 构造差分数组

```cpp
// 方法一：根据定义进行构造
// b[1] = a[1] - a[0] (a[0] == 0)
// b[2] = a[2] - s[1]
// ...
for(int i = 1; i <= n; i++)
	b[i] = a[i] - a[i-1];
```

```cpp
// 方法二：利用差分数组的插入操作直接进行构造
for(int i = 1; i <= n; i++)
    insert(i, i, a[i]);	// 函数定义在下面

// 理解：相当于直接将初始数组转换为其差分数组
// 即在[1,1]插入a[1]，在[2,2]插入a[2]，...
b[1] = a[1], b[2] = -a[1];
b[2] = a[2] - a[1], b[3] = -a[2];
...
b[i] = a[i] - a[i-1], b[i+1] = -a[i];
...
b[n] = a[n] - a[n-1], b[n+1] = -a[n];	// 成功构造出差分数组 b
```

#### 模板

```cpp
// 1. 定义插入操作
void insert(int l, int r, int c)
{
    b[l] += c;		
    b[r + 1] -= c;
}

// 2. 构造差分数组
int b[N];
for(int i = 1; i <= n; i++)
    insert(i, i, a[i]);

// 3. 进行插入操作
insert(l, r, c);

// 4. 得到改变后的序列，即数组b的前缀和数组，即是要求的新序列数组
for(int i = 1; i <= n; i++)
    b[i] += b[i - 1];
```

例题：[797. 差分](https://www.acwing.com/problem/content/description/799/)



### 二维差分

#### 使用差分矩阵

逆向二维前缀和，原矩阵`a[i][j]`，差分矩阵`b[i][j]`，原矩阵是差分矩阵的前缀和

**用法**：给子矩阵的每个元素加上常数 c，输出更改后的矩阵（时间复杂度达到 `O(1)`）

![](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-01-26-111755.png)

1. 修改`b[x1][y1]+c`，使得`b[x1][y1]`到`b[xn][yn]`都加上`c`
2. 修改`b[x1][y2+1]-c`、`b[x2+1][y1]-c`，使内部蓝色矩形以外的减去`c`
3. 修改`b[x2+1][y2+1]+c`，使红色矩阵补上额外减去的`c`

```cpp
// 差分矩阵的插入操作
insert(int x1, int y1, int x2, int y2, int c)
{
    b[x1][y1] += c;
    b[x1][y2+1] -= c;
    b[x2+1][y1] -= c;
    b[x2+1][y2+1] += c;
}
```

#### 构造差分矩阵

```cpp
// 类似构造差分数组的第二种方法
for(int i = 1; i <= n; i++)
    for(int j = 1; j <= n; j++)
        insert(i, j, i, j, a[i][j]);
```

例题：[798.  差分矩阵](https://www.acwing.com/problem/content/800/)



## 追加练习

[leetcode: 238. 除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/)

- 利用`前缀积*后缀积`解决
- 时间复杂度`O(N)`，额外空间复杂度`O(1)`[题解](https://leetcode.cn/problems/product-of-array-except-self/solutions/11472/product-of-array-except-self-shang-san-jiao-xia-sa)

[leetcode: 560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

- 利用`前缀和` + `哈希`

## 参考

[算法基础课模板](https://www.acwing.com/blog/content/277/)

[算法基础课](https://www.acwing.com/activity/content/introduction/11/)

[差分题解](https://www.acwing.com/solution/content/26588/)

[差分矩阵题解](https://www.acwing.com/solution/content/27325/)
