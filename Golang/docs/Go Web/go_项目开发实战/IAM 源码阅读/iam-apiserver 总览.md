## 1. apiserver 服务概述

**iam-apiserver Web 服务**：通过名为 iam-apiserver 的进程，对外提供 RESTful API 接口，完成用户、密钥、策略的 CRUD



### 1.1 服务接口

**认证相关接口**

![image-20240613084826495](../../../../../static/image-20240613084826495.png)

**用户相关接口**

![image-20240613084702180](../../../../../static/image-20240613084702180.png)

**密钥相关接口**

![image-20240613084733865](../../../../../static/image-20240613084733865.png)

**策略相关接口**

![image-20240613084746312](../../../../../static/image-20240613084746312.png)



### 1.2 操作工具

可以通过不同的客户端访问 iam-apiserver：前端、API 调用、SDK、iamctl 等，这些客户端会执行 HTTP 请求，调用 iam-apiserver 提供的 RESTful API 接口

#### cURL 工具

> 标准的 Linux 发行版都安装了 cURL 工具

cURL 工具可以方便地完成 RESTful API 的调用场景：

- 如设置 Header、指定 HTTP 请求方法、指定 HTTP 消息体、指定权限认证信息等 

```bash
# cURL 常用参数
-X/--request [GET|POST|PUT|DELETE|…]  指定请求的 HTTP 方法
-H/--header                           指定请求的 HTTP Header
-d/--data                             指定请求的 HTTP 消息体（Body）
-v/--verbose                          输出详细的返回信息
-u/--user                             指定账号、密码
-b/--cookie                           读取 cookie
```



### 1.3 secret 资源操作步骤

> 其他资源详细使用方法参考 `scripts/install/test.sh` 脚本

1. 启动 iam-apiserver
2. 登录 iam-apiserver，获取 token
3. 创建一个名为 secret0 的 secret
4. 更新 secret0 的描述
5. 获取 secret 列表
6. 删除 secret0



#### 0x01. 启动 iam-apiserver

```bash
$ sudo systemctl daemon-reload
# 重新加载 systemd 配置，以便 systemd 识别新的或修改过的服务配置文件

$ sudo systemctl enable iam-apiserver
# 将 iam-apiserver 服务设置为开机自启

$ sudo systemctl restart iam-apiserver
# 重新启动 iam-apiserver 服务以应用配置更改

$ systemctl status iam-apiserver
# 查看 iam-apiserver 服务的状态以确保其正常运行
```

看到 `avtive (runing)` 字段，表示 apiserver 运行成功

![image-20240613164403658](../../../../../static/image-20240613164403658.png)

登录 iam-apiserver，获取 token

```bash
$ curl -s -XPOST -H"Authorization: Basic `echo -n 'admin:Admin@2021'|base64`" http://127.0.0.1:8080/login | jq -r .token

# 得到 token
<token>
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJpYW0uYXBpLm1hcm1vdGVkdS5jb20iLCJleHAiOjE3MTc0OTk0NjUsImlkZW50aXR5IjoiYWRtaW4iLCJpc3MiOiJpYW0tYXBpc2VydmVyIiwib3JpZ19pYXQiOjE3MTc0MTMwNjUsInN1YiI6ImFkbWluIn0.goVVt5pl78pVk20jXioo-VkBIGA-uEsJO1V-JgmiiAE
```

设置 token 为环境变量（每次登录更新 token）

```bash
# 替换成生成的 token
TOKEN=<token>
```



#### 0x02. 创建一个名为 `secret0` 的 secret

```bash
$ curl -v -XPOST -H "Content-Type: application/json" -H"Authorization: Bearer ${TOKEN}" -d'{"metadata":{"name":"secret0"},"expires":0,"description":"admin secret"}' http://iam.api.marmotedu.com:8080/v1/secrets
```

