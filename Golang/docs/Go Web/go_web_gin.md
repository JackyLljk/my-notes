## 1. RESTful API

**RESTful 风格**：网站即软件，要使用软件开发的模式开发网站；REST（「资源」表现层状态转化）

- 资源：网络上的一个实体，用 URI（统一资源标识符）表示

- 表现层：资源呈现的形式，如文本用 txt、html 等格式表现，在 HTTP 请求头的`Accept`和`Content-Type`字段

- 状态转化：HTTP 协议是无状态协议，每个请求独立，客户端需要通过某种手段让服务端发生「状态转化」
    - ”某种手段“即 `GET, POST, PUT, DELETE` 四种请求方法

在客户端与 Web 服务器之间进行交互的时候，HTTP 协调中的请求方法遵循以下 REST 风格，即是 RESTful API

`GET`：获取资源

`POST`：新建字体

`PUT`：更新资源

`DELETE`：删除资源

<br>

## 2. 使用 gin 框架

[gin](https://github.com/gin-gonic/gin)

```go
import "github.com/gin-gonic/gin"
```

```shell
$ go get -u github.com/gin-gonic/gin
```

### 2.1 基本使用流程

#### 创建路由

```go
r := gin.Default()	// 使用默认路由
```



<br>

## 3. gin 框架路由

### 3.1 路由前缀树

gin 框架路由使用公共前缀的树结构，属于 Tree 树或 Radix Tree

注册路由的过程就是构造前缀树的过程，具有公共前缀的节点共享一个公共父节点

**路由器为每种请求方法管理一颗单独的树**

`节点 - Handler 函数指针`构成一种类哈希表结构

```go
r := gin.Default()

r.GET("/", func1)
r.GET("/search/", func2)
r.GET("/support/", func3)
r.GET("/blog/", func4)
r.GET("/blog/:post/", func5)
r.GET("/about-us/", func6)
r.GET("/about-us/team/", func7)
r.GET("/contact/", func8)
```

得到一个`GET`方法的路由树

```markdown
Priority   Path             Handler
9          \                *<1>
3          ├s               nil
2          |├earch\         *<2>
1          |└upport\        *<3>
2          ├blog\           *<4>
1          |    └:post      nil
1          |         └\     *<5>
2          ├about-us\       *<6>
1          |        └team\  *<7>
1          └contact\        *<8>
```

`Handler 函数`：处理 HTTP 请求

```go
func MyHandler(w http.ResponseWriter, r *http.Request) {}	// http 包的形式

func MyHandler2(c *gin.Context) {}	// gin 框架的形式
```

<br>

### 3.2 路由树节点







## 4. gin context

Context 是 gin 框架最重要的部分，允许中间件之间传递变量、管理流程、验证请求的 JSON 并呈现 JSON 响应

```go
// package gin
type Context struct {
	writermem responseWriter
	Request   *http.Request
	Writer    ResponseWriter

	Params   Params
	handlers HandlersChain
	index    int8
	fullPath string

	engine       *Engine
	params       *Params
	skippedNodes *[]skippedNode

	// 用于 Keys 的互斥锁
	mu sync.RWMutex

	// 请求上下文内存储的键值对
	Keys map[string]any

	// 附加到使用此上下文的所有处理程序中间件的错误列表
	Errors errorMsgs

	Accepted []string

	// 缓存 c.Request.URL.Query() 的查询结果
	queryCache url.Values

	// 缓存 c.Request.PostForm 的表单数据
	formCache url.Values

	// SameSite 允许服务器定义 cookie 属性，使浏览器无法将此 cookie 与跨站点请求一起发送
	sameSite http.SameSite
}
```



















## 参考

[gin 框架介绍及使用](https://www.liwenzhou.com/posts/Go/gin/)

[gin 框架路由拆分与注册](https://www.liwenzhou.com/posts/Go/gin-routes-registry/)

[gin 框架源码解析](https://www.liwenzhou.com/posts/Go/gin-sourcecode/)