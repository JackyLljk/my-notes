---
title: Go 基础学习
date: 2023-11-27 12:00:00
tags: [编程语言, Go]
categories: [编程语言]
---

> Golang 启动!!!!!

**参考博客**：[Go语言学习之路/Go语言教程](https://www.liwenzhou.com/posts/Go/golang-menu/)

<!--more-->

## 知识点大纲/要点

### [变量和常量](https://www.liwenzhou.com/posts/Go/var-and-const/)

1. **变量声明方式**：标准声明，批量声明
2. **初始化**（可以一次初始化/赋值多个变量，`a, b = 4, 3.1415926`）
3. **类型推导**
4. **短变量声明**（`:=`，只能在函数内使用）【常用】
5. **匿名变量**（`_`）
6. **常量**（`const`，通常定义在全局）
7. **`iota`常量计数器**（跳过、插队、定义数量级、多个`iota`）



### [基本数据类型](https://www.liwenzhou.com/posts/Go/datatype/)

1. **整型**：有符号、无符号，特殊整型（不使用的情况） 
2. **数字字面量**：十进制、八进制（0开头，`%o`）、十六进制（0x开头，`%x`），二进制（十进制打印为二进制`%b`)
3. **浮点数**：`float32`，`float64`（`%f`，`%.2f`）
4. **布尔值**：默认`false`，不允许从整型强制转换，无法参与数值运算
5. **字符串**：原生数据类型，`""`内
    - 字符串转义符
    - 多行字符串`''`内
    - 修改字符串
    - 常用操作：`len(str)`，拼接`+`，分隔，`str.contains`，前后缀判断，子串位置（第一个下标），`join`操作
6. **复数**
7. **字符**`byte`, `rune`：`''`内，`%c`
8. **强制类型转换**：`T()`



### [fmt](https://www.liwenzhou.com/posts/Go/fmt/)

1. **输出**：
    - `Print`，`Printf`，`Println`
    
    - 写入文件：`Fprint`
    
    - 返回字符串：`Sprint`
    
    - `Eprint`
    
2. **格式化占位符**

3. **输入**：

    - `Scan`，`Scanf`，`Scanln`
    - `bufio.NewReader`
    - `Fscan`
    - `Sscan`



### [运算符](https://www.liwenzhou.com/posts/Go/operators/) 

1. **类似 C++**
2. **`--`，`++`是单独的一行语句**



### [流程控制](https://www.liwenzhou.com/posts/Go/control-flow/)

1. **if 条件判断**：特殊写法（局部变量声明 + 判断）
2. **for 循环**（省略初始和结束语句（类似`while`），无限循环）
3. **`for range`键值循环**
4. **`switch case`**
5. **`continue`，`break`**
6. **`goto`**



### [数组](https://www.liwenzhou.com/posts/Go/array/)

```go
var a [元素数量]T	// T：数据类型，数组大小不可变，不同元素数量的数组是不同的类型
```

1. **数组初始化：**
    - 初始化列表`[2]string{"abc", "bcd"}`
    - 编译器推测长度`[...]bool{}`
    - 索引值方式`a := [...]int{1: 1, 3: 5}`

2. **遍历数组**：`len(array)`得到数组的长度
    - for 循环遍历
    - for range 循环遍历
3. **二维数组及其遍历**

> 多维数组只有第一层可以使用`...`让编译器推测数组长度，内部的`{}`以`,`为结尾

4. **数组是值类型**：赋值和传参会拷贝数组，改变副本不影响本身



### [切片](https://www.liwenzhou.com/posts/Go/slice/)

> 切片是引用类型，基于数组做的一层封装，内部结构包含地址、长度size、容量cap

#### **切片定义方式：**

> 切片初始化后才能使用！

1. **`var a []T`**

2. **基于数组得到切片**：对切片再次切片：`s := a[low:high]`（简单切片表达式，完整切片表达式`a[low, high, max]`）

`cap = max - low`；`low`可以缺省为 0

