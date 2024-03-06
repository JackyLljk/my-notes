---
title: 并查集
date: 2023-12-10 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---

**并查集用来快速（近乎O(1)）处理：**

1. 将两个集合合并（并）

2. 询问两个元素是否在一个集合当中（查）

<!--more-->

## 基础并查集

基本原理：每个集合用一棵树来表示，树根的编号就是整个集合的编号

- 每个节点存储它的父节点，`p[x]`表示`x`的父节点

```cpp
/* 解决问题 */

// 问题1：如何判断树根（除了树根，p[x]都是存放的父节点）
if(p[x] == x)
    
// 问题2：如何求 x 的集合编号（查询所在集合）
while(p[x] != x)
    x = p[x];	// 最后得到树根的编号，即该集合的编号

// 问题3：如何合并集合（将另一个集合作为该集合树根的一个子节点）
// px是x的集合编号，py是y的集合编号
p[y] = x;	// 把y插入到x的根节点
```

- 路径压缩 △：遍历根节点后，将所有路径上的节点的父节点设置为根节点
- 按秩合并

```cpp
#include <iostream>
using namespace std;
const int N = 1e5 + 10;

int p[N];

// 返回x的祖宗节点 + 路径压缩（核心操作）
int find(int x) {
    if(p[x] != x)
        p[x] = find(p[x]);
    return p[x];
}

int main() {
    int n, m;
    cin >> n >> m;
    
    // 每个节点的父节点都指向自己（即每个节点都初始化为根节点）
    for(int i = 0; i < n; i++)
        p[i] = i;
        
    while(m--) {
        char op;
        int a, b;
        cin >> op >> a >> b;
        // 合并操作
        if(op == 'M') 
            // a集合的根节点的父亲指向b集合的根节点
            p[find(a)] = find(b);
        else 
            if(find(a) == find(b))
                cout << "Yes" << endl;
            else
                cout << "No" << endl;   
    }
    return 0;
}
```

例题：[Acwing: 836. 合并集合](https://www.acwing.com/problem/content/838/)



## 变形

**扩展情况**：维护一些额外信息（如：每个集合中元素个数）

**连通块**：

只需保证根节点的size是有意义的

合并集合的同时

```cpp
size(b) += size(a);
p[a] = p[b];
```

特判：a和b在同一个集合中时，不需要再次合并和更新size

```cpp
#include <iostream>

using namespace std;

const int N = 1e5 + 10;

int n, m;
int p[N], num[N];

int find(int x) {
    if(p[x] != x)
        p[x] = find(p[x]);
    return p[x];
}

int main() {
    cin >> n >> m;
    
    for(int i = 0; i < n; i++) {
        p[i] = i;
        num[i] = 1;
    }
    
    while(m--) {
        string op;
        int a, b;
        cin >> op;
        if(op == "C") {
            cin >> a >> b;
            // 需要特判
            if(find(a) == find(b))
                continue;
            // 记录被连通的根节点的集合数
            num[find(b)] += num[find(a)];
            p[find(a)] = find(b);
        }
        else if(op == "Q1") {
            cin >> a >> b;
            if(find(a) == find(b))
                puts("Yes");
            else 
                puts("No");
        }
        else {
            cin >> a;
            cout << num[find(a)] << endl;
        }
    }
    
    return 0;
}
```

例题：[Acwing: 连通块中点的数量](https://www.acwing.com/problem/content/839/)

练习题：[Acwing: 食物链](https://www.acwing.com/problem/content/242/)

维护每个节点到根节点的距离













