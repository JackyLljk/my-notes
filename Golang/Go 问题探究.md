## 简短问答

### 1. 值类型和引用类型

**值类型**：

- 值类型：`int系列`，`float系列`，`bool`，`string`，`数组`，`结构体`
- 变量直接存储值，在**栈**中分配内存

**引用类型**：

- 引用类型：`指针`，`slice`，`channel`， `interface`，`map`，`函数`
- 变量存储的是地址，地址对应在**堆**中分配的内存空间，该空间内存储的是值

<br>

### 2. 空结构体 struct{} 有什么用法

`struct{}`在 Go 中可以作为**占位符或标记**使用，因为不包含任何字段，所以在内存中**不占用任何空间**

**作为占位符**

```go
// 使用空结构体作为通道元素类型，表示只关注通道的事件，而不需要传输任何数据
ch := make(chan struct{})

// 在map中使用，只关注键时使用，节省掉值占用的内存空间
set := make(map[string]struct{})
```

<br>


## 数组与切片

### 1. 数组和切片的区别 

**相同点**：存储的元素类型，下标访问、len、cap

1. 只能存储一组相同类型的数据结构

2. 都是通过下标来访问，并且有容量长度，长度通过 len 获取，容量通过 cap 获取

**区别**：定长/扩容，类型不同、底层数组、扩容与数组

1. 数组是定长，访问和复制不能超过数组定义的长度，否则就会下标越界，切片长度和容量可以自动扩容

2. 数组是值类型，切片是引用类型，每个切片都引用了一个底层数组，切片本身不能存储任何数据，都是这底层数组存储数据，所以修改切片的时候修改的是底层数组中的数据。切片一旦扩容，指向一个新的底层数组，内存地址也就随之改变

**简洁的回答**：

1. 定义方式不一样
2. 初始化方式不一样，数组需要指定大小，大小不改变 
3. 在函数传递中，数组切片都是值传递。

**数组的定义**

```go
var a1 [3]int
var a2 [...]int{1,2,3}
```

**切片的定义**

```go
var a1 []int
var a2 :=make([]int,3,5)
```

**数组的初始化**

```go
a1 := [...]int{1,2,3}
a2 := [5]int{1,2,3}
// [2]int 和 [3]int 是不同的类型（长度是类型的一部分）
```

**切片的初始化**

```go
b:= make([]int,3,5)
```

**截取切片**

```go
// 下标low开始，截取到high-1的位置，最大容量是max（max小于底层数组容量）
slice = arr[low:high:max]	
```

**补充**

1. 数组未赋值元素默认初始值为0
2. 数组是一片连续的内存，切片实际上是一个结构体

```go
type slice struct {
    array unsafe.Pointer	// 底层数组指针
    // 指向的是底层数组的
    len int	// 长度
    cap int	// 容量
}
```

