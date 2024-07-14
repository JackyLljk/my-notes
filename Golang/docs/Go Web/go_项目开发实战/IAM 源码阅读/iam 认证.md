## 1. IAM 认证功能

### 1.1 功能设计

IAM 用到了 Basic 和 Bearer 认证

**Basic 认证 - 前端登录**

**Bearer 认证 - 调用后端 API 服务**

**iam-apiserver 客户端：**

`iam-apiserver`：提供 IAM 系统的管理流功能接口，其客户端可以是前端（控制台），也可以是 APP 端

`iamctl`：方便用户在 Linux 下调用

`API`：支持第三方代码调用 iam-apiserver 提供的 API 接口，支持 API 调用

`SDK`：提供用户在代码中调用 API 接口的效率

**这些客户端认证方式有所区别：**

1. 控制台、APP 端：需要登录系统，即 `用户名:密码` Basic 认证
2. iamtcl、API 调用、SDK：不用登录系统，采用更安全的 Bearer 认证
3. Basic 认证作为 iam-apiserver 已经集成的认证方式，仍然可以供 iamctl、API 调用、Go SDK 使用



**密钥是否存在服务端？**

？secretKey 是用在资源授权的 ？

iam-apiserver 采用 JWT Token 认证方式，需要密钥 `secretKey`，但可以不需要为每个用户都维护一个密钥

- 将 iam-apiserver 提供的 API 接口注册到 API 网关中，通过 API 网关中的 Token 认证功能，来实现对 iam-apiserver API 接口的认证
- 有很多 API 网关可供选择，例如腾讯云 API 网关、Tyk、Kong 等



### 1.2 IAM 认证设计

![img](../../../../../static/7eed8e2364d358a8483c671d972fd2b6.jpg)

#### 密钥密码

- Basic 认证的用户名、密码，Bearer 认证的密钥都保存在 MySQL 中**持久存储**
- 认证需要获取密码或密钥进行反加密，这就需要查询密码密钥，有两种方式
    - 查询数据库（数据库的查询操作延时高，会导致 API 接口延时较高，所以不太适合用在数据流组件中）
    - 缓存在内存中（直接从内存中查询，从而提升查询速度，提高接口性能）

- 存储在内存需要考虑内存和数据库的一致性，增加代码复杂度
    - 控制流组件：性能延时要求不高，查询数据库
    - 数据流组件：`iam-authz-server` 将密钥存放在内存

**控制流**

- iam-apiserver 创建的密钥对是提供给` iam-authz-server` 使用
- 需要调用 ` iam-authz-server` 提供的 RESTful API 接口 `/v1/authz`
    - ` iam-authz-server`  实现了 Bearer 认证过程（资源授权其实可以直接注册到 API 网关中）
    
    

#### Q1. 不同应用的一致性问题

为了将控制流和数据流区分开来，密钥的 CURD 操作也放在了 iam-apiserver 中，，但是 iam-authz-server 需要用到这些密钥信息

解决方案：gRPC + Redis 订阅

1. `iam-authz-server` 通过 `gRPC API` 请求 `iam-apiserver`，获取所有的密钥信息
2. 当 `iam-apiserver` 有密钥更新时，会 `Pub` 一条消息到 `Redis Channel` 中
3. 因为` iam-authz-server` 订阅了同一个 `Redis Channel`，`iam-authz-searver` 监听到 channel 有新消息时，会获取、解析消息，并更新它缓存的密钥信息
4. 这样，我们就能确保 `iam-authz-server` 内存中缓存的密钥和 `iam-apiserver` 中的密钥保持一致。



#### Q2. 内存是否足够？

8G 的内存大概能保存约 8 千万个密钥信息，完全够用。后期不够用的话，可以加大内存



#### Q3. Redis down 掉，或者出现网络抖动，如何同步密钥数据？

（参考认证功能）



### 1.3 基于「策略模式」的认证策略

JWT 的几个认证方式基于**策略模式**进行构建

**auto 策略**：该策略根据 HTTP 头进行选择认证方式

- `Authorization: Basic XX.YY.ZZ`：Basic 认证
- `Authorization: Bearer XX.YY.ZZ`：Bearer 认证

**basic 策略**：实现 Basic 认证

