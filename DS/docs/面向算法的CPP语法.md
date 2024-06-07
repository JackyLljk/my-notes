---
title: 面向算法的 Cpp 用法
date: 2023-12-22 12:00:00
tags: [数据结构, 算法, 编程语言, C++]
categories: [数据结构与算法]
---

> 求解算法题需要用到的那些 Cpp 语法

<!--more-->

## STL 及常用类型

### vector 变长数组

> 系统为某一程序分配空间时，所需时间与空间大小无关，而与申请的次数有关

- 类似数组，但可以`动态扩展`（找更大的内存空间，将原数组拷贝，释放原空间）
- 倍增思想

头文件：`#inlude <vector>`

#### 定义方式

```cpp
vector<T> v;			// 模板实现
vector<int> a(10, 3);	// 长度为10，元素都初始化为3
vector<int> a[10];		// vector数组，即10个vector
```

#### 使用

```cpp
// 判空和返回大小基本上STL都有
v.empty();		// 判空
v.size();		// 返回元素个数（时间复杂度为 O(1)）

v.clear();		// 清空

v.push_back(elem);		// 尾部插入元素 elem
v.pop_back();			// 删除最后一个元素
v.front();	// 返回容器第一个元素
v.back();	// 返回容器最后一个元素

// 迭代器
v.begin();	// 第0个数
v.end();	// 最后一个数的后面一个数（v.[size()]）

// 支持比较运算
？
```

**使用技巧**

1. 可以用”`[下标]`“操纵容器（支持随机寻址）
2. 在创建 vector 时如果没有指定长度，则 vector 中不存在任何元素，无法被读取
3. 仅当 vector 某位置已经存在元素时，才可以使用下标进行操作
4. 使用`v.size()`获取大小时，返回的类型为`unsigned int`

#### 遍历方式

```cpp
for(int i = 0; i < v.size(); i++) 
    cout << a[i] << endl;

for(vector<int>::iterator i = v.begin(); i != v.end(); i++)	
    cout << *i << endl;

for(auto x : a) 
    cout << x << endl;
```

？迭代器？指针？

v.end()是什么类型、值？

**理解 resize() 和 reserve()**

https://blog.csdn.net/liuweiyuxiang/article/details/88692708



### pair 存储二元组

```cpp
// 将两个数据组成一组数据
pair<T1, T2> p;

p.first;	// 返回第一个元素
p.second;	// 返回第二个元素

typedef pair<T1, T2> PII;	// 简化声明

// 初始化
pair<int, int> p;
p = {1, 2};
p = make_pair(3, 4);

// 支持比较运算（按字典序？），first为第一关键字，second为第二关键字

// 存多个值
pair<int, pair<int>> p3;
```

- 使用`sort()`进行排序时，首先以左端点进行排序，仍然相同的话再以右端点进行排序



### string

```cpp
// 支持用 + 拼接
string str = "hello ";
str += "cpp";	// "hello cpp"

str.size();		// 返回元素个数
str.length();	// 同上
str.empty();	// 判空
str.clear();	// 清空

str.substr(1, 2);	// 从下标1开始，返回2个字母的子串（子串长度）
// 第二个参数省略或超出索引范围，都会返回从1开始的所有子串

str.a_str();	// 返回str字符数组的起始地址
```



### queue 队列

头文件：`#include <queue>`

```cpp
queue<int> q;

q.size();
q.empty();
// 没有清空函数，清空：
q = queue<int>();

q.front();	// 返回队头
q.back();	// 返回队尾
q.push();	// 入队
q.pop();	// 出队
```



### priority_queue 优先队列

> 实现原理是堆，默认为**大根堆**

头文件：`#include <queue>`

```cpp
priority_queue<int> heap;
heap.push();
heap.pop();
heap.top();	// 返回堆顶元素
```

#### 转化为小根堆

**方法1**：存入时存的是负数

**方法2**：

```cpp
// 定义小根堆
priority_queue<int, vector<int>, greater<int>> heap;
```



### stack 栈

```cpp
stack<int> stk;
stk.push();
stk.pop();
stk.top();	// 返回栈顶元素
```



### deque 双端队列

头文件：`#include <deque>`

```cpp
deque<int> dq;
dq.clear();
dq.front();
dq.back();
dq.push_back();
dq.push_front();	// 从头插入

dq.begin();
dq.end();

// 支持随机寻址
dq[i];
```



### set，multiset

头文件：`#include <set>`

- `set`没有重复元素
- `multiset`没有重复元素

```cpp
set<int> s;
multiset<int> ms;

s.clear();
s.insert(i);	// 插入
s.count();
s.find(k);		// 返回查找到 key=k 元素的迭代器（不存在，则返回end()）
s.erase(x);		// 删除所有x，k + logN
s.erase(iterator);	// 删除该迭代器
s.lower_bound();	// 返回大于等于x的最小的数的迭代器
s.upper_bound();	// 返回大于x的最小的数的迭代器
```



