---
title: 离散化 | 区间和并
date: 2023-11-1 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---

> 离散化：当值域跨度大，但值分布稀疏时，将用到的值的坐标映射到较小空间内使用
>
> 区间和并：快速合并所有有交集的区间

<!--more-->

## 离散化

**离散化**：当值域跨度大，但值分布稀疏时，将用到的值的坐标映射到较小空间内使用，这样只需对映射后的数据结构进行操作即可节省空间开销（把无限空间中的有限个体映射到有限的空间中去）

### 基本步骤

1. 存储操作需要数据的坐标到数组`a`
2. 对数组进行去重
3. 通过原坐标值找到现在的坐标，并进行操作（二分）

### 数组去重

```cpp
// “排序 + 去重”
// 存储所有待离散化的值
vector<int> alls;
// 将所有值排序
sort(alls.begin(), alls.end());
// 去掉重复元素
alls.erase(unique(alls.begin(), alls.end()), alls(end));
// unique(begin, end): 将有序数组去重，将重复元素放在容器末端，返回去重后的尾端点
// erase(begin, end): 删掉数组中的元素（删掉尾端点之后的重复元素）
```

#### 补充：`unique()`实现逻辑

采用双指针算法，找出有序数组中所有满足以下条件之一：

1. 该数是第一个数
2. `a[i] != a[i-1]`

即是要找的不同的数

```cpp
vector<int>::iterator unique(vector<int> &a)
{
    int j = 0;
    for(int i = 0; i < a.size(); i++)
    	if(!i || a[i] != a[i-1])
            // 满足条件后，记录在数组前部分，同时移动指针j
            a[j++] = a[i];
    
    return a.begin() + j; 
}
```



### 题型示例

例题1：[802. 区间和](https://www.acwing.com/problem/content/804/)

```cpp 
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// n次插入，2m次查询(l,m)，数量级都是10^5
const int N = 3e5 + 10;

// 存放数据对(1.坐标、c；2.查询区间)
typedef pair<int, int> PII;

int n, m;

// 存放坐标插入的值，和其前缀和数组
int a[N], s[N];

// 存储所有被访问到的坐标(离散化的值)
vector<int> alls;

// 存储插入和查询操作的数据
vector<PII> add, query;

// 将坐标x映射为离散化下标(求x离散化之后的结果)
int find(int x)
{
    int l = 0, r = alls.size() - 1;
    while(l < r)
    {
        int mid = l + r >> 1;
        if(alls[mid] >= x)   
            r = mid;
        else 
            l = mid + 1;
    }
    return r + 1;
}
 
int main()
{
    cin >> n >> m;
    
    for(int i = 0; i < n; i++)
    {
        int x, c;
        cin >> x >> c;
        add.push_back({x, c});  // 存放插入操作的数据对
        alls.push_back(x);      // 存放插入用的的下标x
    }
    
    for(int i = 0; i < m; i++)
    {
        int l, r;
        cin >> l >> r;
        query.push_back({l, r});    // 存放查询操作的数据对
        alls.push_back(l);          // 存放查询用到的下标
        alls.push_back(r);
    }
    
    // 对alls离散化操作
    // 1. 去重：排序 + 去重
    sort(alls.begin(), alls.end());
    alls.erase(unique(alls.begin(), alls.end()), alls.end());
    
    // 2. 映射坐标，进行相应操作
    for(auto item : add)
    {
        int x = find(item.first);
        a[x] += item.second;
    }
    
    for(int i = 1; i <= alls.size(); i++)
        s[i] = s[i-1] + a[i];
    
    for(auto item : query)
    {
        int l = find(item.first), r = find(item.second);
        cout << s[r] - s[l-1] << endl;
    }    
    
    return 0;
}
```



例题2：[759. 格子染色](https://www.acwing.com/problem/content/761/)



## 区间合并

### **题型**

1. 给定 n 个区间`[li,ri]`，要求合并所有有交集的区间

2. 注意如果在端点处相交，也算有交集（有公共端点）

3. 输出合并完成后的区间个数

### **思路**

1. 按区间左端点排序
2. 以左端点为基准，当前区间与之后的区间有三种情况

![](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-01-26-082418.png)

例题：[803. 区间合并](https://www.acwing.com/problem/content/805/)

```cpp
// segs 存放所有区间的首、尾端点
void merge(vector<PII> &segs)
{
    vector<PII> res;
    
    // 按左端点进行排序
    sort(segs.begin(), segs.end());
    
    // 设定为最边界（范围边界*2）
    int begin = -2e9, end = -2e9;
    for(auto seg : segs)
        // 1. 满足上图无交集的情况，且不是初始值时
        if(end < seg.first)
        {
            if(begin != -2e9)
                res.push_back({begin, end});	// 保存答案
            begin = seg.first, end = seg.second;	// 更新当前区间
        }
    	// 2. 满足上图中①、②情况，更新为区间并集
        else
            end = max(end, seg.second);
    
    // 保存最后begin-end维护的区间
    if(begin != -2e9)
        res.push_back({begin, end});
        
    segs = res;
}
```



## 参考

[算法基础课模板](https://www.acwing.com/blog/content/277/)

[算法基础课](https://www.acwing.com/activity/content/introduction/11/)











