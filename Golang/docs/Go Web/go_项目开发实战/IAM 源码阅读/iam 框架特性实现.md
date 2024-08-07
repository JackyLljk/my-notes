## 1. 优雅关停

### 1.1 优雅停止服务示例

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



### 1.2 优雅关停实现 pump

**step1.**  创建 channel 用来接收 `os.Interrupt`（SIGINT）和 `syscall.SIGTERM`（SIGKILL）信号

```go
// iam/internal/pkg/server/signal.go

// onlyOneSignalHandler 传递信号用，通知 SetupSignalHandler 是否只使用一次（通道的用例！）
var onlyOneSignalHandler = make(chan struct{})

var shutdownHandler chan os.Signal

func SetupSignalHandler() <-chan struct{} {
	// 保证 iam-apiserver 组件的代码只调用一次 SetupSignalHandler()
	close(onlyOneSignalHandler) // panics when called twice

	// 缓冲区大小为 2
	shutdownHandler = make(chan os.Signal, 2)

	// 使用 struct{} 类型，该类型不占用任何内存空间，作为一种轻量级的信号传递机制（用作信号通知）
	// 其目的仅用来进行信号传递，不传递实际数据
	stop := make(chan struct{})

	// 注册通道，仅用来接收 os.Interrupt 和 syscall.SIGTERM 信号
	signal.Notify(shutdownHandler, shutdownSignals...)

	// 收到一次 SIGINT/ SIGTERM 信号，程序优雅关闭
	// 收到两次 SIGINT/ SIGTERM 信号，程序强制关闭
	go func() {
		<-shutdownHandler
		close(stop)
		<-shutdownHandler
		os.Exit(1) // second signal. Exit directly.
	}()

	// 返回仅用来发送信号的切片
	return stop
}
```

**step2.** 将 `stop` 通道传递给 HTTP(S)、gRPC 服务的函数，在函数中以 goroutine 的方式启动  HTTP(S)、gRPC 服务，执行 `<-stop` 阻塞 goroutine

**step3.** 当 iam-apiserver 进程收到 `SIGINT/SIGTERM` 信号后，关闭 `stop channel`，继续执行 `<-stop` 后的代码，在后面的代码中，我们可以执行一些清理逻辑，或者调用 `google.golang.org/grpc` 和 `net/http` 包提供的优雅关停函数 GracefulStop 和 Shutdown

```go
// 使用案例
func (s *grpcAPIServer) Run(stopCh <-chan struct{}) {
    listen, err := net.Listen("tcp", s.address)
    if err != nil {
        log.Fatalf("failed to listen: %s", err.Error())
    }

    log.Infof("Start grpc server at %s", s.address)

    go func() {
        if err := s.Serve(listen); err != nil {
            log.Fatalf("failed to start grpc server: %s", err.Error())
        }
    }()

    <-stopCh

    log.Infof("Grpc server on %s stopped", s.address)
    s.GracefulStop()
}
```

> 这种方法 apiserver 没有用到，而是 pump 用到了

```go
// iam/internal/pump/server.go
func (s preparedPumpServer) Run(stopCh <-chan struct{}) error {
	ticker := time.NewTicker(time.Duration(s.secInterval) * time.Second)
	defer ticker.Stop()

	log.Info("Now run loop to clean data from redis")
	for {
		select {
		case <-ticker.C:
			s.pump()
		// exit consumption cycle when receive SIGINT and SIGTERM signal
		case <-stopCh:
			log.Info("stop purge loop")

			return nil
		}
	}
}
```



### 1.3 优雅关停 apiserver

**注册 shutdown 实例**

```go
// iam/internal/apiserver/run.go
// 运行 apiserver
func Run(cfg *config.Config) error {
	server, err := createAPIServer(cfg)
	if err != nil {
		return err
	}

	return server.PrepareRun().Run()
}
```

```go
// iam/internal/apiserver/server.go
func createAPIServer(cfg *config.Config) (*apiServer, error) {
	// 应用关闭回调
	gs := shutdown.New()	// 初始化
	gs.AddShutdownManager(posixsignal.NewPosixSignalManager())	// 添加监听的信号
	
    ...
    
    // 注册服务
	server := &apiServer{
		gs:               gs,
		redisOptions:     cfg.RedisOptions,
		genericAPIServer: genericServer,
		gRPCAPIServer:    extraServer,
	}
}

func (s *apiServer) PrepareRun() preparedAPIServer {
	... 

	// 设置优雅关闭回调函数，执行清理工作
	s.gs.AddShutdownCallback(shutdown.ShutdownFunc(func(string) error {
		mysqlStore, _ := mysql.GetMySQLFactoryOr(nil)
		if mysqlStore != nil {	
			_ = mysqlStore.Close()	// 关闭 MySQL
		}

		s.gRPCAPIServer.Close()	// 关闭 gRPC
        s.genericAPIServer.Close()	// 关闭 HTTP(S) 服务

		return nil
	}))

	return preparedAPIServer{s}
}
```

[优雅关停讲解](https://time.geekbang.org/column/article/402206)

TODO：完全理解优雅关停





<br>

## 2. 健康检查

​	通过会根据进程是否存在判断 iam-apiserver 是否健康，例如执行 `ps -ef|grep iam-apiserver`，但在实际开发中会发现有些服务进程仍然存在，但是 HTTP 服务却不能接收和处理请求

1. 可以在启动 iam-apiserver 进程后，手动调用 iam-apiserver 健康检查接口进行检查
2. 启动服务后自动调用健康检查接口

```go
// iam/internal/pkg/server/genericapiserver.go

// http 服务健康检查接口
func (s *GenericAPIServer) ping(ctx context.Context) error {
	url := fmt.Sprintf("http://%s/healthz", s.InsecureServingInfo.Address)
	if strings.Contains(s.InsecureServingInfo.Address, "0.0.0.0") {
		url = fmt.Sprintf("http://127.0.0.1:%s/healthz", 
                          strings.Split(s.InsecureServingInfo.Address, ":")[1])
	}

	for {
		// Change NewRequest to NewRequestWithContext and pass context it
		req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
		if err != nil {
			return err
		}
		// Ping the server by sending a GET request to `/healthz`.

		resp, err := http.DefaultClient.Do(req)
		if err == nil && resp.StatusCode == http.StatusOK {
			log.Info("The router has been deployed successfully.")

			resp.Body.Close()

			return nil
		}

		// Sleep for a second to continue the next ping.
		log.Info("Waiting for the router, retry in 1 second.")
		time.Sleep(1 * time.Second)

		select {
		case <-ctx.Done():
			log.Fatal("can not ping http server within the specified time interval.")
		default:
		}
	}
}
```



<br>

## 3. 插件化加载中间件













## 记录

1. 异步编程模型的核心思想是，通过不阻塞主线程或主进程，使得程序可以在等待某些操作完成时继续执行其他任务（非阻塞，回调机制，并发处理）

















