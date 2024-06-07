## 1. 什么是 defer

### 1.1 defer

Go 语言提供的一种用于注册延迟调用的机制，让函数或语句可以在当前函数执行完毕后（包括 return 正常结束和 panic 异常结束）执行，是一种语法糖操作

- 常用于成对操作：打开/关闭连接、加/释放锁、打开/关闭文件等

- defer 会有短暂延迟，但对可能引发 panic 的操作，有必要使用 defer 语句

<br>

### 1.2 defer 语句的执行顺序：

defer 语句定义时会把函数压栈，复制函数参数，当**外层函数**退出时，defer 函数会按照定义的顺序**逆序执行**

- defer 执行的函数为 nil，调用时产生 panic
- 后定义的 defer 函数可能依赖前定义的函数资源，所以是逆序执行
- 复制的函数参数如果是指针，则指向的值前后可能不一致
- return 之后的 defer 函数不能注册

<br>

## 2. defer 结构

### 2.1 拆解 defer 语句

**return 语句**：`return xxx`在编译后，实际上生成了三条指令（非原子性）

1. 返回值 = xxx
2. 调用 defer 函数（可能会操作返回值）
3. 空的 return（执行 RET 指令，从函数中返回）

**defer 语句在定义时会直接复制函数参数**：参数在定义的位置就确定好了

<br>

### 2.2 defer 处理 panic

panic 会停掉当前正在执行的程序（不只是线程），退出线程前，会有序地执行**当前线程** defer 列表中的语句

- 在 defer 里定义一个 recover 语句，可以防止程序直接挂掉
- `recover()`函数只有在 defer 的函数中直接调用才有效

<br>

### 2.3 defer 链的遍历执行

`deferproc`：在定义 defer 语句时，会调用`deferproc`函数 new 一个`_defer`结构体，挂在 goroutine 上

- defered 函数所需要的参数复制到和`_defer`相邻的位置，即所谓的「defer 快照」

`deferreturn`：在执行 RET 指令前调用 deferreturn，遍历`_defer`链表，执行链表上的所有 defered 函数

- 从链表末端开始，每执行一个 defer 函数后，会将`_defer`从链表中收回，直至链表为空
- defer 函数执行的顺序，即定义的逆序执行

<br>

## 3. 问题补充

### Q1: 从父 goroutine 恢复子 goroutine 的 panic

goroutine 被设计为一个独立的代码执行单元，拥有自己的函数执行栈，不与其他 goroutine 共享数据

- goroutine 没有返回值，也没有自身的 ID 编号等
- 与其他 goroutine 交互需要通过 channel 或加锁共享内存的方式

解决思路：创建一个用于 panic 通知的 channel，并 recover() 修复

<br>

## 参考

[《Go 程序员面试宝典》- 延迟语句](https://book.douban.com/subject/35871233/)

























