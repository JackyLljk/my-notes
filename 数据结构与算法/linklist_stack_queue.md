---
title: 数组模拟：单双链表/栈/队列 | 单调栈/队列
date: 2023-11-25 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---

> 数组模拟：链表（单链表、双链表）、栈、队列， 单调栈，单调队列

<!--more-->

## 数组模拟链表

> 不使用动态链表（指针+结构体），因为 new 链表很慢（考虑效率）

- 笔试常用的是**使用数组模拟链表**

### 单链表

> 常用于邻接表（存储树、图）

- 采用**静态链表**的方式，数组表示链表

- `e[N]`存放数组的值，`ne[N]`表示`next`指针

![image-20240127091825249](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-01-27-011851.png)

1. `head`表示头结点的下标（指向链表第一个元素）
2. `e[i]`表示节点`i`的值
3. `ne[i]`表示节点`i`的`next`指针是多少
4. 空节点下标指向`-1`
5. `idx`存储当前已经用到了哪个点 

```cpp
int head, e[N], ne[N], idx;

// 初始化
void init()
{
    head = -1;
    idx = 0;    // 都没用到，指向头结点
}

// 头插法
void add_to_head(int x)
{
    ne[idx] = head;
    head = idx;
    e[idx] = x;
    idx++;
}

// 插入到第k个结点之后
void add(int k, int x)
{
    e[idx] = x;
    ne[idx] = ne[k];
    ne[k] = idx;
    idx++;
}

// 删掉第k个结点后面的结点
void remove(int k)
{
    ne[k] = ne[ne[k]];
}
```