```bash
$ curl -v -XPOST -H "Content-Type: application/json" 
				 -H"Authorization: Bearer ${TOKEN}" 
				 -d'{
				 	"metadata":{"name":"secret0"},
				 	"expires":0,
				 	"description":"admin secret"
				 }' http://iam.api.marmotedu.com:8080/v1/secrets
```

![image-20240613170230663](../../../../../static/image-20240613170230663.png)

- 请求返回头中返回了 `X-Request-Id` Header，唯一标识这次请求
    - 请求失败时可以提供给运维或开发定位失败的请求
    - 在微服务场景中，可以将 `X-Request-Id` 透传给其他服务，实现请求调用链



#### 0x03. 获取 `secret0` 的详细信息

```bash
$ curl -XGET -H"Authorization: Bearer ${TOKEN}" http://iam.api.marmotedu.com:8080/v1/secrets/secret0
```

```bash
{
	"metadata":{
		"id":23,
		"instanceID":"secret-yj8m30",
		"name":"secret0",
		"createdAt":"2024-06-03T19:11:36+08:00",
		"updatedAt":"2024-06-03T19:11:36+08:00"
	},
    "username":"admin",
    "secretID":"oiNK5OmXqmFBuLcgi6V5UbaZHDoJ5P6MB9DE",
    "secretKey":"afT90X6MJd3Fl7lktoR9vEURLTicXwHa",
    "expires":0,
    "description":"admin secret"
}
```



#### 0x04. 更新 `secret0` 的描述

```bash
$ curl -XPUT -H"Authorization: Bearer ${TOKEN}" -d'{"metadata":{"name":"secret"},"expires":0,"description":"admin secret(modify)"}' http://iam.api.marmotedu.com:8080/v1/secrets/secret0
```

```bash
{
	"metadata":{"id":23,"instanceID":"secret-yj8m30","name":"secret0","createdAt":"2024-06-03T19:11:36+08:00","updatedAt":"2024-06-03T19:16:41.731+08:00"},
		"username":"admin",
		"secretID":"oiNK5OmXqmFBuLcgi6V5UbaZHDoJ5P6MB9DE",
		"secretKey":"afT90X6MJd3Fl7lktoR9vEURLTicXwHa",
		"expires":0,
		"description":"admin secret(modify)"
}
```



#### 0x05. 获取 `secret0` 列表

```bash
$ curl -XGET -H"Authorization: Bearer ${TOKEN}" http://iam.api.marmotedu.com:8080/v1/secrets
```

```bash
{
	"totalCount":2,
	"items":[
		{"metadata":{"id":23,"instanceID":"secret-yj8m30","name":"secret0","createdAt":"2024-06-03T19:11:36+08:00","updatedAt":"2024-06-03T19:16:41+08:00"},"username":"admin","secretID":"oiNK5OmXqmFBuLcgi6V5UbaZHDoJ5P6MB9DE","secretKey":"afT90X6MJd3Fl7lktoR9vEURLTicXwHa","expires":0,"description":"admin secret(modify)"},
		{"metadata":{"id":22,"instanceID":"secret-53e537","name":"authztest","createdAt":"2024-06-02T04:51:20+08:00","updatedAt":"2024-06-02T04:51:20+08:00"},"username":"admin","secretID":"PDk3mCNbq3RrYICuBpJ5k5OHzLcTZ65DwLaX","secretKey":"uYQmjWFMcBzT1E4FDhGzh4qmMWBGZ9CF","expires":0,"description":"admin secret"}]}
```



#### 0x06. 删除 `secret0` 

```bash
$ curl -XDELETE -H"Authorization: Bearer ${TOKEN}" http://iam.api.marmotedu.com:8080/v1/secrets/secret0
```

```bash
null
```



### 1.4 IAM 应用测试