**jwt 策略**：实现 Bearer 认证，具体实现是 JWT

**cache 策略**：JWT 认证，但 Token 中密钥 ID 是从**内存**中获取

**策略类**

```go
// iam/internal/pkg/middleware/auth.go（存放在项目公用的位置，几个应用都可以实现）

// 策略类：定义策略需要实现的方法 AuthFunc() gin.HandlerFunc
type AuthStrategy interface {
	AuthFunc() gin.HandlerFunc
}

// 抽象出策略执行者，切换不同的策略实现
type AuthOperator struct {
	strategy AuthStrategy
}

// 切换策略
func (operator *AuthOperator) SetStrategy(strategy AuthStrategy) {
	operator.strategy = strategy
}

// 调用策略中的方法
func (operator *AuthOperator) AuthFunc() gin.HandlerFunc {
	return operator.strategy.AuthFunc()
}
```

**添加策略：jwt**（其他策略 basic、auto、cache 类似）

```go
// internal/pkg/middleware/auth/jwt.go

const AuthzAudience = "iam.authz.marmotedu.com"

// 策略实现：JWT 认证
type JWTStrategy struct {
	ginjwt.GinJWTMiddleware
}

// 确保 JWTStrategy 实现了策略集
var _ middleware.AuthStrategy = &JWTStrategy{}

func NewJWTStrategy(gjwt ginjwt.GinJWTMiddleware) JWTStrategy {
	return JWTStrategy{gjwt}
}

// 实现策略集的算法
func (j JWTStrategy) AuthFunc() gin.HandlerFunc {
	return j.MiddlewareFunc()
}
```

- 使用 JWTStrategy 创建 `newJWTAuth().(auth.JWTStrategy)`，即封装的 `ginjwt.GinJWTMiddleware`



### 1.4 基于「抽象工厂模式」封装认证实现

```go
// iam/internal/apiserver/auth.go
func newAutoAuth() middleware.AuthStrategy {
	return auth.NewAutoStrategy(newBasicAuth().(auth.BasicStrategy), newJWTAuth().(auth.JWTStrategy))
}

func newJWTAuth() middleware.AuthStrategy {
	ginjwt, _ := jwt.New(&jwt.GinJWTMiddleware{
		...
	})

	return auth.NewJWTStrategy(*ginjwt)
}

func newBasicAuth() middleware.AuthStrategy {
	return auth.NewBasicStrategy(func(username string, password string) bool {
		...
	})
}
```

- 创建认证策略时，返回的都是认证策略接口，隐藏了内部认证功能的实现





### 1.5 认证功能设计技巧

#### Skill1. 面向接口编程

​	在使用NewAutoStrategy函数创建 auto 认证策略时，传入了middleware.AuthStrategy接口类型的参数，这意味着 Basic 认证和 Bearer 认证都可以有不同的实现，这样后期可以根据需要扩展新的认证方式

- 这也是策略模式的实现思路

#### Skill2. 抽象工厂模式？

​	auth.go文件中，通过 newBasicAuth、newJWTAuth、newAutoAuth 创建认证策略时，返回的都是接口。通过返回接口，可以在不公开内部实现的情况下，让调用者使用你提供的各种认证功能

- 这里的理解是，不创建对象而是创建接口？

#### Skill3. 策略模式

<br>

## 2. iam-apiserver 认证实现

### 2.1 登录认证

- apiserver 创建认证策略，加载到需要认证的 API 路由上，从而实现 API 认证

```go
// internal/apiserver/router.go
func installController(g *gin.Engine) *gin.Engine {
	...
	
	// 初始化 jwt 认证
	jwtStrategy, _ := newJWTAuth().(auth.JWTStrategy)
    
	g.POST("/login", jwtStrategy.LoginHandler)
	g.POST("/logout", jwtStrategy.LogoutHandler)
	// Refresh time can be longer than token timeout
	g.POST("/refresh", jwtStrategy.RefreshHandler)
    
    ...
}
```

- 使用 JWTStrategy 创建 `newJWTAuth().(auth.JWTStrategy)`，即封装的 `ginjwt.GinJWTMiddleware`
- gin-jwt 认证中间件
    - `jwtStrategy.LoginHandler`：Basic 认证 -> JWT 认证
    - `jwtStrategy.LogoutHandler` ：清空 Cookie 中 Bearer 认证相关信息
    - ` jwtStrategy.RefreshHandler` ：先执行 Bearer 认证

