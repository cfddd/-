## 用户登录和登出
如下代码举例，在登陆的过程中不需要使用jwt中间件验证
在登出的过程中，需要使用jwt验证

```go
func (router RouterGroup) UserRouter() {
	app := api.ApiGroupApp.UserApi
	router.Use(sessions.Sessions("sessionid", store))
	router.POST("login", app.QQLoginView)
	router.POST("logout", middleware.JwtAuth(), app.LogoutView)
	router.PUT("user_info", middleware.JwtAuth(), app.UserUpdateNickName)
}

```

登录之后，会给登陆的用户派发一个token，这个token里包含了用户的密码和有效时间等信息。
在登录之后，访问其他的api会携带token，自动通过jwt验证，成功访问。
如果没有登录，肯定是无法通过验证的。

浏览器实现免密登录一般是直接通过在这个页面留下缓存，在下次访问时，自动提交当前用户登陆的信息，以快速访问某些api，免去登录的过程。
更多的详细解释可以参考[理解Cookie和Session - Helldorado - 博客园 (cnblogs.com)](https://www.cnblogs.com/liyutian/p/10277817.html)

回到登出这个话题，在登出后，token是不会改变的，这就需要一个能够快速常看某个token是否登出的功能。redis就恰好可以实现这个功能。

在redis中记录一个token剩余有效期，每次登陆之前，jwt中间件判断redis中是否有当前用户的**有效**token**处于未过期（即存在，因为redis过期自动删除）**，如果有，表示当前用户已经登出了，需要重新登录；如果没有，就可以直接登录。
可能你还会有疑问，为什么要在登出之后把token存进redis，而不是登录就存。**有一个细节就是，一旦redis中的token过期了，那么显然这个token也已经到达了有效时间，自动作废。所以这样的设计是合理的。**
