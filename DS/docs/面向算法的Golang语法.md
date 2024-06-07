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