```go
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]		// {2, 3, 4}：左包含，右不包含
s1 := a[:]		// 从头切到尾
s2 := s[1:2]	// {3}
// 对切片再次切片时，high的上限变成被切切片的容量
```

3. **make 函数构造**：`s := make([]T, size, cap)`



#### 切片的使用

1. **`len()`获得切片长度，`cap()`获得切片容量**

2. **切片的本质**
3. **切片不能直接比较**：`s == nil`表示切片没有底层数组

```go
// 同样长度、容量为0的切片
var s1 []int	// s1 == nil，没有申请内存，需要初始化
var s2 []int{}	// s2 != nil，已经初始化
```

5. **判空：**`len(s)==0`

6. **切片的赋值拷贝**：切片赋值给切片后，二者公用一个底层数组

7. **遍历切片（同数组）**



#### 切片的其他操作

1. **动态添加元素：**`s = append(s, 元素1, 元素2)`，底层数组容量不够时可以自动扩容（重新申请更大的一块内存）

可以追加切片：`s1 = append(s1, s2...)`

> ! 扩容前后的内存地址是不同的

2. **切片的扩容策略**

3. **使用`copy()`复制切片**：`copy(s2, s1)`，s2 使用另一个切片空间

4. **从切片中删除元素**：`s = append(s[:index], s[index+1:]...)`



### [map](https://www.liwenzhou.com/posts/Go/map/)

> map 是无序的基于`key-value`的数据结构，是引用类型，初始化后才能使用

#### map 的定义

> 推荐初始化map时直接指定容量，只有初始化map后才能继续后续操作

1. **定义map**：`map[keyType]valueType`
2. **初始化map：**默认初始值为`nil`，`make(map[keyType]valueType, [cap])`分配内存



#### 用法

1. **添加键值对**
2. **声明时填充元素**
3. **判断是否存在某个键**：`value, ok := map[key]`，`ok`为`bool`类型
4. **遍历**：`for range`遍历，可以同时遍历`key, value`也可以只遍历`key`（遍历顺序与添加顺序**无关**）
5. **删除键值对：**`delete(map, key)`
6. **按指定顺序遍历**：使用切片将`key`排序
7. **元素为map的切片**，**值为切片的map**：



### [函数](https://www.liwenzhou.com/posts/Go/function/)

#### 定义函数

```go
func 函数名(参数) 返回值 {	// 多个返回值需要用"()"包裹，","分隔
    ...
}
```

#### 参数

1. **类型简写**
2. **可变（数量）参数**：`x ...int`（是切片类型），可变参数放在最后

#### 返回值

1. **多个返回值**：也支持类型简写
2. **返回值命名**：在函数定义处定义返回值，不要忘了`return`
3. **返回**`nil`

#### 函数进阶

1. **作用域**
2. **函数类型：**类型为`func()`，可以作为变量使用（赋值给变量，**作为另一个函数的参数**，作为返回值）
3. `defer`**语句**：延迟处理语句（类似栈），处理资源、文件、异常等
4. **匿名函数**：没有函数名`func(参数) (返回值) {}`，用于实现回调函数和闭包
    - 赋值给变量
    - 立即执行
5. **闭包**：函数 + 外层变量的引用
6. **内置函数**



### [指针](https://www.liwenzhou.com/posts/Go/pointer/)

> Go 指针不能进行偏移和运算，是安全指针

1. **重要符号**：取地址`&`，根据地址取值`*`
2. **指针类型**：`*T`，（占位符）`%p`
3. **内存分配**：`new`**、**`make`：
    - `new(Type)`返回该类型的指针（较少使用）
    - `make`只用于`slice`、`map`、`channel`



### [结构体](https://www.liwenzhou.com/posts/Go/struct/)

**自定义类型**：`type MyInt int`，定义具有`int`特性的新的类型，打印类型时会有`MyInt`类型

**类型别名**：`type byte = uint8`，打印类型时仍然是原类型

```go
var a byte = 10
fmt.Printf("%T", a)		// uint8
```

#### 结构体基本使用