### map，multimap

头文件：`#include <map>`

```cpp
map<string, int> a;
a.insert(x);	// x是一个pair
a.erase(x);		// x是pair或迭代器
a.find();

// 可以像数组一样用 map
a["jk"] = 1;	// logN 而不是 O(1)
a["jk"]; 		// 查找

a.lower_bound();	
a.upper_bound();	
```

迭代器++/--：O(logN)



### 哈希表

> unordered_set，unordered_map，unordered_multiset，unordered_mulimap

记得引入头文件

- 增删改查时间复杂度是`O(1)`
- （内部无序）不支持`a.lower_bound() / a.upper_bound()`	
- 不支持迭代器`++/--`



### bitset

> 压位（省八位空间）

```cpp
bitset<num> s;	// 定义长度为num的bitset

// 支持所有位运算
// []
count();	// 返回1的个数
```







## 数据处理方法与技巧

### 选择高效的读入方式

> **经验**：当输入比较多时，选择高效的读入方式

#### 选择`printf/scanf`，而非`cout/cin`

- 当 C++ 读读取大量数据时，使用前者甚至可以**提速十倍**
- 尤其是将`cin`替换为`scanf`，读取速度可以显著提高

```cpp
// printf 输出 用法
printf("字符串");
printf("格式控制符1 输出控制符2..."，输出参数1，输出参数2,...);
// 格式控制符，输出参数的个数一一对应
printf("格式控制符 非输出控制符", 输出参数);

// scanf 输入 用法
scanf("格式控制符", &输入参数);	// 不要忘记"&"
	// 变量前有 & 表明“放到以变量的地址为地址的变量中”
```

#### 优化`cout/cin`

```cpp
// 在main函数内添加如下代码之一
ios::sync_with_stdio(false);	// code1

cin.tie(0);	// code2
```



### 浮点数处理

**经验**：保留 k 位小数，精度下探两位

**用法**：使用`cout`输出，保留固定位数的小数

```cpp
/* 写法一 */
#include <iomanip>	// 需要使用的头文件

cout << setiosflags(ios::fixed) << setprecision(6) << a << endl;
// 输出保留6位小数的变量a

/* 写法二 */
printf("%.6lf", a);		// 更简洁
```

```cpp
// 使用scanf输入double
scanf("%lf", &a);	
```



### 字符串处理

#### 字符串处理函数

```cpp
// substr() 字符串指定长度复制
str.substr(pos, len)
// 返回值 string，包含str中从pos开始的len个字符的拷贝
// pos 的默认值为 0，len 的默认值为 s.size() - pos（默认拷贝整个字符串）
```

语法题：[左旋转字符串](https://www.acwing.com/problem/content/74/)

语法题：[把字符串转换成整数](https://www.acwing.com/problem/content/83/)

**经验**：

1. 字符类型的`0`对应于 ASCII 码`48`，获得整数需要`-'0'`
2. 字符数组存储的元素是数字的判断语句

`str[k] >= '0' && str[k] <= '9'`

3. 可以用`1e11`为界限判断`int`类型是否越界
4. 当结果可能超出范围时，选取更大的类型，并返回时进行强制转换

#### 字符串与字符数组

**字符串类型**：

https://blog.csdn.net/ksws0292756/article/details/79432329

1. `string`
2. `char*`：指向字符串首个字母的指针
3. `char[]`





## 常用辅助函数和特殊值

### 常用辅助函数

```cpp
#include <algorithm>	// 算法头文件

std::max(a, b);			// 返回参数 a 和 b 的最大值
std::min(a, b);			// 返回参数 a 和 b 的最小值
std::swap(a, b);		// 交换参数 a 和 b 的值

```

```cpp
#include <cmath>	// 头文件

pow(4, 2);		// 4 的平方
sqrt(4);		// 4 开方
abs(b-a);		// 绝对值

reverse(first, last);	// 反转 [first，last) 范围内元素的顺序
vector<int> v = {5,4,3,2,1};
reverse(v.begin(), v.end());	// v = {1,2,3,4,5}
string str = "www.mathor.top";
reverse(str.begin(), str.end());	// str = "pot.rohtam.wwww"

// begin(), end() 函数，可用于 string，vector,...
v.begin();	// 返回指向首个元素的指针
v.end();	// 返回指向最后一个元素“之后位置”的指针
```



### 特殊值

#### 最大最小值

```cpp
// 最大最小int
int a = INT_MIN;
int b = INT_MAX;
```





## 等待补充

[puts/gets vs cout/cin](https://blog.csdn.net/qq_29486343/article/details/46993567)



## 参考