3. 对切片扩容后，会另开空间，底层数组地址会发生变化
4. [对切片的切片进行扩容，对原切片的影响](https://golang.design/go-questions/slice/vs-array/)（函数传递都是值传递，但内外也可能是不同的底层数组）
5. `[n]*T`是指针数组，`*[n]T`是数组指针

<br>


### 2. slice 的扩容策略

```go
// 1.21.5 (1.18 之后的新策略)
func growslice(oldPtr unsafe.Pointer, newLen, oldCap, num int, et *_type) slice {
	...
    // 第一步：
	newcap := oldCap
	doublecap := newcap + newcap
    // 1. 判断新长度是否大于两倍旧容量，大于则设置容量为新长度
	if newLen > doublecap {
		newcap = newLen
	} else {
        // 2. 否则设置阈值256，旧容量小于阈值，则新容量为旧容量两倍
		const threshold = 256
		if oldCap < threshold {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
            // 3. 如果旧容量大于阈值，开始循环增加新容量，直到大于新长度
            // 扩增公式：newcap += (newcap + 3*threshold) / 4
			for 0 < newcap && newcap < newLen {
				// Transition from growing 2x for small slices
				// to growing 1.25x for large slices. This formula
				// gives a smooth-ish transition between the two.
				newcap += (newcap + 3*threshold) / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = newLen
			}
		}
	}
    ...
    // 第二步（细节见源码部分）
}
```

**第一步：获得新容量 newcap**：

1. 判断新长度是否大于两倍旧容量，大于则设置容量为新长度
    - 对`nil`切片添加元素，容量变为新长度
2. 否则设置阈值256，旧容量小于阈值，则新容量为旧容量两倍
3. 如果旧容量大于阈值，开始循环增加新容量，直到大于新长度
    - 扩增公式：`newcap += (newcap + 3*threshold) / 4`
    - 扩容系数不会从`2`突变为`1.25`，当`oldcap`远大于`256`时，扩容系数取到`1.25`

**第二步：内存**

1. 之后根据元素的类型（cap*类型大小），对需要的内存空间进行计算，，之后对计算得到的内存进行向上取
2. 

<br>

#### 3. golang中数组和slice作为参数的区别？slice作为参数传递有什么问题？

https://blog.csdn.net/weixin_44387482/article/details/119763558

1. 当使用数组作为参数和返回值的时候，传进去的是值，在函数内部对数组进行修改并不会影响原数据
2. 当切片作为参数的时候穿进去的是值，也就是值传递，但是当我在函数里面修改切片的时候，我们发现源数据也会被修改，这是因为我们在切片的底层维护这一个匿名的数组，当我们把切片当成参数的时候，会重现创建一个切片，但是创建的这个切片和我们原来的数据是共享数据源的，所以在函数内被修改，源数据也会被修改
3. 数组还是切片，在函数中传递的时候如果没有指定为指针传递的话，都是值传递，但是切片在传递的过程中，有着共享底层数组的风险，所以如果在函数内部进行了更改的时候，会修改到源数据，所以我们需要根据不同的需求来处理，如果我们不希望源数据被修改话的我们可以使用copy函数复制切片后再传入，如果希望源数据被修改的话我们应该使用指针传递的方式

#### 4. 从数组中取一个相同大小的slice有成本吗？

没有，因为容量足够的情况下，截取切片只是对底层数组的引用，而不需要额外分配内存空间





<br>

## 接口与反射

### 1. 值接收者和指针接收者

**方法**：给用户自定义类型添加新的行为，区别于函数，它有一个接收者

**实现层面**：

- 实现了接收者是`值`的方法，隐含地实现了接收者是`指针`的方法（都是操作的副本）
- 实现了接收者是`指针`的方法，反之不会自动生成接收者是`值`的方法（如果自动生成，就起不到修改接收者的效果）

**调用层面**：（前提是两种接收者类型的方法都实现了）

- `值类型`可以调用`值接收者`的方法，也可以调用`指针接收者`的方法
- `指针类型`可以调用`指针接收者`的方法，也可以调用`值接收者`的方法

**调用时隐含的语法糖**

| 调用者   | 值接收者（传递副本）       | 指针接收者（目的是修改原始值）                               |
| -------- | :------------------------- | :----------------------------------------------------------- |
| 值类型   | 方法会使用调用者的一个副本 | 使用值的引用来调用方法（会修改原始值，语法糖自动实现）       |
| 指针类型 | 指针被解引用为值（值传递） | 实际上也是“传值”，方法里的操作会影响到调用者，类似于指针传参，拷贝了一份指针 |

- 类型`T`只有接收者是`T`的方法，而类型`*T`有类型`*T`和`T`的方法（需要手动实现`T`接收者方法）

**使用时机**：考虑以下条件，不满足则推荐使用值接收者

1. 需要方法修改接收者的值
2. 避免拷贝大对象（结构体等）

<br>


### 2. 使用接口实现多态

**多态**：是一种运行期的行为，它有以下几个特点：

1. 一种类型具有多种类型的能力
2. 允许不同的对象对同一消息做出灵活的反应
3. 以一种通用的方式对待多个使用的对象
4. 非动态语言必须通过继承和接口的方式来实现

**Go 多态**：不同结构体的方法实现了接口拥有的函数，实际执行时，看的是最终传入的实体类型是什么，调用的是实体类型实现的函数，即当函数传入不同的实体类型时，调用的实际上是不同的函数实现，从而实现多态

<br>


### 3. Go 与“鸭子类型”

> 鸭子类型：如果某个东西长得像鸭子，像鸭子一样会游泳，像鸭子一样嘎嘎叫，那它就可以被看做是一只鸭子

- 鸭子类型是动态编程语言的一种**对象推断策略**，更关注对象被如何**使用**，而不是对象的类型本身
- 在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口决定，而是由它"**当前**方法和属性的集合"决定
- Go 作为一种静态语言，通过接口实现了鸭子类型，实际上是 Go 的编译器在其中作了隐匿的转换工作

<br>


### 4. iface 和 eface

![image-20240321130828863](../static/image-20240321130828863.png)

#### iface

- `iface`包括接口的类型`inrerfacetype`和实体类型`_type`

```go
type iface struct {
    tab  *itab	// 接口表指针：指向itab实体（动态类型）
    data unsafe.Pointer	// 数据指针：指向接口具体值（在堆内的存储）（动态值）
}

type itab struct {
    inter  *interfacetype // 见下
    _type  *_type	// 描述实体类型（大小、内存对齐方式等）
    link   *itab
    hash   uint32 
    bad    bool   
    inhash bool   
    unused [2]byte
    fun    [1]uintptr	// 具体数据类型的方法入口地址
    					// 是按字典序排列的第一个方法的函数指针
}

// interfacetype 包装了_type，描述接口的类型
type interfacetype struct {
    typ     _type	// 见下
    pkgpath name	// 定义该接口的包名
    mhdr    []imethod	// 接口声明的方法列表
}

// _type Golang 描述各种数据类型的结构体
// Go 各种类型都是在 _type 字段基础上，增加一些额外的字段来进行管理的
type _type struct {
    // 类型大小
	size       uintptr
    ptrdata    uintptr
    
    hash       uint32	// 类型的 hash 值
    tflag      tflag	// 类型的 flag，和反射相关
    
    // 内存对齐相关
    align      uint8
    fieldalign uint8
    
    // 类型的编号，有bool, slice, struct 等等等等
	kind       uint8
	alg        *typeAlg
    
	// gc 相关
	gcdata    *byte
	str       nameOff
	ptrToThis typeOff
}
```

#### eface

```go
type eface struct {
    _type *_type	// 只包括实体类型
    data  unsafe.Pointer
}
```

<br>


### 5. 接口转换的原来

#### 动态值和动态类型

- 接口的结构体都包括：动态类型`tab/_type`，动态值`data`

- 接口值的零值是指动态类型和动态值都为`nil`，当仅且当这两部分的值都为`nil`的情况下，这个接口值就才会被认为`接口值 == nil`

####  实例赋值给接口 / 接口转换

- 当类型的方法集完全包含接口的方法集时，认为该类型实现了该接口
    - 接口变量可以存储任何实现了接口定义的所有方法的变量
- 接口的方法是字典序排列的，匹配的复杂度是`O(m+n)`（m个类型方法，n个接口方法）
- 可以将实现了接口的类型实例赋值给接口，从而实现接口转换

```go
// scr/runtime/iface.go 接口转接口源码
func convI2I(inter *interfacetype, i iface) (r iface) {
    // i 是绑定了实体类型的接口
    // r 是转换为 i 相同类型的新iface
	tab := i.tab
	if tab == nil {
		return
	}
	if tab.inter == inter {
		r.tab = tab
		r.data = i.data
		return
	}
	r.tab = getitab(inter, tab._type, false)	// 见下
	r.data = i.data	// 动态数据直接取类型实体的
	return
}

/*
r.tab = getitab(inter, tab._type, false) 的逻辑
（后续研究一下源码）
	1. 根据 inter、_type 在 itabTable 中寻找 itab
	2. 没找到，则上锁后再找一次（找到后，解锁同时返回）
	3. 在哈希表中没有找到，则生成一个新的 itab
	4. 将生成的 itab 添加到全局 itabTable 中
	5. 解锁
	
	上锁的逻辑：当其他goroutine查找相同itab时，在进行第二次查找前被挂住，之后可以直接查到写入哈希表的itab
*/
```

**itabTable**

```go
type itabTableType struct {
	size    uintptr             
	count   uintptr             
	entries [itabInitSize]*itab	// itab 指针数组
}
```

#### 接口转换的一般逻辑

**类型转空接口**：

1. `_type`字段直接复制源类型的`_type`
2. 调用`mallocgc`获得一块新内存，把值复制进去，`data`再指向这块新内存

**类型转非空接口**：

1. 入参`tab`是编译器在编译阶段预先生成好的
2. 新接口`tab`字段直接指向入参`tab`指向的`itab`
3. 调用`mallocgc`获得一块新内存，把值复制进去，`data`再指向这块新内存

**接口转接口**：`itab`调用`getitab`函数获取（只用生成一次，之后直接从哈希表中获取）

<br>


### 6. 类型转换与类型断言

**类型转换**：

```go
var f float64
f = float64(10)
```

**类型断言**：

- 类型断言操作对象是**接口**，比较接口的动态类型是否是断言类型

```go
var i any = new(Person)	// Person 是一个结构体	

s1 := i.(Person)	// 断言失败会 panic

s, ok := i.(Person)	// 安全断言
if ok {
    // 断言成功
}

// switch 断言（后续代码练习）
```

**`fmt.Println`处理参数**：参数类型是`interface`

1. 内置类型：穷举真实类型
2. 自定义类型：如果类型实现了`String()`方法，则直接调用，否则通过反射遍历对象的成员

<br>


### 7. 让编译器检测类型是否实现了接口*

```go
// 两种方法
var interface_name = (*T)(nil)
var interface_name = T{}
```

<br>


### 8. 什么是反射

**反射**：指计算机程序在运行时可以访问、检测和修改它本身状态或行为的一种能力

**本质**：程序在运行期，探知对象的类型信息（Type）和内存结构（Value）

**使用时机**：

1. 不能明确接口调用哪个函数，需要根据传入的参数在运行时决定
2. 不能明确传入函数的参数类型，需要在运行时处理任意对象

不推荐使用的理由：阅读困难、编码过程无法发现反射代码的错误、影响性能

<br>


### 9. 反射的实现

> 接口会存储实体的类型信息，反射就是通过接口的类型信息实现的，建立在类型的基础上

#### 反射的基本函数

`reflect.Type`是反射包里定义的接口，提供类型相关的方法，获取类型相关各种信息

```go
// go 1.21.1
func TypeOf(i any) Type {
	eface := *(*emptyInterface)(unsafe.Pointer(&i))	// 转为reflect包里的eface
	return toType((*abi.Type)(noescape(unsafe.Pointer(eface.typ))))
}

// 和eface基本一致
type emptyInterface struct {
	typ  *abi.Type
	word unsafe.Pointer
}
```

`reflect. Value`是定义在反射包里的结构体 

```go
// go 1.21.1
type Value struct {
	typ_ *abi.Type
	ptr unsafe.Pointer
	flag
}

func ValueOf(i any) Value {
	if i == nil {
		return Value{}
	}
	return unpackEface(i)
}

// unpackEface 将空接口转换成 Value
// 将 typ、word和标志位组装成 Value
func unpackEface(i any) Value {
	e := (*emptyInterface)(unsafe.Pointer(&i))
	t := e.typ
	if t == nil {
		return Value{}
	}
	f := flag(t.Kind())	// 设置标志位，标识Value.ptr是否是指向数据的指针
	if t.IfaceIndir() {
		f |= flagIndir
	}
	return Value{t, e.word, f}
}
```

**总结**：

- `TypeOf()`返回一个接口，通过接口定义的方法可以获取关于类型的所有信息
- `ValueOf()`返回一个结构体，包含类型信息以及实际值

#### 反射三大定律

> Reflection goes from interface value to reflection object.
>
> Reflection goes from reflection object to interface value.
>
> To modify a reflection object, the value must be settable（可设置的，不能是副本）.

- 想要操作原变量，反射变量`Value`必须持有原变量的地址

<br>


### 10. 实现深拷贝

**浅拷贝**：只复制指向某个对象的指针，而不复制对象本身，新旧对象中指针类型的字段还是共享一块内存

**深拷贝**：创造一个内容完全相同的对象，与原对象不共享内存

<p>

#### 方法一：序列化 + 反序列化

- 将对象序列化为一个JSON字串，再反序列化到复制的对象上
- 开销较大，且无法复制私有字段

```go
func Copy(dst any, src any) error {
	if dst == nil || src == nil {
		return fmt.Errorf("nil dst or src")
	}
	bytes, err := json.Marshal(src)
	if err != nil {
		return fmt.Errorf("unable to serialize src: %s", err)
	}
	err = json.Unmarshal(bytes, dst)
	if err != nil {
		return fmt.Errorf("unable to deserialize into dst: %s", err)
	}
	return nil
}
```

#### 方法二：通过反射

```go
// 后续探究
```

<br>


### 11. 判断两个对象是否完全相等*

使用函数`reflect.DeepEqual()`实现

```go
func DeepEqual(x, y any) bool {
	if x == nil || y == nil {
		return x == y
	}
	v1 := ValueOf(x)
	v2 := ValueOf(y)
	if v1.Type() != v2.Type() {
		return false
	}
	return deepValueEqual(v1, v2, make(map[visit]bool))
}

// deepValueEqual() 的逻辑：通过switch语句递归调用到最基本的数据类型，进行判断
// 从而保证两个值在每个层级上都相等
```

| 类型                                  | 深度相等情形                                                 |
| ------------------------------------- | ------------------------------------------------------------ |
| Array                                 | 相同索引处的元素“深度”相等                                   |
| Struct                                | 相应字段，包含导出和不导出，“深度”相等                       |
| Func                                  | 只有两者都是 nil 时判等                                      |
| Interface                             | 两者存储的具体值“深度”相等                                   |
| Map                                   | 1. 都为 nil；2. 非空、长度相等，指向同一个 map 实体对象，或者相应的 key 指向的 value “深度”相等 |
| Pointer                               | 1. 使用 == 比较的结果相等；2. 指向的实体“深度”相等           |
| Slice                                 | 1. 都为 nil；2. 非空、长度相等，首元素指向同一个底层数组的相同元素，即 &x[0] == &y[0] 或者 相同索引处的元素“深度”相等 |
| numbers, bools, strings, and channels | 使用 == 比较的结果为真                                       |

<br>

<br>

## goroutine 和 channel

> Do not communicate by sharing memory; instead, share memory by comminicating
>
> 不要通过共享内存来通信，而要通过通信来实现共享内存

### 1. CSP 模型

**CSP**：Communicating Sequential Processes，通信顺序进程

- 是一种用于描述两个独立的并发实体，通过共享的通讯管道进行通信的并发模型

- 重视`input`和`output`，尤其重视并发编程
- Go 实现了CSP模型，使用`goroutine`作为并发实体，使用`channel`通讯实现实体之间的数据共享

<br>

### 2. goroutine 的使用

- goroutine 是 Go 程序中最基本的并发单元
- 每一个Go程序都至少包含一个`goroutine`，即`main goroutine`，Go 程序启动时会自动创建

```go
go f() // 创建新的goroutine执行函数f
```

- goroutine 有动态栈内存，且初始动态栈很小（`2KB`）

<br>

### 3. channel 的应用

```go
// 基本用法
var 变量名称 chan 元素类型	// 声明
make(chan 元素类型, [缓冲大小])	// 初始化

var ch1 chan int	// 未初始化，为零值nil
ch2 := make(chan int)
ch3 := make(chan bool, 1)

ch := make(chan int)
ch <- 10	// 发送10到通道
x := <-ch	// 接收1：赋值
<-ch		// 接收2：不处理
x1, ok := <-ch	// 接收3：ok != false 时，接收的的都是有效值
if ok {
    // 接收
}

close(ch)	// 关闭通道（可被垃圾回收，不是必要操作）

func f1(ch <-chan int){}	// 从通道接收数据
func f2(ch chan<- int){}	// 发送数据给通道
```

**常见应用场景**

1. 停止信号
2. 定时任务

```go
// 超时控制：100ms没收到值，返回flase
select {
	case <-time.After(100 * time.Millisecond)
	case <-s.stopc:
		return false
}

// 定时任务
func woker() {
    ticker := time.Tick(1 * time.Second)
    for {
        select {
            case <-ticker:
            	// 执行1s定时任务
        }
    }
}
```

3. 解耦生产方和消费方：生产方只需先`channel`发送任务，消费方只需不断从工作队列中取任务
4. 控制并发数：

```go
// 缓冲型 channel
// 遍历任务列表，执行任务 w() 前需要取得 token，执行完后归还
var token := make(chan int, 3)
func main() {
    //...
    for _, w := range work {
        go func() {
            token <- 1
            w()
            <-token
        }()
	}
    //...
}
```

5. 其他：消息过滤，信号广播，事件订阅与广播，请求、响应转发，任务分发，结果汇总，限流，同步与异步等

<br>

### 4. select 与通道

#### 非阻塞模式

```go
// select 非阻塞接收：当接收失败时，会快速检测并执行 default 语句
select {
    case data := <-ch:
    	//...
    default:
    	//...接收失败
}
// 使用其他三种接收写法，当通道为空时且未关闭，会阻塞当前线程直到接收到数据
data := <-ch
<-ch
data, ok := <-ch
```

```go
// select 非阻塞发送
select {
    case ch <- 42:
    	//...
    default:
    	//...发送失败
}
// 阻塞发送，如果发送失败则会阻塞当前线程，直到通道有空间可用
ch <- 42
```

#### 多路复用

```go
select {
    case <-ch1:
    	//...
	case data := <-ch2
    	//...
    case ch3 <- 10:
    	//...
    default:
    	//默认操作
}
```

- 处理多个 channel 的发送/接收操作
- 当多个 case 满足时，会随机选择一个执行
- 都不满足时，会阻塞 goroutine

<br>

### 5. channel 底层结构

#### 数据结构

![image-20240323014526664](../static/image-20240323014526664.png)

```go
type hchan struct {
	qcount   uint			// 通道内元素数量           
	dataqsiz uint   		// 循环数组长度（记）
	buf      unsafe.Pointer	// 循环数组指针 （记）
	elemsize uint16			// 元素大小
	closed   uint32			// channel 是否被关闭
	elemtype *_type 		// 元素类型（记）
	sendx    uint   		// 下一个要发送的元素的下标（记）
	recvx    uint   		// 下一个要接收的数组下标（记）
	recvq    waitq  		// 等待接受的 goroutine 队列（记）
	sendq    waitq  		// 等待发送的 goroutine 队列（记）
    
	lock mutex	// 互斥锁，不允许并发读写（记）
}
```

![image-20240323014849088](../static/image-20240323014849088.png)

```go
// waitq 是 sudog 的双向链表，sudog 是对 goroutine 的一个封装
// sudog (synchronization data structute for goroutine, goroutine 同步数据结构)
type waitq struct {
    first *sudog
    last *sudog
}
```

<br>

#### 从 channel 接收

根据是否带`ok`判断通道是否被关闭，分为`chanrecv1()`（不带）和`chanrecv2()`写法，但而这实际上调用了同一个函数

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {...}
// ep 指向填充的内存空间，为nil时说明忽略了接收者
// block 标识阻塞型/非阻塞型接收
// 返回值 received 即对应的是 ok
```

**执行大致流程**

1. channel 为`nil`：非阻塞模式直接返回，阻塞模式下 goroutine 会被永远挂起
2. 快速检测：针对非阻塞模式且 channel 为空的情况
    - channel 未关闭，直接返回
    - `ep != nil`，返回类型零值
3. channel 关闭且`buf`无元素：非缓冲型关闭/缓冲型关闭但`buf`无元素的情况
    - 返回零值，返回`received = false`
    - 如果是`select`的 case 之一，这里属于被选中，可用作通知信号
4. `sendq`不为空：有 goroutine 等待发送数据给 channel（先检测发送队列）
    - 非缓冲型：内存复制`sender goroutine -> receiver goroutine`
    - 缓冲型：执行发送流程，将发送者的元素放入循环数组尾部
    - 接收前先判断是否向 channel 发送数据！
5. `buf`内有元素，正常接收，同时清理掉循环数组中的值，返回`true, true`
    - 即使 channel 关闭的情况下，也能正常接收，直到`c.qcount = 0`
6. 处理阻塞模式：构造`sudog`保存各种值，添加到`recvq`中，挂起 goroutine
7. 唤醒后，接收完毕

- ps：从快速检测到执行结束，会上锁保证并发安全（之前不加锁，是因为满足一个失败场景，即可直接返回）

<br>

#### 向 channel 发送

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {...}
```

**执行大致流程**

1. channel 为`nil`：非阻塞模式直接返回，阻塞模式下 goroutine 会被永远挂起
2. 快速检测：针对非阻塞模式，快速检测以下失败场景
    - 非缓冲型：`recvq`为空
    - 缓冲型：`buf`已满
3. channel 关闭：`panic`！
4. `recvq`不为空：直接将发送的数据复制到接收 goroutine
5. 缓冲型且有空间：正常发送，从`ep`复制数据
6. 缓冲型但`buf`已满：
    - 非阻塞模式：发送失败，返回 false
    - 阻塞模式：构造`sudog`，添加到`sendq`中
7. 唤醒：
    - channel 关闭：`panic`！
    - 正常发送

- ps：同样的，从快速检测到执行结束，会上锁保证并发安全

<br>

#### 关闭通道

```go
func closechan(c *hchan) {...}
```

**执行大致流程**

> 在不清楚是否还有发送 goroutine 的情况下，不要贸然关闭通道！

1. channel 为`nil`：`panic`！
2. ---- 上锁 ----
3. 修改标志位`c.closed = 1`
4. 释放所有`recvq`中的`sudog`：接收者收到零值
5. 释放所有`sendq`中的`sudog`：如果有，该 goroutine 直接引发 panic！
6. ---- 解锁 ----
7. 唤醒所有`sudog`相应的 goroutine
    - 对于发送 goroutine，会直接引发 panic！

**优雅地关闭通道**（场景问题）

1. 只有一个 sender：从发送端关闭就好
2. N sender 1 receiver：非阻塞模式下，设置一个传递关闭信号的 channel，该通道只有一个 sender，当该通道关闭后，直接退出不再发送数据，而原本发送数据的通道由 GC 代劳关闭（在接收端关闭）
3. N sender M receiver：
    - 设置`channel toStop`: 中间人，`channel stopCh`: 传递关闭信号
    - 当中间人收到任何**第一个** receiver 或 sender 的关闭信号后，关闭`stopCh`通道，从而避免重复关闭通道而引发 panic

<br>

#### 其他补充

**值的复制**：接收和发送数据的本质，都是**值的复制**，而非地址操作

**channel 是线程安全的**：接收、发生、关闭都使用了锁保证并发安全，其设计目的也是在多线程之间传递数据，自然是线程安全的

**无缓冲区和有缓冲区**：

- 无缓冲区，向通道发送时需要有 goroutine 接收，从通道接收时需要有 goroutine 读取，否则会阻塞
- 有缓冲区：正常读写，从空缓冲区通道读数据，或向满缓冲区通道写数据也会发生阻塞

**通道引发资源泄露**：goroutine 操作 channel 后，处于发送/接收的阻塞态，而 channel 一直处于满/空状态，同时垃圾回收器不会回收此类资源，进而导致 goroutine 无法被唤醒执行

**错误使用总结**

1. 对关闭的通道再发送值 => `panic`
2. 关闭已经关闭的通道 => `panic`
3. 关闭`nil`通道 => `panic`
4. 读/写一个`nil `channel => 永远阻塞

<br>

## 逃逸分析













<br>

## Tips

1. 不能给内置类型和接口定义方法
2. Go 不允许隐式类型转换，`=`两边类型必须相同
3. Go 中所有类型都”实现了“空接口，空接口没有定义任何函数



## 问题记录

1. 常见的panic
2. 需要make初始化的类型



<br>

## 参考

《程序员面试宝典》

[golang-guide](https://github.com/mao888/golang-guide/blob/main/golang/go-Interview/GOALNG_INTERVIEW_COLLECTION.md#%E5%9B%9B%E6%8E%A5%E5%8F%A3)

