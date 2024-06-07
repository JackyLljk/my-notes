## 1. 数组和切片的区别 

### 1.1 array vs slice

**相同点**：存储的元素类型，下标访问、len、cap

1. 只能存储一组相同类型的数据结构

2. 都是通过下标来访问，并且有容量长度，长度通过 len 获取，容量通过 cap 获取

**区别**：定长/扩容，类型不同、底层数组、扩容与数组

1. 数组是定长，访问和复制不能超过数组定义的长度，否则就会下标越界，切片长度和容量可以自动扩容

2. 数组是值类型，切片是引用类型，每个切片都引用了一个底层数组，切片本身不能存储任何数据，都是这底层数组存储数据，所以修改切片的时候修改的是底层数组中的数据。切片一旦扩容，指向一个新的底层数组，内存地址也就随之改变

<br>

### 1.2 slice 基本用法

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
b := make([]int, 3, 5)
```

**截取切片**

```go
// 下标low开始，截取到high-1的位置，最大容量是max（max小于底层数组容量）
slice = arr[low:high:max]	
```

**补充**

1. 数组未赋值元素默认初始值为 0
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


## 2. slice 的扩容策略

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
            // 3. 如果旧容量大于阈值，开始循环增加新容量，直到大于新长度
            // 扩增公式：newcap += (newcap + 3*threshold) / 4
			for 0 < newcap && newcap < newLen {
				newcap += (newcap + 3*threshold) / 4
			}
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

**第二步：内存分配**

- 根据元素的类型（cap*类型大小），对需要的内存空间进行计算（内存分配策略）

**补充**

- 可以向 nil 切片添加元素，append 会调用 mallocgc 申请一块内存

<br>

## 3. 切片传参问题

1. Go 语言的函数传递，只有值传递，没有引用传递（指针也是传的副本）
2. 函数参数无论是切片还是其指针，改变底层数组，都会反映在切片实参上
    - 传值虽然是结构体的副本，但具体到底层数组，传的其实是指针

<br>
