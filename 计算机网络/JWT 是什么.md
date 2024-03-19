> [JSON Web Token 入门教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
> [JWT 基础概念详解 | JavaGuide](https://javaguide.cn/system-design/security/jwt-intro.html)

## cookie + session 跨域认证的问题
### 基本流程
> 1、用户向服务器发送用户名和密码。
> 
> 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
> 
> 3、服务器向用户返回一个 session_id，写入用户的 Cookie。
> 
> 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
> 
> 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

### 存在的问题
> **扩展性不好**——如果是服务器集群，或者是跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。
> 
> 比如，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站如何自动登录？

### 解决方法
 1. session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。
	 这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。
2. 服务器不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。**JWT 就是这种方案的一个代表。**

**JWT 让服务器不保存任何 session 数据，即服务器变成无状态，从而比较容易实现扩展。**
## JWT 的原理

JWT 本质上就是一组字串，通过（`.`）切分成三个为 **Base64 编码**的部分：

- **Header** : 描述 JWT 的元数据，定义了生成签名的算法以及 `Token` 的类型。
- **Payload** : 用来存放实际需要传递的数据
- **Signature（签名）**：服务器通过 Payload、Header 和一个密钥(Secret)使用 Header 里面指定的签名算法（默认是 HMAC SHA256）生成。
> **注意：** JWT 不是加密后的数据，可以直接通过Base64翻译出来

### Header
Header 通常由两部分组成：
- `typ`（Type）：令牌类型，也就是 JWT。
- `alg`（Algorithm）：签名算法，比如 HS256。

JSON 形式的 Header 被转换成 Base64 编码，成为 JWT 的第一部分。
### Payload
Payload 也是 JSON 格式数据，其中包含了 Claims(声明，包含 JWT 的相关信息)。

Payload 部分默认是不加密的，**一定不要将隐私信息存放在 Payload 当中！！！**

JSON 形式的 Payload 被转换成 Base64 编码，成为 JWT 的第二部分。

### Signature
Signature 部分是对前两部分的签名，作用是防止 JWT（主要是 payload） 被篡改。

这个签名的生成需要用到：

- Header + Payload。
- 存放在服务端的密钥(一定不要泄露出去)。
- 签名算法。
## JWT 使用
服务端发送合法的 JWT 给客户端

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage，之后每次与服务器通信，都要带上这个 JWT。

**建议放在客户端请求头部 Authorization 字段**
## JWT 注意事项
1、 JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。因此，不要把敏感信息放到未加密的 JWT 中

2、 JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

3、 JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

4、 JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

5、 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。
