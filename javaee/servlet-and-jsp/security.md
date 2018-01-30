# 身份验证的几种方案【转载】
http://www.bijishequ.com/detail/258307


现在大部分的网站都有用户系统，有些事情只能登陆之后才能做，比如发一条微博。有了用户系统就会有身份验证，本篇记录常用的客户端和服务器的身份验证方案，以备不时之需。

典型的用户身份验证标准（方案）：

* HTTP BASIC Authentication
* HTTP Digest Authentication
* Form-based Authentication
* Token Based Authentication
* X.509 Certificate Authentication

通常情况下用户认证失败在HTTP协议中的表现是："401，Access Denied"

# HTTP BASIC Authentication
什么是 HTTP Basic Authentication？见Basic_access_authentication ,在真实场景中的表现是：当用访问需要登录验证的页面时，浏览器会自动弹出一个对话框，要求输入用户名/密码，输入正确后可以正常访问。

在这种方式，浏览器会把用户名和密码通过BASE64编码在HTTP HEAD 里面
```
Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l
```
服务器端解析之后做身份验证，并给客户端返回
```
WWW-Authenticate: Basic realm="User Visible Realm"
```
客户端每次请求都会携带用户名密码，需要通过HTTPs来保证安全。另外客户端需要缓存用户名和密码，以保证不必每次请求都要用户重新输入用户名和密码，通常浏览器会在本地保存10分钟左右的时间，超过之后需要用户再次输入用户名密码。

这是基于HTTP协议的比较传统的身份验证方案，现在已经很少使用。

# HTTP Digest Authentication
具体见维基：Digest Access Authentication

Digest authentication 是对前面 Basic access authentication 的升级版本，它不再使用Base64的用户名/密码而是对于用户名密码做哈希取得一个摘要
字符串再传给服务器，这样在传输的过程中不会暴露用户名和密码。

# Form-based Authentication
目前为止我们在登陆网页时看到的登陆页面基本都是基于Form-based Authentication，是最流行的身份验证方式。

当用户访问一个未授权网页的时候，服务器会返回一个登陆页面，用户输入用户名/密码并点击提交按钮，浏览器把表单信息发送给服务器，服务器验证之后创建Session，并把Cookie返回给浏览器。在下次请求的时候，浏览器会把Cookie附加在每个请求的HEAD里面，服务器通过验证Cookie来校验接下来的请求。**由于表单信息是明文传输的，所以需要额外的措施来保证安全（比如：HTTPS）。**

PS：网上有时候叫做 Cookie-Based Authentication

# Token Authentication
这种授权方式源于OAuth，现在在单页面应用(SPA)中逐渐流行起来（普遍应用在移动App中）。它的大概过程和基于Form/Cookie的授权方式一致，客户端
发送用户名/密码给服务器，服务器返回一个Token（token包含一个过期时间）给客户端
```
{
    "refresh_token":"xxxx"
    "token": "xxxxx"
}
```
客户端拿到Token之后被缓存在本地，以后每次请求的时候在HEAD里面带上Token，这样服务器便可以验证客户端, 如果Token过期客户端可以通过RefreshToken再次获取新的Token。。
```
Authorization: xxxx
```
通常在基于Cookie的身份验证中，Cookie存储的是SessionId，服务器端需要通过Session来维护会话的状态。但是在SPA或者移动类的REST应用中，状态在本地维护一般使用token来实现无状态的服务器，简化服务器端的逻辑。

更多关于Token和Cookie的对比看下面两篇文章：

Token Based Authentication for Single Page Apps (SPAs)

Cookies vs Tokens. Getting auth right with Angular.JS

# X.509 Certificate Authentication
这种验证方式在面向普通大众的Web服务中很少见到，但是在开发人员中比较流行。比如使用Git给Github上的Repo提交代码，需要提前在Github网站上配置公钥并在本地~/.ssh目录配置私钥。这就是典型的证书验证配置。

另外一种典型应用是HTTPS，但是这里证书的配置并不是为了验证用户身份，而是为了验证服务器的身份同时建立安全的连接（SSL/TLS）。一般情况下，服务器提供的证书是由特殊的CA结构（持有根证书）认证的，而浏览器在发布的时候都会提前预值根证书，这样当用户用浏览器访问一个网页的时候，浏览器会用根证书验证服务器端的证书以确认服务器不是伪造的（或者是中间人）。