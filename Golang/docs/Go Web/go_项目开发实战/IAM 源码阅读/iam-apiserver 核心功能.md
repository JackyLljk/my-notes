## 1. 应用框架相关特性

### 1.1 优雅关停

#### 优雅停止服务示例

通常可以通过以下方式停止我们的服务：

1. 在 Linux 终端键入 Ctrl + C（发送 `SIGINT` 信号）
2. 发送 `SIGTERM` 信号：如 `kill` 或 `systemctl stop` 等

使用这两种方式停止服务，会产生下面的问题，会对业务造成影响：

- 有些请求正在处理，如果服务端直接退出，会造成客户端连接中断，请求失败
- 我们的程序可能需要做一些清理工作，比如等待进程内任务队列的任务执行完成，或者拒绝接受新的消息等



> 在 Go 开发中，通常通过拦截 `SIGINT` 和 `SIGTERM` 信号，来实现优雅关停

```go
// 简单优雅关停示例
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "time"

    "github.com/gin-gonic/gin"
)

func main() {
    router := gin.Default()
    router.GET("/", func(c *gin.Context) {
        time.Sleep(5 * time.Second)
        c.String(http.StatusOK, "Welcome Gin Server")
    })

    srv := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }

    go func() {
        // 将服务在 goroutine 中启动
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("listen: %s\n", err)
        }
    }()

    quit := make(chan os.Signal)
    signal.Notify(quit, os.Interrupt)	// 注册 channel，以接收 os.Interrupt 信号的通知
    // os.Interrupt 是在终端按下 Ctrl+C 时发送的终止信号
    
    <-quit // 阻塞等待接收 channel 数据
    
    log.Println("Shutdown Server ...")

    // 5s 缓冲时间处理已有请求
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second) 
    defer cancel()
    
    // 调用 net/http 包提供的优雅关闭函数：Shutdown
    if err := srv.Shutdown(ctx); err != nil { 
        log.Fatal("Server Shutdown:", err)
    }
    log.Println("Server exiting")
}
```

1. 将 HTTP 服务放在 goroutine 中运行，程序不阻塞，继续执行
2. 创建一个无缓冲的 `channel quit`，调用 `signal.Notify(quit, os.Interrupt)`；通过 `signal.Notify` 函数调用，可以将进程收到的 `os.Interrupt（SIGINT`）信号，发送给 `channel quit`
3. `quit` 阻塞当前 main goroutine（防止启动 HTTP 服务的 goroutine 退出），等待从通道中接收关停信息
    - 唯一目的是阻塞 goroutine，收到的信号直接丢弃即可
4. 打印退出消息，提示准备退出当前服务
5. 调用 `net/http` 包提供的 `Shutdown` 方法：在指定的时间内处理完现有请求，并返回
6. 执行结束，退出 main 函数



#### iam-apiserver 优雅关停实现















## 记录

1. Linux 相关：ctrl + c，kill，systemctl，SIGINT、SIGTERM













