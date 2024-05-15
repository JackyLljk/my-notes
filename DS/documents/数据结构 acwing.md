## 链表

- 不使用动态链表（指针+结构体），因为 new 链表很慢（考虑效率）

- 笔试常用的是**使用数组模拟链表**

### 单链表

> 常用于邻接表（存储树和图）

- 采用**静态链表**的方式，数组表示链表

`e[N]`存放数组的值，`ne[N]`表示 next

<img src="/Users/jk/Desktop/同步/笔记/图片/截屏2023-06-27 15.58.37.png" alt="截屏2023-06-27 15.58.37" style="zoom:33%;" />

（黄框内是上面指针的数组表示）

1. `head`表示头结点的下标（指向链表第一个元素）
   - 头结点指第一个结点，也存放数据
2. `e[i]`表示节点`i`的值
3. `ne[i]`表示节点`i`的`next`指针是多少
4. `idx`存储当前已经用到了哪个点

[实现单链表](https://www.acwing.com/problem/content/828/)





### 双链表

> 用来优化某些问题

> 省略定义头、尾结点，用下标 0 表示头 head，下标 1 表示尾 tail

1. `e[i]`表示结点的值
2. `l[i]`表示 i 前驱结点
3. `r[i]`表示 i 后继结点
4. `idx` 存储当前已经用到了哪个结点



## 栈和队列

> 栈：后进先出，队列：先进先出

### 数组模拟栈

![截屏2023-07-02 11.02.12](../图片/截屏2023-07-02 11.02.12.png)

### 数组模拟队列

<img src="../图片/截屏2023-07-09 10.16.00.png" alt="截屏2023-07-09 10.16.00" style="zoom:50%;" />

### 单调栈

- 给定序列，求序列中每一个数左边离他最近（且满足条件）的数

- 先想暴力做法，再通过性质优化



题型思路：求序列中每一个数左边离它最近，且小于它的数  [单调栈（左侧最近最小）](https://www.acwing.com/problem/content/832/)

1. **思考暴力解法**：双循环枚举（时间复杂度 O(N^2)，直接超时 ^_^）

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

   - 规则：当前为 a[i]，对于栈顶元素 a[tt] 构成逆序时（即 tt < i 且 a[tt] >= a[i]），则弹出栈顶直至栈顶小于 a[i]；

     按照这种方式入栈，得到的是单调栈，且栈顶即是 a[i] 满足题意条件的数

   - 思路：a[i] 左侧的逆序数将永远不会被当做答案（a[i] 用不到，对于 a[i] 右侧的数，a[i] 显然是更好的选择）
   
   - 时间复杂度：每个元素 x 最多只会进栈和出栈一次，最多 2N，即时间复杂度为 O(N)

```cpp
#include <iostream>

using namespace std;

const int N = 1e5 + 10;
int n, tt, stk[N];

int main(void)
{
    scanf("%d", &n);	// 使用 scanf 速度要快几倍
    for(int i = 0; i < n; i++)
    {
        int x;
        scanf("%d", &x);
        while(tt && stk[tt] >= x) tt--;
        if(tt)
            cout << stk[tt] << " ";
        else 
            cout << -1 << " ";
        
        stk[++tt] = x;
    }  
		return 0;
}
```

  

### 单调队列（后续二刷理解）

滑动窗口求最值：输出每次滑动窗口移动后的最大最小值    [滑动窗口](https://www.acwing.com/problem/content/156/)

1. 思考暴力做法：用队列维护窗口，遍历得到最值	O(nk) 

2. 用单调队列优化：考虑队列中没用的元素

   1. 如果存在逆序关系，则先进队列的元素一定不是最小值

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

    for(int i = 0; i < n; i++)	scanf("%d", &a[i]);
        
    int hh = 0, tt = -1;
    
    for(int i = 0; i < n; i++)
    {
      // 判断队头是否已经滑出窗口
        if(hh <= tt && i - k + 1 > q[hh])
            hh++;
        while(hh <= tt && a[q[tt]] >= a[i])
            tt--;
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



## KMP（字符串匹配）

> KMP 是由 Knuth，Morris，Pratt 三人设计的线性时间字符串匹配算法

[题目链接：831.KMP字符串](https://www.acwing.com/problem/content/833/)

### 暴力解法

```cpp
for(int i = 1; i <= n; i++)
{
  bool flag = true;
 	for(int j = 1; j <= m; j++)
  {
    if(s[i+j-1] != p[j])
    {
      flag = false;
      break;
    }
  }
}
```



### 使用 KMP 优化

#### 用到的概念

`s[]`：**模式串**，即被匹配的字符串

`p[]`：**模板串**，用模板串去匹配模式串

**前缀**：非平凡前缀，指除了最后一个字符外，一个字符串的全部头部组合

**后缀**：非平凡后缀，指除了第一个字符外，一个字符串的全部尾部组合

**部分匹配值值**：前缀和后缀的最长共有元素的长度

`next[]`数组：**部分匹配值表**，存储每一个下标对应的部分匹配值



#### next[] 数组详解

`next[]`数组是由对模板串（简称p串）预处理得到

- `next[j]`表示`p[1,j]`的部分匹配值（`p[1,j]`串中前缀和后缀相同字符的最大长度）

- `next[j] = i`，即`p[1,i] = p[j - i + 1, j]`

​	<img src="../图片/image-20230805084949499.png" alt="image-20230805084949499" style="zoom:50%;" />

**求 next[] 数组过程**：通过模板串自己与自己进行匹配得到（匹配过程类似KMP匹配）

```cpp
for(int i = 2, j = 0; i <= n; i++)	// 根据后缀定义，从第二个字符开始匹配
{
  	while(j && p[i] != p[j + 1])	j = ne[j];
  
  	if(p[i] == p[j+1])	j++;	// 下一个位置匹配成功，移动
  
  	ne[i] = j;	// 当前长度的字符串匹配成功，记录已经匹配的长度（即前缀长度 = 后缀长度 = j）
}
```





#### KMP匹配过程

这里 s串和 p串都是从下标1开始

<img src="../图片/image-20230805090500359.png" alt="image-20230805090500359" style="zoom: 33%;" />	

匹配过程如上图所示：

1. 当`s[a,b]`与`p[1,j]`匹配，但下一个位置`s[i] != p[j+1]`时，要移动 p串
2. 区别于暴力解法，p 串并非向后移动1位，而是直接移动到**下次能匹配该部分**的位置（② 对应 ①）
3. 这种移动操作可由`j = next[j]`完成

```cpp
// KMP 匹配过程
for(int i = 1, j = 0; i <= m; i++)
{
		while(j && s[i] != p[j + 1]) j = ne[j];
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
    cin >> n >> p + 1 >> m >> s + 1;	// "+1"表示下标从1开始
    
    // 求next数组过程
    for(int i = 2, j = 0; i <= n; i++)
    {
        while(j && p[i] != p[j + 1]) j = ne[j];
        if(p[i] == p[j + 1]) j++;
        ne[i] = j;
    }
    
    // kmp匹配过程
     for(int i = 1, j = 0; i <= m; i++)
     {
         while(j && s[i] != p[j + 1]) j = ne[j];
         if(s[i] == p[j + 1]) j++;
         if(j == n)
         {
             printf("%d ", i - n);
             j = ne[j];     
         }
     } 
    return 0;
}
```



时间复杂度 O(N)



**参考**

1. 题解：[KMP 字符串](https://www.acwing.com/solution/content/14666/)
2. acwing 算法基础课（数据结构 (一) ）





## Trie 树

> 用来高效地**存储**和**查找字符串集合**的数据结构

[题目链接：Tire字符串统计](https://www.acwing.com/problem/content/837/)

【题目】维护一个字符串集合（仅包含小写英文字母），支持：

（题目一般会限制为小写英文字母/大写英文字母/数字...）

1. 向集合中插入字符串
2. 询问一个字符串在集合中出现了几次

![截屏2023-09-14 10.26.20](/Users/jk/Desktop/同步/笔记/图片/截屏2023-09-14 10.26.20.png)

```cpp
#include <iostream>

using namespace std;

const int N = 1e5 + 10;

int son[N][26];	// 存储Tire树每个点的子节点(最多是26个字母)
int cnt[N];		// 以当前点结尾的字符串的数量
int idx;		 // 当前用到的下标（为0时，既是根节点，又是空节点）
char str[N];

// 插入一个字符串
void insert(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i++)
    {
        int u = str[i] - 'a';	// 将字母映射成 0~25
        if (!son[p][u]) 
          	son[p][u] = ++idx;
        p = son[p][u];
    }
    cnt[p]++;
}

// 查询字符串出现的次数
int query(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) 
        		return 0;
        p = son[p][u];
    }
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



[参考：acwing 算法基础课（数据结构 (二) ）](https://www.acwing.com/file_system/file/content/whole/index/content/4796/)



## 并查集

近乎 O(1) 下快速完成两个操作：

1. 将两个集合合并
2. 询问两个元素是否在一个集合当中

































