---
title: KMP字符串匹配
date: 2023-08-05 10:20:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---



> KMP 是由 Knuth，Morris，Pratt 三人设计的线性时间字符串匹配算法

<!--more-->

[模板题目链接：831.KMP字符串](https://www.acwing.com/problem/content/833/)

## 暴力解法

```cpp
for(int i = 1; i <= n; i++)
{
  bool flag = true;
 	for(int j = 1; j <= m; j++)
        // 看作 i + (j-1)
    	if(s[i+j-1] != p[j])
    	{
      		flag = false;
      		break;
    	}
}
```

- 怎么理解`i+j-1`？



## 使用 KMP 优化

### 用到的概念

`s[]`：**模式串**，即被匹配的字符串

`p[]`：**模板串**，用模板串去匹配模式串

**前缀**：非平凡前缀，指除了最后一个字符外，一个字符串的全部头部组合

**后缀**：非平凡后缀，指除了第一个字符外，一个字符串的全部尾部组合

**部分匹配值**：前缀和后缀的最长共有元素的长度

`next[]`数组：**部分匹配值表**，存储每一个下标对应的部分匹配值



### next[] 数组详解

`next[]`数组是由对模板串（简称p串）预处理得到

- `next[j]`表示`p[1,j]`的部分匹配值（`p[1,j]`串中前缀和后缀相同字符的最大长度）

- `next[j] = i`，即`p[1,i] = p[j - i + 1, j]`

![next数组](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-25-083238.png)

**求 next[] 数组过程**：通过模板串**自己与自己进行匹配**得到（匹配过程类似KMP匹配）

```cpp
// 根据后缀定义，从第二个字符开始匹配
for(int i = 2, j = 0; i <= n; i++)	
{
  	while(j && p[i] != p[j + 1])	
        j = ne[j];
  
    // 下一个位置匹配成功，移动
  	if(p[i] == p[j+1])	
        j++;	
  
    // 当前长度的字符串匹配成功，记录已经匹配的长度（即前缀长度 = 后缀长度 = j）
  	ne[i] = j;	
}
```



### KMP匹配过程

这里 s串和 p串都是从下标1开始

![kmp 匹配](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-25-065344.png)

匹配过程如上图所示：

1. 当`s[a,b]`与`p[1,j]`匹配，但下一个位置`s[i] != p[j+1]`时，要移动 p串
2. 区别于暴力解法，p 串并非向后移动1位，而是直接移动到**下次能匹配该部分**的位置（② 对应 ①）
3. 这种移动操作可由`j = next[j]`完成

```cpp
// KMP 匹配过程
for(int i = 1, j = 0; i <= m; i++)
{
    while(j && s[i] != p[j + 1]) 
        j = ne[j];
  	// 1. j != 0 表示j没有退回起点，并非从头开始匹配
  	// 2. s[i] != p[j+1] 说明当前位置的下一个字符并不匹配
  	// j = ne[j] 调用next[j]，将模板串后移到后缀位置
  
  	if(s[i] == p[j + 1]) j++;
  	// 下一个位置匹配成功
  
    if(j == n)	// 整个模板串匹配成功
    {
        printf("%d ", i - n);	// 匹配成功
        j = ne[j];		// 继续匹配下一个位置
    }
}
```



### 完整代码

```cpp
#include <iostream>

using namespace std;
const int N = 1e5 + 10, M = 1e6 + 10;

int n, m;
char p[N], s[M];
int ne[N];  // next数组（以防当作保留字报错）

int main()
{
    // 数组名+1，加上一个数组元素的字节数，即下标为1的元素的地址
    // "+1"表示下标从1开始
    cin >> n >> p + 1 >> m >> s + 1;	
    
    // 求next数组过程
    for(int i = 2, j = 0; i <= n; i++)
    {
        while(j && p[i] != p[j + 1]) 
            j = ne[j];
        if(p[i] == p[j + 1]) 
            j++;
        ne[i] = j;
    }
    
    // kmp匹配过程
     for(int i = 1, j = 0; i <= m; i++)
     {
         while(j && s[i] != p[j + 1]) 
             j = ne[j];
         if(s[i] == p[j + 1]) 
             j++;
         if(j == n)
         {
             printf("%d ", i - n);
             j = ne[j];     
         }
     } 
    return 0;
}
```

时间复杂度： O(N)



## **参考**

[题解：KMP 字符串](https://www.acwing.com/solution/content/14666/)

[算法基础课](https://www.acwing.com/activity/content/introduction/11/)































