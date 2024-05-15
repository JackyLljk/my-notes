## 常用函数

copy()

reflect.DeepEqual()







## 标准库 List

[package list](https://studygolang.com/pkgdoc)

`list.List`是 Go 标准库实现的「**双向链表**」

- 链表内的元素类型是相同的
- 可以使用 interface 作为元素

```go
import "container/list"
```

```go
l := list.New()	// 声明

l.PushBack(1)	// 添加到末尾（返回新生成的节点，类型为*list.Element）
l.PushFront(2)	// 添加到表头（返回新生成的节点，类型为*list.Element）

l.Front()	// 返回表头
l.Back()	// 返回表尾

l.Remove(e)	// 删除元素为e的节点

l.Len()

// 遍历
for e := l.Front(); e != nil; e = e.Next() {
    fmt.Println(e.Value)
}

// list.Element 是list的节点
```

