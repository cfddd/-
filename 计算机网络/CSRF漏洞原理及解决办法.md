> [什么是CSRF？如何防御CSRF攻击？知了堂告诉你 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/343515825)
> 
> [CSRF漏洞原理攻击与防御（非常细）-CSDN博客](https://blog.csdn.net/qq_43378996/article/details/123910614)
> 
> [跨站请求伪造CSRF攻击的原理以及防范措施_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1UH4y1M7Dg/?spm_id_from=333.337.search-card.all.click&vd_source=2d885cb62bb9393fa8a5379c72eabd82)

## 什么是CSRF
CSRF（Cross-Site Request Forgery），也被称为 one-click attack 或者 session riding，即**跨站请求伪造攻击**。

CSRF是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。

可以简单的理解为：攻击者可以盗用你的登陆信息，以你的身份模拟发送各种请求对服务器来说这个请求是完全合法的，但是却完成了攻击者所期望的一个操作，比如以你的名义发送邮件、发消息，盗取你的账号，添加系统管理员，甚至于购买商品、虚拟货币转账等。攻击者只要借助少许的社会工程学的诡计，例如通过 QQ 等聊天软件发送的链接（有些还伪装成短域名，用户无法分辨），攻击者就能迫使 Web 应用的用户去执行攻击者预设的操作。
## CSRF的防御方法

### 验证HTTP Referer字段

根据 HTTP 协议，在 HTTP 头中有一个字段叫Referer，它记录了该 HTTP 请求的来源地址。

> 比如我们google搜索出来有网站A，我们点击后跳转到网站A，请求体就中有来源地址：www.google.com

在通常情况下，访问一个安全受限页面的请求来自于同一个网站（比如银行转账）。

对于每一个转账请求验证其 Referer 值，如果是合法网站的域名，则说明该请求是来自有效网站自己的请求，是合法的；

如果 Referer 是其他未知网站的话，则有可能是黑客的 CSRF 攻击，拒绝该请求。

但是使用验证 Referer 值的方法，就是把安全性都依赖于第三方（即浏览器）来保障，从理论上来讲，这样并不安全。
### 使用 CSRF 令牌（CSRF Token）

CSRF 令牌是一种**基于随机数生成的验证机制**。

服务器在向用户发送页面时，会生成一个随机的 CSRF 令牌，并将其包含在页面中（**通常是隐藏的表单字段或 HTML5 的自定义属性**）。

当用户提交表单或执行操作时，这个令牌会随请求一起发送回服务器。服务器收到请求后，会验证令牌是否与之前生成并存储的令牌一致，如果一致则认为请求是合法的，否则可能是 CSRF 攻击。

同时需要确保这个随机数不容易被窃取或泄露；定期更换随机数，以防止长时间使用相同的随机数。
#### 在非GET请求中增加token并验证
要抵御 CSRF关键在于在**请求中放入黑客所不能伪造的信息**，并且该信息不存在于 cookie 之中。

在HTTP请求中加入一个随机产生的token，并在服务器端建立一个拦截器来验证这个token。如果请求中没有token或者token内容不正确，则认为可能是CSRF攻击而拒绝该请求。

1. **生成随机token**：
    
    - 在用户登录时，服务器为用户生成一个随机的token，并将其存储在用户的会话中（通常是在cookie中）。
    - 这个token是一个伪随机值，每次用户登录或会话开始时都会重新生成。
    - 服务器还将这个token添加到每个表单中，**作为隐藏字段或请求头的一部分**。
2. **验证token**：
    
    - 当用户提交一个非GET请求（例如POST请求）时，服务器会检查请求中的token是否与用户会话中存储的token匹配。
    - 如果匹配成功，服务器会处理请求；如果不匹配，服务器会拒绝请求。
    - 这样，即使攻击者能够伪造请求，也无法伪造有效的token，因为他们无法获取用户的会话信息。
3. **保持token的安全性**：
    
    - 确保token不容易被窃取或泄露。使用安全的cookie设置，例如将token标记为`HttpOnly`，以防止JavaScript访问它。
    - 定期更换token，以防止长时间使用相同的token。
### SameSite Cookie 属性
SameSite 是 Cookie 的一个属性，用于限制 Cookie 在跨站请求中的发送方式。当 SameSite 属性设置为`Strict`或`Lax`时，可以在一定程度上防止 CSRF 攻击。
- **Strict 模式**：在`Strict`模式下，只有当请求来自于与设置 Cookie 的网站完全相同的源（包括协议、域名和端口）时，浏览器才会在请求中发送 Cookie。这意味着如果一个恶意网站试图发起 CSRF 攻击，由于浏览器不会发送用户在目标网站的登录 Cookie，攻击将无法成功。
- **Lax 模式**：`Lax`模式相对宽松一些，允许在一些安全的（如GET请求）跨站请求（如导航到目标网站的链接）中发送 Cookie，但对于大多数跨站请求（如 POST 请求）仍然限制 Cookie 的发送。

## 小结
- CSRF也被称为**跨站请求伪造攻击**
- 常见的抵御方法有：
	- **HTTP Refer**字段：判断请求的来源是否是用户的域名/ip
	- **CSRF令牌**：基于随机数，在隐藏的表单中添加，放到Http 的头部，让攻击者难以识别，但是服务端必须要去验证
	- **SameSite Cookie**属性：大部分浏览器支持，cookie只能给目标网站使用，禁止网页私自转发。有两种严格的模式

## 实操

### 传统前端设置Cookie
是否会自动携带 cookie，**关键取决于 `http` 这个请求工具（比如 axios）默认的配置**。

- **【同域调用】**：**浏览器只会自动携带与当前请求域名匹配且符合路径、SameSite 等条件的 cookie**，即同域（协议+域名+端口都相同），浏览器会自动带 cookie，无需额外配置
- **【跨域调用】**：**前端发起跨域请求时，需要额外设置**，在 axios 或 fetch 中显式配置 `withCredentials: true`（fetch 对应 `credentials: 'include'`）

### 使用 HttpOnly Cookie 存储 Token 的方案（推荐做法）

让后端在登录成功后，把 token 写入一个带有 `HttpOnly` 属性的 Cookie 中，这样前端无法通过 JS 访问 token，但浏览器请求时会自动携带它，从而完成身份认证。
#### 前端流程

1. 用户填写用户名 + 密码，点击登录
2. 发起登录请求：`POST /api/login`
3. 登录成功后，后端 **通过 `Set-Cookie` 返回带有 HttpOnly 的 token**
4. 前端**拿不到 token**，但浏览器之后发起请求时，会自动携带 cookie

```js
// 登录请求（注意 credentials）
fetch('/api/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  credentials: 'include', // 关键：让浏览器携带 cookie
  body: JSON.stringify({
    username: 'admin',
    password: '123456'
  })
})

```

#### 后端流程（伪代码）
```js
// 假设这是 Node.js/Express 或类似后端框架
res.cookie('token', accessToken, {
  httpOnly: true,     // JS 不能访问（防止 XSS）
  secure: true,       // 只在 HTTPS 下传输
  sameSite: 'Strict', // 防止跨站请求伪造（CSRF）
  maxAge: 1000 * 60 * 60 * 24 * 7 // 有效期 7 天
})
res.json({ success: true })

```
之后：
- 每个请求浏览器都会自动携带这个 cookie
- 后端可以读取 `req.cookies.token` 来验证身份