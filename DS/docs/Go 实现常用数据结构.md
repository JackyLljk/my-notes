## 切片模拟栈

```go
stack := []int{}	// 初始化

stack[len(stack) - 1]	// top()

stack = stack[:len(stack) - 1]	// pop()

stack = append(stack, elem)	//  push()
```



## 实现队列

#### 标准库容器 list 实现队列

> 适用于基本类型的队列元素

```go
import "container/list"

queue := list.New()	// 初始化

queue.PushBack(elem)	// 入队

queue.Front()	// 队首

pop := queue.Front()
queue.Remove(pop)	// 出队

size := queue.Len()	// 队长（长度为0，用于判空）
```

### 切片实现队列



## 技巧

1. 使用函数体外的类似全局变量的效果时，可以在函数体内定义函数类型，将外部函数当做整个作用域 [Leetcode: 树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/submissions/512370804/?envType=study-plan-v2&envId=top-100-liked)

```go
func diameterOfBinaryTree(root *TreeNode) (res int) {
    // 定义内部函数
    var maxLen func(*TreeNode) int
    maxLen = func(root *TreeNode) int {
        if root == nil {
            return 0
        }
        l := maxLen(root.Left)
        r := maxLen(root.Right)
        res = max(res, l + r)
        return max(l, r) + 1
    }
    maxLen(root)
    return
}
```



### 将`[]int`追加到`[][]int`中

```go
var tmp []int
var res [][]int
...
// 将 tmp 复制到创建的新切片（非同一个底层数组）中，再将新切片追加到 res 中
// 保证 res 中的每个切片是独立的
res = append(res, append([]int(nil), tmp...))
```
