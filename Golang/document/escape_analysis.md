## 1. 什么是逃逸分析

### 1.1 逃逸分析

**逃逸分析**：编译器决定了一个变量是分配在栈上、还是堆上的过程，这一操作在编译期完成

- 这个堆就是运行时维护的 Go 堆`mheap`

目的*

1. 让栈和堆对程序员透明，将程序员从内存管理的复杂机制中解放（面试不让解放...）
2. 分配在栈空间的对象，函数执行结束时自动收回；而分配在堆上的对象，函数结束后还需要等待垃圾回收，从而增加开销；需要认识到，堆内存分配导致垃圾回收的开销远远大于栈空间分配与释放的开销

<br>

### 1.2 Go 语言的逃逸分析

**结论**：Go 中一个函数内局部变量，**不管是不是`new`得到的**，都是由编译器进行逃逸分析后，决定它被分析到堆上还是栈上

**实现方式**：Go 程序变量会携带有一组校验数据，用来证明它的整个生命周期是否在运行时完全可知；如果变量在**编译期**通过了这些校验，它就可以在栈上分配；否则就说它逃逸了，必须在堆上分配

**我的理解**：逃逸，即编译期在栈上分配对象，运行时则需要分配到堆上

- 在编译阶段，编译器会尽可能地在栈上分配对象，以提高内存访问的效率

- 但是，如果对象的生命周期超出了它所在函数的作用域，或者被函数返回时，这些对象就会发生逃逸，需要在堆上分配内存以保证其在函数外部仍然可以被访问和使用

<br>

## 2. 常见逃逸场景

### 2.1 指针逃逸

**判断依据**：函数内创建了对象，在函数结束后返回该对象的指针，函数虽然退出，但由于指针的存在，对象的内存不能随着函数结束而回收，只能分配在堆上（生命周期大于函数）

<br>

### 2.2 空接口动态类型逃逸

**在 interface 类型上调用方法**：在`interface`类型上调用方法都是动态调度的，即方法的真正实现只能在运行时知道

**对`[]any`进行赋值**：也是在赋值后根据动态类型确定的，会发生逃逸

<br>

### 2.3 栈空间不足

**爆栈**：栈空间通常较小，当递归函数使用不当时，容易导致栈溢出

**大小限制**：操作系统对内核线程使用的栈空间是有大小限制的，64 位系统上通常是 8 MB

![image-20240327205234873](../../static/image-20240327205234873.png)

- 当栈空间超过限制时，会逃逸到堆上

### 2.4 闭包

- 闭包函数和对其环境的引用捆绑在一起，当函数变量存在时，环境内的变量就一直存在，即逃逸到堆上

<br>

## 3. 如何确定发生了逃逸

```shell
go build -gcflags '-m -l' main.go
# -gcflags 启用编译器支持的额外标志
# -m 输出编译器的优化细节（包括逃逸分析），可以用 -N 关闭编译器优化
# -l 禁用内联优化
```

<br>

**不准确总结**：当发生在编译期无法确定的内存分配，局部变量超过了函数的生命周期（不再属于函数栈），`interface`等使用编译期无法确定的动态类型等情况，就会发生逃逸

<br>

## 参考

[Golang中逃逸现象, 变量“何时栈?何时堆?”](https://www.yuque.com/aceld/golang/yyrlis)