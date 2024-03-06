---
title: Trie 树（字典树）
date: 2023-08-10 12:00:00
tags: [数据结构, 算法]
categories: [数据结构与算法]

---

> Trie 树（字典树）是用来高效地**存储和查找字符串集合**的数据结构

<!--more-->

## 模板题目

[模板题目链接：835. Tire字符串统计](https://www.acwing.com/problem/content/837/)

【题目】维护一个字符串集合（仅包含小写英文字母），支持：

> 题目一般会限制为小写英文字母/大写英文字母/数字...

1. 向集合中插入字符串
2. 询问一个字符串在集合中出现了几次

## 图解 Trie 树

![截屏2023-09-14 10.26.20](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-20-031524.png)

```cpp
#include <iostream>

using namespace std;

const int N = 1e5 + 10;

// 存储Tire树每个点的子节点(最多是26个字母)是第几个节点，没有则为0
int son[N][26];	
// 以当前点结尾的字符串的数量
int cnt[N];	
// 当前插入的节点是第几个（为0时，既是根节点，又是空节点）
int idx;	
// 存储字符串
char str[N];	

// 插入一个字符串
void insert(char *str)
{
    int p = 0;
    // 字符串结尾为'\0'，可以用str[i]判断是否走到结尾
    for (int i = 0; str[i]; i++)
    {
        // 将字母映射成 0~25
        int u = str[i] - 'a';	
        
        // 当结点不存在子节点u时，创建节点
        if (!son[p][u])
          	son[p][u] = ++idx;
        p = son[p][u];
    }
    // 记录以该字母结尾的数量
    cnt[p]++;
}

// 查询字符串出现的次数
int query(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        // 不存在满足的子节点，说明没有该字符串
        if (!son[p][u]) 
        		return 0;
        p = son[p][u];
    }
    // 满足字符串顺序，返回之前记录的值
    return cnt[p];
}

int main(void)
{
	int n;
  	scanf("%d", &n);
  	while(n--)
  	{
    	char op[2];
    	scanf("%s%s", op, str);
    	if(op[0] == 'I')
      		insert(str);
    	else
      		printf("%d\n", query(str));
  	}
  
	return 0; 
}
```

### 对 idx 的理解

1. `idx`相当于分配了一个`son[idx]`节点存储下标（类似单链表的`idx`）
2. `son[idx][u]`记录该下标对应的子节点的下标
3. 当查询时，根据`son[idx][u]`即可按照插入的字符串**顺序**查找到字符串出现的次数



[补充例题：143. 最大异或对](https://www.acwing.com/problem/content/145/)：Tire 树存储整数



## 参考

[算法基础课](https://www.acwing.com/activity/content/introduction/11/)

[理解 idx（评论是精髓）](https://www.acwing.com/solution/content/5673/)