Basic 认证通过用户名和密码来进行认证，通常用在登陆接口 `/login `中

用户登陆成功后，会返回 JWT Token，前端会保存该 JWT Token 在浏览器的 Cookie 或 LocalStorage 中，供后续请求使用

**Q. 为什么要刷新 Token？**

- 通过登陆接口签发的 Token 有过期时间，有了刷新接口，前端就可以根据需要，自行刷新 Token 的过期时间

- 过期时间可以通过 iam-apiserver 配置文件的 `jwt.timeout` 配置项来指定。登陆后签发 Token 时，使用的密钥（secretKey）由jwt.key 配置项来指定



### 2.2 路由组 `/v1` 认证

#### 初始化认证策略

```go
// internal/apiserver/router.go
auto := newAutoAuth()
```

- 基于抽象工厂模式，初始化认证策略

```go
// internal/apiserver/auth.go
func newAutoAuth() middleware.AuthStrategy {
	return auth.NewAutoStrategy(newBasicAuth().(auth.BasicStrategy), newJWTAuth().(auth.JWTStrategy))
}
```

- 这里分别初始化了 Basic 认证策略和 jwt 认证策略

```go
// internal/apiserver/auth.go
func newBasicAuth() middleware.AuthStrategy {
	return auth.NewBasicStrategy(func(username string, password string) bool {
		// fetch user from database
		user, err := store.Client().Users().Get(context.TODO(), username, metav1.GetOptions{})
		if err != nil {
			return false
		}

		// Compare the login password with the user password.
		if err := user.Compare(password); err != nil {
			return false
		}

		user.LoginedAt = time.Now()
		_ = store.Client().Users().Update(context.TODO(), user, metav1.UpdateOptions{})

		return true
	})
}
```

```go
// internal/apiserver/router.go
v1.Use(auto.AuthFunc())
```

- 路由组 `/v1` 使用了 auto 认证策略

```go
// internal/pkg/middleware/auth/auto.go
type AutoStrategy struct {
	basic middleware.AuthStrategy
	jwt   middleware.AuthStrategy
}

var _ middleware.AuthStrategy = &AutoStrategy{}

func NewAutoStrategy(basic, jwt middleware.AuthStrategy) AutoStrategy {
	return AutoStrategy{
		basic: basic,
		jwt:   jwt,
	}
}
```

- auto 策略的实现，重点是下面的 `auto.AuthFunc()`

```go
// internal/pkg/middleware/auth/auto.go
func (a AutoStrategy) AuthFunc() gin.HandlerFunc {
	return func(c *gin.Context) {
		operator := middleware.AuthOperator{}
        
        // 解析出认证方式
		authHeader := strings.SplitN(c.Request.Header.Get("Authorization"), " ", 2)

		if len(authHeader) != authHeaderCount {
			core.WriteResponse(
				c,
				errors.WithCode(code.ErrInvalidAuthHeader, "Authorization header format is wrong."),
				nil,
			)
			c.Abort()

			return
		}

        // 根据解析出的字符串，设置认证策略为 Basic 或 JWT
		switch authHeader[0] {
		case "Basic":
			operator.SetStrategy(a.basic)
		case "Bearer":
			operator.SetStrategy(a.jwt)
			// a.JWT.MiddlewareFunc()(c)
		default:
			core.WriteResponse(c, errors.WithCode(code.ErrSignatureInvalid, 
                                                  "unrecognized Authorization header."), nil)
			c.Abort()

			return
		}

		operator.AuthFunc()(c)

		c.Next()
	}
}
```



#### Basic 认证策略

如果选择了 Basic 认证

IAM Basic 认证支持以下两种请求方式（gin LoginHandler 中间件的 Basic 认证）

