## HTTP 无状态协议

HTTP 本身是一个无状态的连接协议，无状态的意思是：每条请求/响应都是独立进行的，服务端每处理完一个客户端的请求之后就会断开连接，并且每条请求/响应与其之前（或之后）的请求/响应是没有任何关系的。即HTTP自身不具备保存之前发送过的请求或者响应的功能。

比如当我们打开微博的登录页，输入用户名和密码之后，浏览器将这些数据发送给服务器端，服务端验证成功之后，跳转到了我们的微博首页，但是接着如果我们希望查看自己聊天消息，由于 HTTP 的这种无状态性，服务端将无法知道我们在上一次HTTP请求中已经通过了验证，从而无法正确的处理这次请求。

为了解决这个问题。我们首先想到的办法是可以在所有的请求中都带上用户名和密码，这种方法虽然可行，但也大大加重了服务器的负担（每个请求都需到数据库验证）。更好的办法其实是使用 Cookie 与 Session 机制。

在 Web 应用中跟踪用户状态的办法有以下四种：

- 建立包含有跟踪数据的隐藏字段（比如使用`<input type="hidden">`）
- 重写包含额外参数的URL（文末会介绍）
- 使用持续的 Cookie
- 使用 Session

Session 和 Cookie 是 Web 程序中较为常见的两个概念。它们的目的相同，都是为了克服 http 协议无状态的问题。

## Cookie

Cookie 实际上是一小段由客户端（如浏览器）保存在本地的文本文件，它记录了你的用户ID、密码、浏览过的网页、浏览时间等信息。当客户端再次访问同一个网站时，客户端就会把请求连同 Cookie 一起发送给服务端，服务端通过检查该 Cookie，就可以辨认用户状态了。

### Cookie 的传递流程

Cookie 是 HTTP 协议头的一部分，用于浏览器和服务器之间传递信息。

当客户端第一次向服务端发起请求时，服务端会创建一个 Cookie，并通过 HTTP 响应头中的 Set-Cookie 属性，来将 Cookie 信息返回给客户端，并通知客户端保存起来。

当客户端再次向服务端发起请求时，客户端就会把之前保存在本地的 Cookie 放入请求头中一并发送给服务端，服务端获得 Cookie 之后，就可以得到客户端的状态信息了。

![](https://img2018.cnblogs.com/blog/662236/201901/662236-20190116163735373-1219268861.png)

1.请求报文（没有Cookie信息时）

```vbnet
GET /index HTTP/1.1
Host: www.baidu.com
* 首部字段内没有Cookie的相关信息
```

2.响应报文（服务器端生成 Cookie 信息）

```xml
HTTP/1.1 200 OK
Server: Apache
<Set-Cookie: sid=57111807181018; path=/;expires=Web,10-OCT-12 07:12:20 GMT>
Content-Type: text/plain; charset=UTF-8
```

3.请求报文（自动发送保存着的 Cookie 信息）

```vbnet
GET /image/ HTTP1.1
HOST: www.baidu.com
Cookie: sid=57111807181018
```

Cookie 中主要包含了NAME（名称）、path（路径）、domain（域名）、expires（有效期）、max-age（过期时间）几种不同的属性，它们各自起到不同的作用。例如通过设置 Cookie 的 maxAge 属性可以设置 Cookie 的过期时间，不设置 maxAge 被称为会话 Cookie，会话 Cookie 生命周期为从创建浏览器到关闭浏览器为止，只要关闭浏览器窗口，会话 Cookie 就会消失，它一般保存在内存中，而不是硬盘中。

如果设置了过期时间（setMaxAge(60*60*24)）,浏览器就会把 Cookie 保存在硬盘上，对于关闭后重新打开的浏览器，这些 Cookie 依旧有效。

cookie过期时间设置方式：

cookie.setMaxAge(0);//不记录cookie

cookie.setMaxAge(-1);//会话级cookie，关闭浏览器失效

cookie.setMaxAge(60*60);//过期时间为1小时

## Session

除了使用 Cookie 之外，Web 应用程序还经常使用 Session 来跟踪用户状态，与 Cookie 保存在客户端浏览器不同的是，Session 是将状态信息保存在了服务端上。服务端使用一种类似散列表的结构（也可能就是使用散列表）来保存信息。

### session的工作原理

1. 客户端第一次向服务器端发送请求时，服务端程序会为此客户端创建一个session，并生成一个与此session相关联的sessionId。（sessionId 的值应该是一个不会重复并且难以伪造的字符串）
    
2. 服务端向客户端返回响应时，同时会将 sessionId 一起返回给客户端，客户端会将 sessionId 字符串保存下来。
    
3. 当客户端再次访问服务端时，将 sessionId 一并发送给服务端。
    
4. 服务端获取从客户端发送过来的 sessionId，就可以根据这个 id 获取保存在服务器中相应的数据了
    
5. 当其他客户端也访问服务端时，就又会产生一个新的sessionId，类似以上的步骤进行处理。
    

客户端保存 sessionId 的方式可以是使用 Cookie。这样在交互过程中，浏览器就可以自动按照规则把这个标识发送给服务端。除此之外，也可以通过使用表单隐藏字段的方式，来将sessionId 传递给服务端，比如：`<input type="hidden" name="sessionid" value="5711A1B807F18B1018">`。

又或者还可以通过 URL 重写来代替。所谓 URL 重写就是在返回给用户的页面里的所有的 URL 后面都追加 Session 标识符，这样用户在收到响应之后，无论点击响应页面里的哪个链接或提交表单，都会自动带上 Session 标识符。

经常有这样一种误解说的是当浏览器关闭后，session 也就被销毁了，实际上这个说法并不准确。  
这是因为 sessionId 通常是被当作会话 Cookie 进行处理的，所以当我们关闭浏览器后，sessionId 也就销毁了，所有也就无法找到原来的 session 了。如果服务器设置的 Cookie 被保存在了硬盘上，或者直接改写了浏览器发出的 HTTP 请求头，把原来的 sessionId 发送给了服务端，那么这是仍然能够找到原来的 session 的。

## **小结：**
Session是另一种记录客户状态的机制，不同的是**Cookie保存在客户端浏览器中，而Session保存在服务器上**。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。


参考：  
[https://www.cnblogs.com/andy-zhou/p/5360107.html](https://www.cnblogs.com/andy-zhou/p/5360107.html)  
[https://justsee.iteye.com/blog/1570652](https://justsee.iteye.com/blog/1570652)
[理解Cookie和Session - Helldorado - 博客园 (cnblogs.com)](https://www.cnblogs.com/liyutian/p/10277817.html)
[彻底理解cookie，session，token - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/63061864)
[彻底了解Cookie和Session的区别（面试）_session和cookie的区别-CSDN博客](https://blog.csdn.net/weixin_45393094/article/details/104747360)
