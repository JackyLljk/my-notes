---
title: 高精度
date: 2023-08-15 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]
---
> C++ 选手需要处理大整数运算

**C++ 存储大整数的方式**：将每一位存储到数组中，下标`0`元素对应个位、`1`元素对应十位...，依次类推（高位存储在数组末端，方便在运算时进行进位操作）

<!--more-->

**常用运算模板**

- **A + B 型**（两大整数相加）：`（位数）len(A) <=  10^6`，`len(B) <=  10^6`
- **A - B 型**（两大整数相减）：`len(A) <=  10^6`，`len(B) <=  10^6`
- **A * b 型**（大整数乘小整数）：`len(A) <=  10^6`，`b <= 10^9`
- **A / b 型**（大整数除小整数）



## 从题中读入、输出大整数

```cpp
// 已知输入为两行，分别为大整数 A 和 B
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    // 大整数用字符串先读入
    string a, b;
    vector<int> A, B;
    
    cin >> a >> b;
    
    // 存入 vector（e.g. "123456" -> [6,5,4,3,2,1]）
    for(int i = a.size() - 1; i >= 0; i--)
        // 存数字，字符要减去偏移量'0'
        A.push_back(a[i] - '0');
    for(int i = b.size() - 1; i >= 0; i--)
        B.push_back(b[i] - '0');
    
    // 进行操作
    auto C = func(A, B);
    
    // 读入和输出都是从最高位开始
    for(int i = C.size() - 1; i >= 0; i--)
        printf("%d", C[i]);
    
    return 0;
}
```





## A + B 型

`Ci = (Ai + Bi + ti)%10, ti = (Ai-1 + Bi-1)/10`

```cpp
// C = A + B, A >= 0, B >= 0
vector<int> add(vector<int> &A, vector<int> &B)
{
    if(A.size() < B.size())	return add(B, A);
    
    vector<int> C;
    int t = 0;
    for(int i = 0; i < A.size(); i++)
    {
        // 加上进位，如果有B[i]也加上
        t += A[i];
        if(i < B.size()) t += B[i];
        
        // 取余为i位的值，保存进位t
        C.push_back(t % 10);
        t /= 10;
    }
    
    if(t) C.push_back(t);
    
    return C;
}
```

例题：[791.高精度加法](https://www.acwing.com/problem/content/793/)



## A - B 型

> 先判断 A 和 B 的大小，选择运算次序

```cpp
// 判断 A >= B 
bool cmp(vector<int> &A, vector<int> &B)
{
    if(A.size() != B.size()) return A.size() > B.size();
    
    for(int i = A.size() - 1; i >= 0; i--)
        if(A[i] != B[i])
            return A[i] > B[i];
    
    return true;	// 包含A == B的情况
}

// C = A - B, A >= B, A >= 0, B >= 0
vector<int> sub(vector<int> &A, vector<int> &B)
{
    vector<int> C;
    
    // 已保证 A >= B
    for(int i = 0, t = 0; i < A.size(); i++)
    {
        // 减去上一位借位
        t = A[i] - t;
        
        // 判断 B 是否有这一位
        if(i < B.size()) 
            t -= B[i];
        
        // 精妙处理！：综合处理两种情况：1. t>=0, 即t；2. t<0, 借一位
        C.push_back((t + 10) % 10);
        
        // 判断是否需要借一位
        if(t < 0) t = 1;
        else t = 0;
    }
    
    // 去掉高位存在的0
    while(C.size() > 1 && C.back() == 0)
        C.pop_back();
    
    return C;
}
```

例题：[792. 高精度减法](https://www.acwing.com/problem/content/794/)



## A * b 型

```cpp
// C = A * b, A >= 0, b >= 0
vector<int> mul(vector<int> &A, int b)
{
    vector<int> C;
    
    int t = 0;
    
    // "||t" 条件表示进位 t 还没处理完
    for(int i = 0; i < A.size() || t; i++)
    {
        if(i < A.size())
 			// 如果A还没处理完：t = 上一位的进位 + A[i]*b
            t += A[i] * b;
        
        // 每次只考虑取余得到的个位和进位（下一位的个位）
        C.push_back(t % 10);
        t /= 10;
    }
    
    // 去掉高位存在的0
    while(C.size() > 1 && C.back() == 0)
        C.pop_back();
    
    return C;
}
```

例题：[793. 高精度乘法](https://www.acwing.com/problem/content/795/)



## A / b 型

```cpp
// C = A / b 余 r, A >= 0, b > 0
// 余数 r 用的是实参（答案可能要用到）
vector<int> div(vector<int> &A, int b, int &r)
{
    vecotr<int> C;
    r = 0;
    
    for(int i = A.size() - 1; i >= 0; i--)
    {
        // 按照除法的逻辑：上一位的余数补在A[i]的左边一位
        r = r * 10 + A[i];
        
        // 保存当前位，以及余数 r
        C.push_back(r / b);
        r %= b;
    }
    
    // C的开头存储的是数字的高位，需要调转
    reverse(C.begin(), C.end());
    
    // 删掉高位的0
    while(C.size() > 1 && C.back() == 0)
        C.pop_back();
    return C;
}
```

例题：[794. 高精度除法](https://www.acwing.com/problem/content/796/)



## 参考

[算法基础课模板](https://www.acwing.com/blog/content/277/)

[算法基础课](https://www.acwing.com/activity/content/introduction/11/)