例题：[826. 实现单链表](https://www.acwing.com/problem/content/828/)



### 双链表

> 用来优化某些问题

 **省略定义头、尾结点，用下标 0 表示头 head，下标 1 表示尾 tail**

1. `e[i]`表示结点的值
2. `l[i]`表示`i`前驱结点
3. `r[i]`表示`i`后继结点
4. `idx` 存储当前已经用到了哪个结点

```cpp
int e[N], l[N], r[N], idx;

// 初始化
void init()
{
    // 0表示左端点，1表示右端点
    // 初始情况：
    r[0] = 1, l[1] = 0;
    
    // 前两个已经被占用了
    idx = 2;
}

// 在第k个点右插x
void add(int k, int x)
{
    r[idx] = r[k], l[idx] = k;
    l[r[k]] = idx;
    r[k] = idx;
    e[idx] = x;
    idx++;
}

// 删除第k个点
void remove_k(int k)
{
    r[l[k]] = r[k], l[r[k]] = l[k];
}
```

例题：[827. 双链表](https://www.acwing.com/problem/content/829/)



> 栈：后进先出，队列：先进先出

## 数组模拟栈

```cpp
int stk[N], tt;	// tt 表示栈顶下标

// 插入
stk[++tt] = x;

// 删除
tt--;

// 判空
if(tt > 0)	// not empty
else		// empty

// 得到栈顶元素
stk[tt];
```



### 单调栈

> 常见题型：给定序列，求序列中每一个数左边（或右边）离他最近（或满足其他条件）的数

例题： [单调栈（左侧最近最小）](https://www.acwing.com/problem/content/832/)

问题：求序列中每一个数左边离它最近，且小于它的数 

- 先想暴力做法，再通过性质优化

1. **思考暴力解法**：双循环枚举（时间复杂度`O(N^2)`，直接超时）

```cpp
#include <iostream>
using namespace std;
const int N = 1e5 + 10;

// 双循环枚举暴力解法
int n, a[N];

int main(void)
{
    cin >> n;
    for(int i = 0; i < n; i++)
        cin >> a[i];
    
    for(int i = 0; i < n; i++)
    {
        bool flag = false;
        // 枚举每个数左边的数
        for(int j = i - 1; j >= 0; j--)	
            if(a[j] < a[i])
            {
                cout << a[j] << " ";
                flag = true;
                break;
            }
        if(!flag) cout << -1 << " ";
    }
    return 0;
}
```

2. **思考单调栈**：在遍历数组的同时，用栈存储当前位置左侧的元素

**规则**：当前为`a[i]`，对于栈顶元素`a[tt]`构成逆序时（即` tt < i 且 a[tt] >= a[i]`），则弹出栈顶直至栈顶小于`a[i]`

- 按照这种方式入栈，得到的是单调栈，且栈顶即是`a[i]`满足题意条件的数

**思路**：`a[i]`左侧的逆序数将永远不会被当做答案（`a[i]`用不到，对于`a[i]`右侧的数，`a[i]`显然是更好的选择）

- 时间复杂度：每个元素`x`最多只会进栈和出栈一次，最多`2N`，即时间复杂度为`O(N)`

```cpp
#include <iostream>
using namespace std;
const int N = 1e5 + 10;

int n, tt, stk[N];

int main(void)
{
    scanf("%d", &n);	
    for(int i = 0; i < n; i++)
    {
        int x;
        // 使用 scanf 而非 cin，优化读入速度
        scanf("%d", &x);
        
        // 栈非空，且栈顶大于当前数 -> 弹出（永远不会被用到）
        while(tt && stk[tt] >= x) 
            tt--;
        // 栈非空，则栈顶即是“x 左边最近、小于它的数”
        if(tt)
            cout << stk[tt] << " ";
        // 否则，没有这样的数
        else 
            cout << -1 << " ";
        
        // 当前值入栈
        stk[++tt] = x;
    }  
	
    return 0;
}
```





## 数组模拟队列

```cpp
// 队头 hh，队尾 tt（初始化指向-1，也可以指向0）
int q[N], hh, tt = -1;	

// 队尾插入
q[++tt] = x;

// 队头弹出
hh++;

// 判空
if(hh <= tt)	// not empty
else			// empty
    
// 取出队头元素
q[hh]
```



### 单调队列（后续二刷理解）

> 常见题型：滑动窗口求最值，输出每次滑动窗口移动后的最大最小值  

例题：[154. 滑动窗口](https://www.acwing.com/problem/content/156/)

**暴力做法**：用队列维护窗口，遍历得到最值`O(nk) `（k 是窗口的大小）

- 每次移动窗口：1. 新元素插入队尾；2. 从队头弹出元素

**单调队列优化**：考虑队列中“**没用**”的元素

1. 如果存在逆序关系，则先进队列的元素一定不是最小值（序小值大）
2. 对于得到的单调队列，队头即是最小元素

```cpp
#include <iostream>
using namespace std;
const int N = 1e6 + 10;

int n, k;
int a[N], q[N];

int main(void)
{
    scanf("%d%d", &n, &k);
    for(int i = 0; i < n; i++)	
        scanf("%d", &a[i]);
    
    // 默认队尾指针指向 -1
    int hh = 0, tt = -1;
    
    for(int i = 0; i < n; i++)
    {
        // 这里队列存储的是数组a的下标
    	// 判断队头是否已经滑出窗口：
        // 1. 判空 2. 判断队头是否滑出窗口（i-k+1表示窗口的第一个元素）
        if(hh <= tt && i - k + 1 > q[hh])
            hh++;	// ?
        
        // 弹出队列中“不小于”当前值的元素
        while(hh <= tt && a[q[tt]] >= a[i])
            tt--;
        
        // 插入当前值
        q[++tt] = i;
        if(i >= k - 1) 
            cout << a[q[hh]] << " ";
    }
    
    puts("");
    
    hh = 0, tt = -1;
    
    for(int i = 0; i < n; i++)
    {
        if(hh <= tt && i - k + 1 > q[hh])
            hh++;
        while(hh <= tt && a[q[tt]] <= a[i])
            tt--;
        q[++tt] = i;
        if(i >= k - 1) 
            cout << a[q[hh]] << " ";
    }
    
    puts("");
    
    return 0;
}
```

 