```bash
# "用户名:密码" 通过 base64 加码后，通过 HTTP Authorization Header 进行传递，因为密码非明文，建议使用这种方式

# 通过 base64 加码
$ echo -n 'admin:Admin@2021'|base64
YWRtaW46QWRtaW5AMjAyMQ==

$ curl -XPOST -H"Authorization: Basic YWRtaW46QWRtaW5AMjAyMQ==" http://127.0.0.1:8080/login 
# 返回了过期时间和 token
{"expire":"2024-06-06T21:16:16+08:00","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJpYW0uYXBpLm1hcm1vdGVkdS5jb20iLCJleHAiOjE3MTc2Nzk3NzYsImlkZW50aXR5IjoiYWRtaW4iLCJpc3MiOiJpYW0tYXBpc2VydmVyIiwib3JpZ19pYXQiOjE3MTc1OTMzNzYsInN1YiI6ImFkbWluIn0.H1QxJ8CmnENDzn6z3AjXQ8HDRn53mK3WRYdBRX3yj2I"}
```

```bash
# "用户名:密码" 在 HTTP Body 中传递，因为密码是明文，所以这里不建议实际开发中，使用这种方式。
$ curl -s -XPOST -H'Content-Type: application/json' -d'{"username":"admin","password":"Admin@2021"}' http://127.0.0.1:8080/login 
{"expire":"2024-06-06T21:16:47+08:00","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJpYW0uYXBpLm1hcm1vdGVkdS5jb20iLCJleHAiOjE3MTc2Nzk4MDcsImlkZW50aXR5IjoiYWRtaW4iLCJpc3MiOiJpYW0tYXBpc2VydmVyIiwib3JpZ19pYXQiOjE3MTc1OTM0MDcsInN1YiI6ImFkbWluIn0.kfuZpspnJPGoEfswiliWtL89hcMPKHLqX8e-Qj7JzLE"}
```

- 具体实现则是调用了 gin 框架的中间件



#### JWT 认证策略

如果选择了 jwt 认证策略

```go
// internal/pkg/middleware/auth/jwt.go
const AuthzAudience = "iam.authz.marmotedu.com"

type JWTStrategy struct {
	ginjwt.GinJWTMiddleware
}

var _ middleware.AuthStrategy = &JWTStrategy{}

func NewJWTStrategy(gjwt ginjwt.GinJWTMiddleware) JWTStrategy {
	return JWTStrategy{gjwt}
}

// 最终调用的是 gin-jwt 包 GinJWTMiddleware 结构体的 middlewarelmpl 方法
func (j JWTStrategy) AuthFunc() gin.HandlerFunc {
	return j.MiddlewareFunc()
}
```



## 3. iam-auth-server 认证实现

- `iam-authz-server` 通过加载 Gin 中间件的方式，在请求 `/v1/authz` 接口时进行访问认证，采用的的认证方式是 JWT

> cache 策略实现的 Bearer 认证（JWT）

```go
// internal/authzserver/router.go
func installController(g *gin.Engine) *gin.Engine {
    
	auth := newCacheAuth()
    
	...

    // 在 /v1 路由分组中加载 cache 认证中间件使用 cache 认证策略
	apiv1 := g.Group("/v1", auth.AuthFunc())
	{
		authzController := authorize.NewAuthzController(cacheIns)

		// Router for authorization
		apiv1.POST("/authz", authzController.Authorize)
	}

	return g
}
```

```go
// internal/authzserver/jwt.go
func newCacheAuth() middleware.AuthStrategy {
	return auth.NewCacheStrategy(getSecretFunc())
}

func getSecretFunc() func(string) (auth.Secret, error) {
	return func(kid string) (auth.Secret, error) {
		cli, err := cache.GetCacheInsOr(nil)
		if err != nil || cli == nil {
			return auth.Secret{}, errors.Wrap(err, "get cache instance failed")
		}

		secret, err := cli.GetSecret(kid)
		if err != nil {
			return auth.Secret{}, err
		}

		return auth.Secret{
			Username: secret.Username,
			ID:       secret.SecretId,
			Key:      secret.SecretKey,
			Expires:  secret.Expires,
		}, nil
	}
}
```

- 创建 cache 认证策略，通过 `getSecretFunc()` 返回密钥信息，包括：

```go
type Secret struct {
	Username string
	ID       string
	Key      string
	Expires  int64
}
```

- 下面是 cache 认证策略的 jwt 认证具体实现

