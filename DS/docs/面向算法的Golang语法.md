## 基本语法

Golang 不支持连续赋值的写法

```go
a = b = c = 1
```

但可以改成下面的写法

```go
a, b, c = 1, 1, 1
```



```go
// 将切片赋值给数组
// slice a
// array b
copy(b, a)	// 底层原理？
// 类似 vector中 b.assign(a.begin(), b.end()); 用法
```



## 内置 sort 排序

- Go 内置的 `sort` 包中提供了一系列排序函数
- 大多数函数是快排的改良版，最坏时间复杂度是 `O(NlogN)`
- [参考](https://learnku.com/articles/38269)

### 整型、浮点数、字符串切片排序

```go
import "sort"

sort.Ints()
sort.Floats()
sort.Strings()
```

```go
s := []int{1, 1, 4, 5, 1, 4}
sort.Ints(s)
```



### 自定义比较器排序

`sort.Slice` 函数：使用一个用户提供的函数对序列进行排序，函数类型为 `func(i, j int) bool`，i、j 为切片索引

`sort.SliceStable` ：在排序时会保留相等元素的原始顺序

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    animals := []struct {
        Name string
        Age int
    } {
        {"dog", 1},
        {"cat", 5},
        {"horse", 3},
    }
    
   	// Age desc
    sort.SliceStable(animals, func(i, j int) bool {
        return animals[i].Age > animals[j].Age
    })
    
    // Age asc, Name Desc
    sort.SliceStable(animals, func(i, j int) bool {
        if animals[i].Age != animals[j].Age {
            return animals[i].Age < animals[j].Age
        }
        return strings.Compare(animals[i].Name, animals[j].Name) == 1
    })
    
    
    for _, v := range animals {
        fmt.Println(v)
    }
    
}
```



### 排序任意数据结构

- 使用 `sort.Sort` 或 `sort.Stable` 函数可以排序实现了 `sort.Interface` 接口的任意类型

```go
package sort

type Interface interface {
    Len() int	// 长度
    Less(i, j int) bool	// 两个元素比较结果
    Swap(i, j int)	// 交换元素的方式
}
```

```go
type Person struct {
    Name string
    Age  int
}

// ByAge 通过对age排序实现了sort.Interface接口
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

func main() {
    family := []Person{
        {"David", 2},
        {"Alice", 23},
        {"Eve", 2},
        {"Bob", 25},
    }
    sort.Sort(ByAge(family))
    fmt.Println(family) // [{David, 2} {Eve 2} {Alice 23} {Bob 25}]

    sort.
}
```





