```bash
$ ./scripts/install/test.sh iam::test::test # 测试整个IAM应用是否正常运行
$ ./scripts/install/test.sh iam::test::login # 测试登陆接口是否可以正常访问
$ ./scripts/install/test.sh iam::test::user # 测试用户接口是否可以正常访问
$ ./scripts/install/test.sh iam::test::secret # 测试密钥接口是否可以正常访问
$ ./scripts/install/test.sh iam::test::policy # 测试策略接口是否可以正常访问
$ ./scripts/install/test.sh iam::test::apiserver # 测试iam-apiserver服务是否正常运行
$ ./scripts/install/test.sh iam::test::authz # 测试authz接口是否可以正常访问
$ ./scripts/install/test.sh iam::test::authzserver # 测试iam-authz-server服务是否正常运行
$ ./scripts/install/test.sh iam::test::pump # 测试iam-pump是否正常运行
$ ./scripts/install/test.sh iam::test::iamctl # 测试iamctl工具是否可以正常使用
$ ./scripts/install/test.sh iam::test::man # 测试man文件是否正确安装
```

- 每次发布完 iam-apiserver 后，可以执行以下命令完成冒烟测试：

```bash
$ export IAM_APISERVER_HOST=127.0.0.1 # iam-apiserver部署服务器的IP地址
$ export IAM_APISERVER_INSECURE_BIND_PORT=8080 # iam-apiserver HTTP服务的监听端口
$ ./scripts/install/test.sh iam::test::apiserver
```





## 2. apiserver 目录结构

```bash
iam
├── cmd	
│   ├── iam-apiserver
│   │   └── apiserver.go	# apiserver 组件的 main 函数
```

```bash
# iam/internal/apiserver 存放真实应用代码，对项目外私有
.
├── config
│   ├── config.go
│   └── doc.go
├── controller
│   └── v1
│       ├── cache
│       ├── policy
│       ├── secret
│       └── user
├── options
├── service
│   └── v1
├── store
│   ├── etcd
│   ├── fake
│   ├── mysql
│   ├── mock_store.go
│   ├── policy_audit.go
│   ├── doc.go
│   ├── policy.go
│   ├── secret.go
│   ├── store.go
│   └── user.go
├── testing
├── app.go
├── grpc.go
├── router.go
├── run.go
├── server.go
└── auth.go
```







### 2.1 从配置入手

iam-apiserver 有三种配置：

![image-20240614135809348](../../../../../static/image-20240614135809348.png)

- Option 配置：构建命令行参数，它的值来自于命令行选项或者配置文件（也可能是二者 Merge 后的配置）。Options 可以用来构建应用框架，Options 配置也是应用配置的输入
- 应用配置：iam-apiserver 组件中需要的一切配置。有很多地方需要配置，例如，启动 HTTP/GRPC 需要配置监听地址和端口，初始化数据库需要配置数据库地址、用户名、密码等
- HTTP/GRPC 服务配置：启动 HTTP 服务或者 GRPC 服务需要的配置



#### Option 配置

> 使用了选项模式 Options Pattern 构建可选的命令行参数

```bash
# Option 配置目录结构

# main 函数入口
iam
├── cmd	
│   ├── iam-apiserver
│   │   └── apiserver.go	# apiserver 组件的 main 函数
							# 调用 internal/apiserver/app.go 创建 cli

# internal/apiserver/app.go (应用配置代码)
internal/apiserver
├── options
│   ├── options.go	# 包含初始化 apiserver 的 flags 选项
│   └── validation.go
├── pkg
│   ├── code
│   ├── logger
│   ├── middleware
│   ├── options
│   │   ├── authentication.go
│   │   ├── doc.go
│   │   ├── etcd_options.go
│   │   ├── feature.go
│   │   ├── grpc.go
│   │   ├── insecure_serving.go
│   │   ├── jwt.go
│   │   ├── mysql_options.go
│   │   ├── redis_options.go
│   │   ├── secure_serving.go
│   │   └── server_run_options.go	# 
│   ├── README.md
│   ├── server
│   ├── util
│   └── validation
│       ├── doc.go
│       └── validation.go
└── app.go

```





### 3.2 请求处理流程设计

![image-20240619155244818](../../../../../static/image-20240619155244818.png)





## 记录

1. 仓库层用 etcd 替代 mysql













