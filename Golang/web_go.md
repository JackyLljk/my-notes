---
title: Go Web开发
date: 2024-01-10 12:00:00
tags: [编程语言, Go]
categories: [编程语言]
---

> Golang 启动!!!!!

**参考**

[Go语言学习之路/Go语言教程](https://www.liwenzhou.com/posts/Go/golang-menu/)：Web开发部分

<!--more-->

## Go Web 

### vscode 新建项目

1. 新建文件夹，作为根目录
2. 打开终端，`go mod init 文件夹名`，生成一个`go.mod`存放项目依赖的文件
3. vscode 打开项目，根目录中创建`main.go`
4. 执行

```shell
go mod vendor	# 复制依赖到vendor目录下
go mod tidy		# 添加或删除 modules（不要随意使用）
```



### About Web

> Web 就是基于HTTP协议进行交互的网络应用，是通过使用浏览器/APP访问各种资源

![image-20240220181329612](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-20-101334.png)（一个请求对应一个响应)

   

**后端关注点**：访问url/地址时，应该返回什么信息，通过函数控制访问内容

- 后端可以返回JSON格式数据，使网站和App都能使用

**Web 开发**：浏览器发送请求，服务器接收后返回一个响应，一次请求对应一次响应，响应的内容按照浏览器支持一些特殊规则返回



#### 快速搭建一个简单的服务器

[net/http: 服务端部分](https://www.liwenzhou.com/posts/Go/http/)



### [Go 标准库之http/template](https://www.liwenzhou.com/posts/Go/template/)

#### 使用模板引擎

1. 定义模板文件
2. 解析模板文件
3. 模板渲染

#### 模板语法

> Go 模板中，用`pipeline`指产生数据的操作，如`{{.}}`

1. `{{.}}`：也可以访问复杂对象的字段（struct、map等）
2. 注释：`{{/* 注释内容 */}}`
3. **变量**：`$obj := {{.}}`，在模板文件中使用
4. 移除空格：`{{- .Name -}}`
5. **if 条件判断**
6. `range`：遍历常用
7. `with`：【用法】造一个局部作用域，省略访问字段的写法
8. **预定义函数**
9. **自定义函数**：在后端函数中定义函数，并告知模板`t.Funcs(template.FuncMap{})`
10. **嵌套**`template`：在模板中嵌套其他`template`，需要解析每个·
11. `block`：定义根模板，使用块模板继承，并自定义新的内容
12. **修改默认的标识符**：防止和其他前端框架起冲突

- 比较`text/template`与`html/template`





## Gin 框架

### [Gin 框架介绍](https://www.liwenzhou.com/posts/Go/gin/)

> Gin 是基于`httprouter`开发的用 Go 编写的 Web 框架（[中文文档](https://gin-gonic.com/zh-cn/docs/)）

下载安装

```shell
go get -u github.com/gin-gonic/gin
```

#### RESTful 架构（风格）

> 网站即软件，要使用软件开发的模式开发网站；REST（"资源"表现层状态转化）

[理解RESTful架构](https://www.ruanyifeng.com/blog/2011/09/restful.html)

​	URI（统一资源标识符）表示一种资源，客户端和服务端之间传递这种资源的某中**表示层**，客户端通过四个**HTTP动词**，对服务端资源进行操作，实现”**表现层状态转化**“

- GET 获取资源
- POST 新建资源
- PUT 更新资源
- DELETE 删除资源

可以使用Postman作为客户端的测试工具



#### Gin渲染*

> 目前使用 gin 模板渲染其实较少

1. HTML 渲染：`LoadHTMLGlob()`，`LoadHTMLFilhues()`
2. 自定义模板函数`router.SetFuncMap()`
3. 静态文件处理：在解析前添加`gin.Static`解析静态文件（html, css, js, 图片等）
4. 使用模板继承
5. JSON 渲染：返回`JSON`格式数据，使用map/gin.H/结构体
    - 使用结构体渲染时，内部字段要大写（反射）（可以使用tag定制）
6. XML 渲染、YAML 渲染
7. protobuf 渲染



#### 获取参数

> 一个请求对应一个响应

1. **获取`querystring`参数**：使用`&`连接多个query参数

![image-20240227130923681](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-28-124935.png)

2. **获取`form`参数**：使用`r.POST()`处理浏览器页面表单的请求动作
3. **获取`path/url`参数**
4. **参数绑定**：`ShouldBind()`获取参数（常用一点）



#### 文件上传

1. **单个文件上传**：`FormFile()`， `SaveUploadFile()`，会有内存限制
1. 多个文件上传：`MultpartForm()`，循环读取



#### 重定向

1. **HTTP 重定向**：`Redirect()`
2. **路由重定向**：`HandleContext()`



#### Gin 路由

1. **普通路由**：通常用法如下
    - `r.GET()`：浏览器请求获取服务器的信息
    - ` r.POST()` ：把数据发送给服务器（创建）
    - `r.PUT()`：更新操作
    - `r.DELETE()`：删除操作
    - `r.NoRoute()`：专门设置没有匹配到路由的404页面
    - `r.Any()`：匹配所有请求方法（不太安全）
2. **路由组**：`r.Group()`设置共有前缀
3. 路由原理：[httprouter源码](https://github.com/julienschmidt/httprouter)



#### Gin 中间件

> Gin 允许在处理请求时加入用着自己的钩子（Hook）函数，即中间件，在响应请求前处理一些公共业务逻辑

![image-20240228140047991](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-28-124940.png)

1. **定义中间件**：`gin.HandlerFunc 类型`
2. **理解`c.Next()`的处理流程**：调用后续的处理函数

![image-20240228143534765](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-28-124944.png)

3. `c.About()`：阻止执行后续处理函数

![image-20240228143923783](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-28-124947.png)

4. **注册中间件**
5. 默认中间件
6. gin中间件中使用goroutine：必须拷贝`c.Copy()`后使用副本



## GORM 框架

> GORM 是Go编写的ORM框架，支持主流数据库

 ![image-20240228145824213](https://jkey-imgs.oss-cn-nanjing.aliyuncs.com/2024-02-28-124950.png)



`数据表 <--> 结构体`

`数据行 <--> 结构体实例`

`字段 <--> 结构体字段`

优点：提高开发效率

缺点：牺牲执行性能、灵活性，弱化SQL能力（对性能要求高、大型项目不建议使用）

### [GORM 入门](https://www.liwenzhou.com/posts/Go/gorm/#c-0-0-0)

[官方文档](https://gorm.io/zh_CN/)	[github gorm](https://github.com/go-gorm/gorm?tab=readme-ov-file)

#### GORM Model

内置`gorm.Model`结构体，可以嵌入到自己的模型中

```go
// gorm.Model 定义
type Model struct {
  ID        uint `gorm:"primary_key"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt *time.Time
}
```

**结构体标记**：结构体标记，关联相关标记

#### GORM 约定

- 主键、表名、列名、时间戳

#### [GORM CRUD 要点记录](https://www.liwenzhou.com/posts/Go/gorm-crud/)

**C**：处理字段默认零值/空串的情况（有些字段设置为指针类型的原因）

`db.Debug().语句`可以查看执行的数据库语句





## 问题记录

1. 创建模板`New()`操作有什么用？
1. any 是什么类型？
1. 了解反射的原理！
1. tag中的 form/json
1. 在一个包内创建多个包/创建多个项目  



## 参考

[使用VS code快速搭建一个Golang项目](https://blog.csdn.net/apple_51931783/article/details/127805320)

[Go语言学习之路/Go语言教程](https://www.liwenzhou.com/posts/Go/golang-menu/)



















