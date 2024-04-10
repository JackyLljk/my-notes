## 1. Go 的动态类型

> version: Go SDK 1.22.1

```go
type Type struct {
	Size_       uintptr	// 类型大小
	PtrBytes    uintptr // 类型中能包含指针的前缀字节数
	Hash        uint32  // 类型的哈希值（可以在哈希表中快速找到类型）
	TFlag       TFlag   // 附加的类型信息标志
	Align_      uint8   // 对齐方式
	FieldAlign_ uint8   // 类型在结构体字段中的对齐方式
	Kind_       uint8   // enumeration for C
	
	Equal func(unsafe.Pointer, unsafe.Pointer) bool	// 用于比较该类型对象是否相等的函数

	GCData    *byte		// GC 程序或指针掩码位图
	Str       NameOff 	// 类型的字符串形式
	PtrToThis TypeOff 	// 指向该类型的指针，可能为 0
}
```