```go
// internal/pkg/middleware/auth/cache.go
// cache 策略的 jwt 认证实现
// 这里的 jwt 使用的是 github.com/golang-jwt/jwt/v4
func (cache CacheStrategy) AuthFunc() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 1. 从 Authorization: Bearer XX.YY.ZZ 请求头中获取 JWT Token
		header := c.Request.Header.Get("Authorization")
		if len(header) == 0 {
			core.WriteResponse(c, errors.WithCode(code.ErrMissingHeader, 
                                                  "Authorization header cannot be empty."), nil)
			c.Abort()

			return
		}

		var rawJWT string
		// Parse the header to get the token part.
		fmt.Sscanf(header, "Bearer %s", &rawJWT)

		// Use own validation logic, see below
		var secret Secret

		claims := &jwt.MapClaims{}

		// Verify the token
		// 2. 验证 Token
		parsedT, err := jwt.ParseWithClaims(rawJWT, claims, func(token *jwt.Token) (interface{}, error) {
			// Validate the alg is HMAC signature
			if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
				return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
			}

			kid, ok := token.Header["kid"].(string)
			if !ok {
				return nil, ErrMissingKID
			}

			var err error
			secret, err = cache.get(kid)
			if err != nil {
				return nil, ErrMissingSecret
			}

			return []byte(secret.Key), nil
		})
		if err != nil || !parsedT.Valid {
			core.WriteResponse(c, errors.WithCode(code.ErrSignatureInvalid, err.Error()), nil)
			c.Abort()

			return
		}

		if KeyExpired(secret.Expires) {
			tm := time.Unix(secret.Expires, 0).Format("2006-01-02 15:04:05")
			core.WriteResponse(c, errors.WithCode(code.ErrExpired, "expired at: %s", tm), nil)
			c.Abort()

			return
		}

		c.Set(middleware.UsernameKey, secret.Username)
		c.Next()
	}
}
```

**step1.** 从 `Authorization: Bearer XX.YY.ZZ` 请求头中获取 JWT Token，格式为 `XX.YY.ZZ`

**step2.** 调用 `ParseWithClaims`，校验 Token，步骤如下：

- 调用 `ParseUnverified` 函数：
    1. 获取 XX，base64 解码后得到 JWT Token 的 Header `{“alg”: xxx,“kid”:xxx,“typ”:xxx}`
    2. 获取 YY，base64 解码后得到 JWT Token 的 Payload `{“aud”:xxx,“exp”:xxx,“iat”:xxx,“iss”:xxx,“nbf”:xxx}`
    3. 根据 alg 获取 Token 加密函数
    4. 返回 Token 类型的变量，包含 Method、Header、Claims、Valid 这些重要字段
- 调用 keyFunc 获取密钥 secretKey（每个用户对应一个 Token？）

```go
// internal/pkg/middleware/auth/cache.go
func(token *jwt.Token) (interface{}, error) {
    // 解析加密算法
    if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
    	return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
    }

	// 获取 kid（secretID）
    kid, ok := token.Header["kid"].(string)
    if !ok {
    	return nil, ErrMissingKID
    }

    var err error
    // 根据 secretID，从内存中获取密钥 secretKey
    secret, err = cache.get(kid)
    if err != nil {
    	return nil, ErrMissingSecret
    }

    return []byte(secret.Key), nil
}
```

- 从 Token 中获取 Signature 签名字符串，即 `ZZ`
- 获取到 secretKey 之后，token.Method.Verify 验证 Signature 签名字符串 ZZ，也即 Token 的第三段是否合法
    - token.Method.Verify 实际上是使用了相同的加密算法和相同的 secretKey 加密 XX.YY 字符串
    - 假设加密之后的字符串为 WW，接下来会用 WW 和 ZZ base64 解码后的字符串进行比较，如果相等则认证通过，如果不相等则认证失败）

**step3. **调用 KeyExpired，验证 secret 是否过期。secret 信息中包含过期时间，只需要拿该过期时间和当前时间对比就行

**step4.** 设置 HTTP Header `username: secret.Username`

**注意**：apiserver 使用的 jwt 策略中，使用了全局的 secretKey 生成 Token，而 cache 策略中，每个用户都维护一个 secretID 对应的 secretKey 作为一种资源，后者的 secretKey 具有过期时间这种属性（后续再理解看看）





## 问题

1. 注册中间件到路由，怎么使用？















