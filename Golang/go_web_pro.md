> 建议主动查看源码进行学习，利用好Golang自动代码格式化的有点

## Gin 框架源码解析

[gin 框架源码解析](https://www.liwenzhou.com/posts/Go/gin-sourcecode/)

#### 安装 gin

**方法一**

```go
import "github.com/gin-gonic/gin"
```

```shell
$ go mod tidy		# 查询当前import的库是否与go.mod中一致，否则将添加/清除
```

**方法二**

```shell
$ go get -u github.com/gin-gonic/gin
```

### 路由详解

gin框架基于[httprouter](https://github.com/julienschmidt/httprouter)，路由的实现结构基于[Radix Tree](https://ivanzz1001.github.io/records/post/data-structure/2018/11/18/ds-radix-tree#2-radix-tree%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E4%B8%BE%E4%BE%8B)（基数树）

- 每种请求方法管理一颗单独的树
- 优先匹配树中最长（层级）的路径

对象池？pool？垃圾回收？



## Go 数据库

[go-sql-driver](https://github.com/go-sql-driver/mysql)

`db.Close()`类似的操作时，要确保`db`不为空，即在创建且处理错误之后再使用其函数，不然会引发`panic`

#### mysql 预处理

- 一次编译（命令部分），多次执行（数据部分）、
- 避免SQL注入问题（在数据交互地方，恶意输入SQL语句，访问数据库）

[sql注入问题](https://www.cnblogs.com/malongfeistudy/p/16745900.html)

#### 事务

ACID

学习后练习

#### Web项目CLD层

![image-20240305154112205](/Users/jk/Desktop/%E5%90%8C%E6%AD%A5/%E7%AC%94%E8%AE%B0/Golang/assets/image-20240305154112205.png)

### 在Go项目中使用日志

[Uber-go Zap 日志](https://www.liwenzhou.com/posts/Go/zap/)

[使用zap接收gin框架默认的日志并配置日志归档](https://www.liwenzhou.com/posts/Go/zap-in-gin/)

logger/sugarLogger



### 管理配置文件

[Go语言配置管理神器——Viper中文教程](https://www.liwenzhou.com/posts/Go/viper/)















## 记录

1. 不推荐`go get`方式安装第三方库，推荐先在`import`中引入后，命令行运行`go mod tidy`







## 问题

1. 不同的`err`怎么处理？什么时候需要`panic(err)`
2. `defer`用法和使用时机



 