> 结构体成员没有初始化，默认为类型零值，结构体是值类型

1. **结构体定义**：`type 类型名 struct { ... }`

2. **实例化**：`var a structName`，使用`.`访问字段
3. **匿名结构体**：`var 实例名 struct{ ... }`
4. **指针类型结构体**：可以直接用`.`访问，而不用取值（“语法糖”）
5. **取结构体地址实例化**

6. **初始化**：键值对初始化，值列表初始化

7. **内存布局**：结构体占用一块连续内存，**空结构体**不占空间，[Go 结构体的内存对齐](https://www.liwenzhou.com/posts/Go/struct-memory-layout/)

8. **构造函数**：是自己实现的构造函数，且返回值一般是结构体指针类型（降低开销）



#### 方法和接受者

1. **方法**：可以为`自定义类型`或`结构体类型`定义方法
2. **接受者**：指针类型接受者（对接受者直接做操作），值类型接受者（拷贝副本）



#### 结构体进阶

> 在为结构体内`slice`和`map`字段赋值时根据实际需要，考虑是否需要拷贝后再赋值

1. **匿名字段**
2. **嵌套结构体**：可以结合匿名字段使用
    - 可以直接访问内部匿名结构体的字段 
    - 模拟**”继承**“
3. **字段的可见性**：定义的标识符首字母大写（公开），小写（私有，仅当前包可见）
4. **JSON 序列化与反序列化**：json 包无法访问首字母小写的字段
5. **结构体标签 Tag**

```go
// 字段必须按照标准写法，由一对反引号包裹，key-value之间不要有空格，例如：
ID     int    `json:"id"`
```



### [包](https://liwenzhou.com/posts/Go/package/)

1. **包的定义**：（简单理解为放`.go`的文件夹），包的声明，`main`包
2. **包的引入**：`import packageName "path/packageName"`，packageName 可以替换为**别名**
3. **可见性**：除了对包外可见的变量，首字母都要小写
4. **匿名导入**
5. `init`**初始化函数**：全局声明 -> `package.init()` -> `main.init()`

- 不允许导入包但不使用，不允许循环引入包



### [接口](https://www.liwenzhou.com/posts/Go/interface/)

> Go 语言的接口是一种抽象的类型，是一组方法的集合

1. 接口不管你是什么类型，只管你要实现什么方法

```go
type 接口类型名 inference {
    // 方法名大写，且接口类型名大写时，该方法可以被其他包的代码访问
    方法名1(参数列表1) 返回值列表1		
    方法名2(参数列表2) 返回值列表2
    ...
}
```

2. **使用接口**
3. **值接受者、指针接受者**（只有类型指针能够保存到接口变量中）实现接口
4. **一个类型可以实现多个接口**
5. **接口组合**
6. **空接口**：可以存储任意类型的值（作为函数参数、map的值）
7. **接口值**：类型 + 值
8. **类型断言**：`v, ok := x.(T)`，可以猜测接口变量`x`的类型`T`
9. [error接口](https://www.liwenzhou.com/posts/Go/error/)



### [文件操作](https://www.liwenzhou.com/posts/Go/file/)

> 用到标准库`os`，打开文件->读取文件内容->关闭文件

1. **打开和关闭文件**：`os.Open()`，`file.close()`，记得要关闭文件！
1. **读取文件**：`file.Read()`

```go
// 打开文件
file, err := os.Open("./xx")
if err != nil {
    fmt.Println("open file failed!, err: ", err)
    return
}

// 关闭文件
defer file.Close()

// 循环读取
var content []byte
var tmp = make([]byte, 128)
for {
    n, err := file.Read(tmp)
    if err == io.EOF {	// end of file
        fmt.Println("文件读完了")
        break
    }
    if err != nil {
        fmt.Println("read file failed, err: ", err)
        return
    }
    // 读取 n byte
    content = append(content, tmp[:n]...)
}
fmt.Println(string(content))
```

3. `bufio`
4. **读取整个文件**：`import "io/ioutil", ioutil.ReadFile()`，不要一次性读取大文件
5. **写入操作**：`os.OpenFile()`打开文件，三种写入操作
    - `Write()`，`WriteString()`
    - `bufio.NewWriter()`
    - `ioutil.WriteFile()`
6. 获取当前路径：`str, _ := os.Getwd()`



### [反射](https://www.liwenzhou.com/posts/Go/reflect/)

> Go 语言中变量分为两部分：类型信息（预先定义好的元信息），值信息（程序运行过程中可动态变化的），反射用到内置`reflect`包，反射可以在运行时获取反射信息（包括类型信息、值信息等）

#### 反射基础

1. **反射的作用**：在程序**运行期间**对程序本身进行访问和修改

2. **获取任意值的类型对象**：`reflect.TypeOf()`，类型对象为`*reflect.rtype`、

    - 类型还可以划分为类型`Name`和种类`Kind`

    ```go
    type magicGirl struct{} 
    var a magicGirl
    t := reflect.TypeOf(a)	// t.Name(): magicGirl; t.Kind(): struct
    ```

3. **获取变量的值信息**：`reflect.ValueOf()`，值的类型为`reflect.value`

    - 通过`.Kind()`获取值对应的类型
    - 通过方法以不同的类型返回该值
    - 通过反射修改变量的值：`Elem()`获取指针对应的值，e.g.`v.Elem().SetInt(100)`
    - `isNil()`、`isValid()`

#### 结构体反射

1. **获得结构体成员信息**：`NumField()`获取索引, `Field()`获取字段信息等
2. **描述结构体字段信息**：`StructField`
3. **获取结构体方法**
4. **反射的优缺点**



### [并发](https://www.liwenzhou.com/posts/Go/concurrence/)

> Go 语言采用的并发模型是 CSP，提倡通过**通信共享内存**，而不是通过共享内存而实现通信（如设置全局变量）

#### 基本概念

**串行**：多个任务按顺序执行，完成一个后才能进行下一个任务

**并发**：同一**时间段**内执行多个任务

**并行**：同一**时刻**执行多个任务

进程、线程、协程



#### goroutine

> Goroutine 是 Go 程序中最基本的并发执行单元，每个 Go 程序都至少包含 main goroutine，Go 语言不需要自己实现进程、线程、协程，并发只需开启一个 goroutine

1. `go`**关键字**：在函数或方法前加上`go`关键字
2. `sync.WaitGroup`保证`main goroutine`执行结束前，其他goroutine全部执行完毕 
    - 创建goroutine执行函数需要一定开销
    - 匿名函数闭包并发问题

```go
// goroutine demo
var wg sync.WaitGroup	// 定义全局等待组变量

func hello() {
	fmt.Printf("迎接Stesins;Gate")
    wg.Done()	// 通知wg，计数器-1
}

// 开启main goroutine执行main函数
func main() { 
    // 计数器+1
	wg.Add(1)
    // go 关键字，开启一个goroutine执行函数
	go hello() 
	fmt.Printf("跨越1%%的高墙\n")
    // 停顿程序，等待另一线程也能执行完毕，该方式并不准确
	// time.Sleep(time.Second)	
    
    // 等待所有其他goroutine运行结束
	wg.Wait()	
}
```

3. **动态栈**：自动为goroutine分配合适的栈空间
4. `goroutine`**调度**
5. `runtime.GOMAXPROCS`：设置程序并发时占用的CPU逻辑核心数
6. 小结
    - 一个操作系统线程对应用户态多个goroutine
    - go程序可以使用多个操作系统线程
    - goroutine和OS线程是多对多的关系（`m:n`）
7. `work pool`（二刷补充）



#### channel

- Go 语言中的通道`channel`是一种特殊的引用类型，可以让一个goroutine发送特定值到另一个goroutine，并且遵循`FIFO`保证收发数据的顺序

1. `channel`**类型**：`var 变量名 chan 元素类型`
    - 元素类型表示通道传递元素的类型
2. `channel`**零值**：`nil`（引用类型）
3. **初始化**：`make(chan 元素类型, [缓冲大小])`
    - 缓冲区大小可选（大小表示最多存放几个值）
4. **操作**：发送，接收，关闭
    - 通道值可以被垃圾回收掉的，关闭不是必须

```go
ch <- 10	// 发送到通道
x := <= ch	// 从通道接收并赋值给x
<-ch		// 从通道接收
close(ch)	// 关闭通道(可以从关闭的有值通道中取值)
```

5. **无缓冲通道**：阻塞的通道/同步通道，必须有一个接收方才能发送成功
6. `len`，`cap`
7. **多返回值接收**：`value, ok := <- ch`
8. `for range`**接收值**

```go
for v := range ch {
    fmt.Println(v)	// 值被接收完后自动退出循环
}
```

9. **单向通道**

- `<- chan int` 只接收通道
- `chan <- int`只发送通道

10. `select case`**多路复用**：从多个通道取值，都满足条件则随机取值

**通道问题总结**

- 对**已关闭的**通道执行`close`会引发`panic`
- 不能简单通过`len(ch)`判断通道是否被关闭
- 对**关闭通道**发送值会引发`pannic`
- 对**关闭的**无值通道执行接收操作会得到类型零值
- 对**关闭的**有值通道可以一直接受到通道为空

![image-20240215164703578](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-15-084713.png)



#### 并发安全和锁

> 多个goroutine同时操作一个资源（临界区），即**竞态问题**

1. **互斥锁**：`sync.Mutex.Lock()`，`sync.Mutex.Unlock()`
    - 多个goroutine同时等待一个锁时，唤醒的策略是`随机的？`
2. **读写锁**：`sync.RWMutex`
3. `sync.WaitGroup`：是一个结构体，`Add()`、`Done()`、`Wait()`
4. `sync.Once`：限制在并发场景下只使用一次，单例模式
5. `sync.Map`：并发安全的map
6. **原子操作**：`atomic`包

#### [处理并发错误](https://www.liwenzhou.com/posts/Go/error-in-goroutine/)



### [网络编程](https://www.liwenzhou.com/posts/Go/socket/)

- OSI七层模型

- `socket`**抽象层**：用户只需调用Socket相关函数，而将复杂的TCP/IP协议当做“黑盒”

#### TCP通信

1. **TCP服务端**：
    - 监听端口
    - 接收客户端请求建立连接
    - 创建goroutine处理连接
2. **TCP客户端**：
    - 建立与服务端的连接
    - 进行数据收发
    - 关闭连接
3. **TCP黏包**：（后续研究）



#### UDP通信（后续研究！）

> 用户数据报协议，是无连接、不可靠、无时序的通信

1. **UDP服务端**
2. **UDP客户端**



## 补充/提示

### 要点补充

- 单行注释、多行注释（同 C++）

- 花括号不能单独另起一行；语句末不需要分号
- 可在全局声明和初始化变量，但不能单独赋值
- `go fmt`可以自动格式化代码，使风格统一，会自动调用
- 使用驼峰命名法
- 首字母大写的变量/结构体是在其他包内可见的，按规范写法需要写注释进行说明
- 区别 Go 的函数和方法
- Go 语言需要初始化的引用类型：`slice`， `map`，`channel`



### 常用快捷键 GoLand

1. 切换窗口：`control + tab` / `ctrl + tab`
2. 直接换行：`⇧ + enter`





## 问题记录/待补充

1. **other declaration of main**：在 Go 中，一个目录（文件夹）就是一个包，一个包只能有一个给定名称的函数（除了`init()`），不能在同一目录下出现多个同名函数

2. <img src="https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-01-09-080438.png" alt="image-20240109152320918" style="zoom: 50%;" />

3. map 为什么是无序的

4. 切片和数组的区别

5. go 语言函数传参永远是值拷贝，函数内部拿到的是副本，修改实参需要传入指针

6. `tmp, ok := `用法总结

7. socket与门面模式

8. 错误处理为什么要有一个return？

9. go变量的默认初始值







