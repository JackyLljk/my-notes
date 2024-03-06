---
title: 二分法
date: 2023-06-27 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---

> 二分的本质：在区间中找到满足某种性质的**边界**，使得区间二分为“不重叠”的两部分，每次选择答案所在的区间进行下一步处理，当区间长度为1时，即得到查找的答案（或可判断为无解）

有单调性一定可以二分，没有单调性也可以二分（二分本质不是单调性）

<!--more-->

## 整数二分的两个模板

> 两个模板的区别主要是看 mid 属于左边，还是右边，属于右边时 mid = (r + l + 1) / 2！

<img src="https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2023-12-14-090355.png" alt="image-20231211150712625" style="zoom:20%;" />

### 模板一：mid 属于左区间

区间`[l, r]`被划分成`[l, mid]`和`[mid + 1, r]`

```cpp
bool check(int x) {/* ... */} // 检查x是否满足某种性质

int bsearch_1(int l, int r)
{
    while (l < r)
    {
        int mid = l + r >> 1;		// 找中间值 (l+r)/2
        
        // check()判断 mid 是否满足性质
        if (check(mid)) 			
            r = mid;    	// 满足（该情况下左区域包含 mid）
        else 
            l = mid + 1;	// 不满足（从 mid 下一个位置）
    }
    return l;	// 最后 l == r，跳出循环
}
```



### 模板二：mid 属于右区间

区间`[l, r]`被划分成`[l, mid - 1]`和`[mid, r]`

```cpp
bool check(int x) {/* ... */} // 检查x是否满足某种性质

int bsearch_2(int l, int r)
{
    while (l < r)
    {
        int mid = l + r + 1 >> 1;
        // 补 +1，当 l = r-1 时，确保不会死循环（两个模板的主要区分点）
        
        if (check(mid)) 
            l = mid;
        else 
            r = mid - 1;
    }
    return l;
}
```



### **整数二分重要思想**

> 有序数列，查找 --- 选取二分法

1. 先写 `check()` 函数
2. 根据 `check(mid)` 判断 mid 的值在`左 / 右`区间，选取模板
3. 如果无解，记得满足题意

例题：[789. 数的范围](https://www.acwing.com/problem/content/791/)



## 浮点数二分

> 浮点数二分不需要区分边界，每次都是严格取得中间值

- 当最终区间范围足够小时，可认为取得最终答案

```cpp
bool check(double x) {/* ... */} // 检查x是否满足某种性质

double bsearch_3(double l, double r)
{
    const double eps = 1e-6;    // eps 表示精度，取决于题目对精度的要求
    							// 一般是题目要求的精度下探两位
    while (r - l > eps)			// 也可以是循环 100 次
    {
        double mid = (l + r) / 2;
        if (check(mid)) 
            r = mid;
        else 
            l = mid;
    }
    return l;
}
```



## 例题

#### [4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)

**暴力解法**

1. 直接将第二个数组添加到第一个末尾 -> 排序 -> 求中位数 `O(m+n)`
2. 双指针合并两个数组 -> 排序 -> 求中位数`O(m+n)`

**参考解法**

- [参考题解](https://algo.itcharge.cn/Solutions/0001-0099/median-of-two-sorted-arrays/#%E9%A2%98%E7%9B%AE%E9%93%BE%E6%8E%A5)：使用二分法切分两个数组，求中间的值

## 参考

[模板链接](https://www.acwing.com/blog/content/277/)









