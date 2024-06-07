---
title: 堆
date: 2023-12-15 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---

> 手写一个堆

<!--more-->

堆：（STL 堆也支持）

1. 插入一个数
2. 求集合中的最小值
3. 删除最小值
4. 删除任意一个元素
5. 修改任意一个元素

堆是一颗完全二叉树

性质：（小根堆）每个点小于等于左右儿子，根节点是最小元素

对节点下标的性质

堆的存储：用一维数组存储

<img src="/Users/jk/Desktop/Coding/myblog/source/_posts/algo/assets/image-20240228101246463.png" alt="image-20240228101246463" style="zoom:50%;" />

stl里的树是优先队列（priority queue）

**堆的基本操作**：

```cpp
down(x);	// 节点向下调整
// 和两个儿子比较，找最小值，和当前节点的值进行交换
// 直到满足标准堆

up(x);		// 节点向上调整
// 和父节点比较，如果更小，当前节点的值和父节点的值进行交换
```

#### 实现堆的常用操作

size应该就是idx

插入一个数：`heap[++size] = x; up(x);`（插入到最后一个节点，向上调整堆）logn

求最小值：`heap[1]`o(1)

删除堆顶（最小值）：最后一个元素覆盖堆顶元素，向下调整堆顶`heap[1] = heap[size];`

`size--;	down(1);`logn

删除最后一个元素：`size--;`

删除任意一个元素（第k个）：`heap[k] = heap[size]; size--; down(k); up(k);`

(up 和 down 只会执行一个，另一个判断后不会执行，简化判断都写一下)

修改任意一个元素：`heap[k] = x; down(k); up(k);`

#### O(N) 的建堆方式：

从 `n/2`down到1（2/n 的高度是1，即倒数第二层节点），如果从根开始就是O(NlogN)

![image-20240228104332957](/Users/jk/Desktop/Coding/myblog/source/_posts/algo/assets/image-20240228104332957.png)

例题：[Acwing: 堆排序](https://www.acwing.com/problem/content/840/)

```cpp
#include <iostream>
using namespace std;
const int N = 1e5 + 10;

int n, m, num;
int h[N];

void down(int k) {
    int t = k;
    if(k*2 <= num && h[k*2] < h[t])
        t = k * 2;
    if(k*2 + 1 <= num && h[k*2 + 1] < h[t])
        t = k * 2 + 1;
    // 加判断条件
    if(t != k) {
        swap(h[t], h[k]);
        down(t);
    }
}

int main() {
    cin >> n >> m;
    for(int i = 1; i <= n; i++)
        cin >> h[i];
    num = n;
    
    // O(N) 的建堆方式
    for(int i = n/2; i; i--)
        down(i);
        
    while(m--) {
        cout << h[1] << " ";
        h[1] = h[num];
        num--;
        down(1);
    }
    
    return 0;
}
```

也可以补充到排序部分，贴一个链接

```cpp
// up 操作
void up(int u) {
    // u/2 > 0
    while(u/2 && h[u/2] > h[u]) {
        swap(h[u/2], h[u]);
        u /= 2;
    }
}
```

例题：[Acwing: 模拟堆](https://www.acwing.com/problem/content/841/)（带映射版堆）

需要记录是第几个插入的点，更新堆的同时，要更新记录数组的内容（定义全新的交换操作）

```cpp
// 开两个额外的数组存第k个插入的数是什么
void heap_swap(int i, int j) {
	swap(ph[hp[i]], ph[hp[j]]);
    swap(hp[i], hp[j]);
    swap(h[i], h[j]);
}
// 画个图理解
```

strcmp()

代码链接：https://www.acwing.com/activity/content/code/content/45305/



