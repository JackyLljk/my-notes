## 1. 为什么需要 context 

> context 是 Go1.7 加入的标准库

在 Go http 包的Server中，每个请求都对应一个 goroutine 去处理

- 用来处理一个请求的 goroutine 通常需要访问一些与请求特定的数据

- 比如终端用户的身份认证信息、验证相关的token、请求的截止时间

Handler 请求处理函数通常会额外启动 goroutine 用来访问后端

当一个请求被取消或超时时，所有用来处理该请求的 goroutine 都应该迅速退出，让系统释放这些 goroutine 占用的资源



context：简化对于处理单个请求的多个 goroutine 之间与请求域的数据、取消信号、截止时间等相关操作