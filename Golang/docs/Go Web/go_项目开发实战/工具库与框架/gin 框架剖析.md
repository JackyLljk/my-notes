## 中间件

### gin-jwt

### GinJWTMiddleware

`middlewarelmpl()`：从 Token 的 Payload 中获取 identity 域的值，identity 域的值是在签发 Token 时指定的

```go
func (mw *GinJWTMiddleware) middlewareImpl(c *gin.Context) {
    // 1. 从 HTTP 请求中获取 Authorization Header，并解析出 Token 字符串，进行认证，最后返回 Token Payload
    claims, err := mw.GetClaimsFromJWT(c)
    if err != nil {
        mw.unauthorized(c, http.StatusUnauthorized, mw.HTTPStatusMessageFunc(err, c))
        return
    }

    if claims["exp"] == nil {
        mw.unauthorized(c, http.StatusBadRequest, mw.HTTPStatusMessageFunc(ErrMissingExpField, c))
        return
    }

    if _, ok := claims["exp"].(float64); !ok {
        mw.unauthorized(c, http.StatusBadRequest, mw.HTTPStatusMessageFunc(ErrWrongFormatOfExp, c))
        return
    }

    // 2. 校验 Payload 中的 exp 是否超过当前时间，如果超过就说明 Token 过期，校验不通过
    if int64(claims["exp"].(float64)) < mw.TimeFunc().Unix() {
        mw.unauthorized(c, http.StatusUnauthorized, mw.HTTPStatusMessageFunc(ErrExpiredToken, c))
        return
    }

    // 3. 给 gin.Context 中添加 JWT_PAYLOAD 键，供后续程序使用（当然也可能用不到）
    c.Set("JWT_PAYLOAD", claims)
    
    // 4. 在 gin.Context 中添加 IdentityKey 键，IdentityKey 键可以在创建GinJWTMiddleware结构体时指定
    identity := mw.IdentityHandler(c)

    if identity != nil {
    	c.Set(mw.IdentityKey, identity)
    }

    // 5. 调用 authorizator callbackbone 函数
    if !mw.Authorizator(identity, c) {
        mw.unauthorized(c, http.StatusForbidden, mw.HTTPStatusMessageFunc(ErrForbidden, c))
        return
    }

    c.Next()
}
```









#### LoginHandler 中间件

参考源码：[auth_jwt.go](https://github.com/appleboy/gin-jwt/blob/v2.6.4/auth_jwt.go#L431)

```go
// LoginHandler 登录认证
func (mw *GinJWTMiddleware) LoginHandler(c *gin.Context) {
    if mw.Authenticator == nil {
        mw.unauthorized(c, http.StatusInternalServerError, 
                        mw.HTTPStatusMessageFunc(ErrMissingAuthenticatorFunc, c))
        return
	}

    // 执行 Authenticator 完成 Basic 认证
	data, err := mw.Authenticator(c)
    if err != nil {
        mw.unauthorized(c, http.StatusUnauthorized, mw.HTTPStatusMessageFunc(err, c))
        return
    }

    // 认证通过，颁发 JWT Token
    token := jwt.New(jwt.GetSigningMethod(mw.SigningAlgorithm))
    claims := token.Claims.(jwt.MapClaims)

    // 执行 PayloadFunc 设置 Token Payload
    if mw.PayloadFunc != nil {
        for key, value := range mw.PayloadFunc(data) {
            claims[key] = value
        }
    }

    expire := mw.TimeFunc().Add(mw.Timeout)
    claims["exp"] = expire.Unix()
    claims["orig_iat"] = mw.TimeFunc().Unix()
    tokenString, err := mw.signedString(token)

    if err != nil {
        mw.unauthorized(c, http.StatusUnauthorized, mw.HTTPStatusMessageFunc(ErrFailedTokenCreation, c))
        return
    }

    // SendCookie == true，在 Cookie 中添加认证相关的信息
    if mw.SendCookie {
        expireCookie := mw.TimeFunc().Add(mw.CookieMaxAge)
        maxage := int(expireCookie.Unix() - mw.TimeFunc().Unix())

        if mw.CookieSameSite != 0 {
      		c.SetSameSite(mw.CookieSameSite)
    	}

        c.SetCookie(
            mw.CookieName,
            tokenString,
            maxage,
            "/",
            mw.CookieDomain,
            mw.SecureCookie,
            mw.CookieHTTPOnly,
        )
    }

    // 返回 Token 和 Token 过期时间给调用者
    // 前端可以将这些信息缓存在 Cookie 中或 LocalStorage 中，之后的请求都可以使用 Token 来进行认证
    mw.LoginResponse(c, http.StatusOK, tokenString, expire)
}
```

**Basic 认证过程**

```go
// authenticator 获取用户名和密码，并校验是否合法
func authenticator() func(c *gin.Context) (interface{}, error) {
    return func(c *gin.Context) (interface{}, error) {
        var login loginInfo
        var err error

        // support header and body both 解析用户名和密码
        if c.Request.Header.Get("Authorization") != "" {
            // 从 HTTP Authentication Header 解析
      		login, err = parseWithHeader(c)
        } else {
            // 从 Body 解析
      		login, err = parseWithBody(c)
        }
        if err != nil {
      		return "", jwt.ErrFailedAuthentication
        }

        // 校验用户名和密码
        // Get the user information by the login username.
        // 1. 从数据库查询用户名对应的加密后的密码
        user, err := store.Client().Users().Get(c, login.Username, metav1.GetOptions{})
        if err != nil {
      		log.Errorf("get user information failed: %s", err.Error())

      		return "", jwt.ErrFailedAuthentication
        }

        // Compare the login password with the user password.
        // 2. 密码加密后，与查询得到的密码进行比较
        if err := user.Compare(login.Password); err != nil {
      		return "", jwt.ErrFailedAuthentication
        }

        return user, nil
    }
}
```

`parseWithHeader` 解析：

1. 获取 `Authorization` 头的值，并调用 `strings.SplitN` 函数，获取一个切片变量 `auth`
2. 将 `auth` 进行 base64 解码，得到 `username:password`
3. 调用 `strings.SplitN` 函数，分别得到用户名和密码

`parseWithBody` 解析：调用 gin 的 `ShouldBindJSON` 函数，从 Body 中解析用户名和密码

获取用户名和密码后，可以从数据库中查询出该用户对应的加密后的密码



#### RefreshHandler 中间件

- 先进行 Bearer 认证，通过则重新签发 Token



#### LogoutHandler 中间件

```go
func (mw *GinJWTMiddleware) LogoutHandler(c *gin.Context) {
    // delete auth cookie
    if mw.SendCookie {
        if mw.CookieSameSite != 0 {
            c.SetSameSite(mw.CookieSameSite)
        }

        c.SetCookie(
            mw.CookieName,
            "",
            -1,
            "/",
            mw.CookieDomain,
            mw.SecureCookie,
            mw.CookieHTTPOnly,
        )
    }

    mw.LogoutResponse(c, http.StatusOK)
}
```

- 清空 Cookie 中 Bearer 认证相关信息





