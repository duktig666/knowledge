## 什么是OAuth？

> **OAuth 引入了一个授权层，用来分离两种不同的角色：客户端和资源所有者。......资源所有者同意以后，资源服务器可以向客户端颁发令牌。客户端通过令牌，去请求数据。**

**OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。**

OAuth2.0是OAuth协议的延续版本，但不向前兼容OAuth 1.0(即完全废止了OAuth1.0)。

## 应用场景举例

> 有一个"云冲印"的网站，可以将用户储存在Google的照片，冲印出来。用户为了使用该服务，必须让"云冲印"读取自己储存在Google上的照片。
>
> 问题是只有得到用户的授权，Google才会同意"云冲印"读取这些照片。那么，"云冲印"怎样获得用户的授权呢？

传统方法是，用户将自己的Google用户名和密码，告诉"云冲印"，后者就可以读取用户的照片了。这样的做法有以下几个严重的缺点。

> （1）**"云冲印"为了后续的服务，会保存用户的密码，这样很不安全**。
>
> （2）**Google不得不部署密码登录，而我们知道，单纯的密码登录并不安全**。
>
> （3）**"云冲印"拥有了获取用户储存在Google所有资料的权力，用户没法限制"云冲印"获得授权的范围和有效期**。
>
> （4）**用户只有修改密码，才能收回赋予"云冲印"的权力。但是这样做，会使得其他所有获得用户授权的第三方应用程序全部失效**。
>
> （5）**只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有被密码保护的数据泄漏**。

OAuth就是为了解决上面这些问题而诞生的。

## OAuth相关名词定义

>
> （1） **Third-party application**：第三方应用程序，又称"客户端"（client），即例子中的"云冲印"。
>
> （2）**HTTP service**：HTTP服务提供商，简称"服务提供商"，即例子中的Google。
>
> （3）**Resource Owner**：资源所有者，又称"用户"（user）。
>
> （4）**User Agent**：用户代理，本文中就是指浏览器。
>
> （5）**Authorization server**：认证服务器，即服务提供商专门用来处理认证的服务器。
>
> （6）**Resource server**：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

OAuth的作用就是让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。

![OAuth2.0主要角色](https://cos.duktig.cn/typora/202109011416017.png)

## OAuth的思路

OAuth在"客户端"与"服务提供商"之间，设置了一个**授权层（authorization layer）**。

"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。

"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。

"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

## OAuth的运行流程

![抽象OAuth协议流程](https://cos.duktig.cn/typora/202109011137923.png)

> （A）用户打开客户端以后，客户端要求用户给予授权。
>
> （B）用户同意给予客户端授权。
>
> （C）客户端使用上一步获得的授权，向认证服务器申请令牌。
>
> （D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
>
> （E）客户端使用令牌，向资源服务器申请获取资源。
>
> （F）资源服务器确认令牌无误，同意向客户端开放资源。

不难看出来，上面六个步骤之中，**B是关键，即用户怎样才能给于客户端授权**。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。

![授权流程例子](https://cos.duktig.cn/typora/202109011419950.jpg)

客户端获取授权有四种模式。



## OAuth 2.0 的四种方式

OAuth 2.0 对于如何颁发令牌的细节，规定得非常详细。具体来说，一共分成四种授权类型（authorization grant），即四种颁发令牌的方式，适用于不同的互联网场景：

- 授权码（authorization code）
- 简化模式（隐藏式）（implicit）
- 密码式（password）
- 客户端凭证（client credentials）

注意，不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：**客户端 ID**（client ID）和**客户端密钥**（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。

### 一、授权码模式

> **授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**
>
> 功能最完整、流程最严密的授权模式。

这种方式是最常用的流程，安全性也最高，它**适用于那些有后端的 Web 应用**。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

![授权码模式-RFC 6749](https://cos.duktig.cn/typora/202109011142456.png)

![授权码方式流程](https://cos.duktig.cn/typora/202109010957694.png)

第一步，A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。下面就是 A 网站跳转 B 网站的一个示意链接。

```javascript
https://b.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

- `response_type`参数表示要求返回授权码（`code`）
- `client_id`参数让 B 知道是谁在请求
- `redirect_uri`参数是 B 接受或拒绝请求后的跳转网址
- `scope`参数表示要求的授权范围（这里是只读）。

第二步，用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A 网站授权。用户表示同意，这时 B 网站就会跳回`redirect_uri`参数指定的网址。跳转时，会传回一个授权码，就像下面这样。

```javascript
https://a.com/callback?code=AUTHORIZATION_CODE
```

- `code`参数就是授权码

第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。

```javascript
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

- `client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求）
- `grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码
- `code`参数是上一步拿到的授权码
- `redirect_uri`参数是令牌颁发后的回调网址。

第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。

```javascript
{    
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。

### 二、简化模式（隐藏式）

> 有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**
>
> 所有步骤在浏览器（前端）中完成，令牌对访问者是可见的，且客户端不需要认证。

![简化模式-RFC 6749](https://cos.duktig.cn/typora/202109011358233.png)

![隐藏式流程](https://cos.duktig.cn/typora/202109011001308.png)

第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。

> ```javascript
> https://b.com/oauth/authorize?
>   response_type=token&
>   client_id=CLIENT_ID&
>   redirect_uri=CALLBACK_URL&
>   scope=read
> ```

上面 URL 中，`response_type`参数为`token`，表示要求直接返回令牌。

第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。

> ```javascript
> https://a.com/callback#token=ACCESS_TOKEN
> ```

上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。

注意，令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），**这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险**。

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

### 三、密码模式

> **如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**
>
> 用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。
>
> 在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分，或者由一个著名公司出品。而认证服务器只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。

![密码模式-RFC 6749](https://cos.duktig.cn/typora/202109011404961.png)

第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。

> ```javascript
> https://oauth.b.com/token?
>   grant_type=password&
>   username=USERNAME&
>   password=PASSWORD&
>   client_id=CLIENT_ID
> ```

上面 URL 中，`grant_type`参数是授权方式，这里的`password`表示"密码式"，`username`和`password`是 B 的用户名和密码。

第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。

这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

### 四、客户端模式（凭证式）

> **客户端模式（client credentials），指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证**。严格地说，客户端模式并不属于OAuth框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。
>
> **适用于没有前端的命令行应用，即在命令行下请求令牌。**

![客户端模式-RFC 6749](https://cos.duktig.cn/typora/202109011406349.png)

第一步，A 应用在命令行向 B 发出请求。

> ```javascript
> https://oauth.b.com/token?
>   grant_type=client_credentials&
>   client_id=CLIENT_ID&
>   client_secret=CLIENT_SECRET
> ```

上面 URL 中，`grant_type`参数等于`client_credentials`表示采用凭证式，`client_id`和`client_secret`用来让 B 确认 A 的身份。

第二步，B 网站验证通过以后，直接返回令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

## 令牌

### 令牌类型

1. **授权码(Authorization Code Token)** ：仅用于授权码授权类型，用于交 换获取访问令牌和刷新令牌 。

2. **刷新令牌(Refresh Token)** ：用于去授权服务器获取一个新的访问令牌

3. **访问令牌(Access Token)** ：用于代表一个用户或服务直接去访问受保护的资源

4. **Bearer Token** ：不管谁拿到Token都可以 访问资源，像现钞 

5. **Proof of Possession(PoP) Token** 可以校验client是否对Token 有明确的拥有权 

### 令牌的使用

A 网站拿到令牌以后，就可以向 B 网站的 API 请求数据了。

此时，每个发到 API 的请求，都必须带有令牌。具体做法是在请求的头信息，加上一个`Authorization`字段，令牌就放在这个字段里面。

> ```bash
> curl -H "Authorization: Bearer ACCESS_TOKEN" \
> "https://api.b.com"
> ```

上面命令中，`ACCESS_TOKEN`就是拿到的令牌。

### 更新令牌

令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。

具体方法是，B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token 字段）。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。

> ```javascript
> https://b.com/oauth/token?
>   grant_type=refresh_token&
>   client_id=CLIENT_ID&
>   client_secret=CLIENT_SECRET&
>   refresh_token=REFRESH_TOKEN
> ```

上面 URL 中，`grant_type`参数为`refresh_token`表示要求更新令牌，`client_id`参数和`client_secret`参数用于确认身份，`refresh_token`参数就是用于更新令牌的令牌。

B 网站验证通过以后，就会颁发新的令牌。

### 令牌（token）与密码（password）的区别与联系？

令牌（token）与密码（password）的作用是一样的，都可以进入系统，但是有三点差异：

1. 令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。
2. 令牌可以被数据所有者撤销，会立即失效。密码一般不允许被他人撤销。
3. 令牌有权限范围（scope）。对于网络服务来说，只读令牌就比读写令牌更安全。密码一般是完整权限。

OAuth 2.0 的优点：保证了令牌既可以让第三方应用获得权限，同时又随时可控，不会危及系统安全。

注意，只要知道了令牌，就能进入系统。系统一般不会再次确认身份，所以**令牌必须保密，泄漏令牌与泄漏密码的后果是一样的。** 这也是为什么令牌的有效期，一般都设置得很短的原因。

## **OAuth2**授权认证中心架构

### 传统单体应用架构应用安全

![传统单体应用安全架构图](https://cos.duktig.cn/typora/202109011423230.png)

### 微服务应用架构应用安全

![微服务应用安全架构图](https://cos.duktig.cn/typora/202109011424719.png)



![OAuth2.0应用安全架构简图](https://cos.duktig.cn/typora/202109011425268.png)

## 第三方登录

> 所谓第三方登录，实质就是 OAuth 授权。用户想要登录 A 网站，A 网站让用户提供第三方网站的数据，证明自己的身份。获取第三方网站的身份数据，就需要 OAuth 授权。

### GitHub登录

举例来说，A 网站允许 GitHub 登录，背后就是下面的流程。

> 1. A 网站让用户跳转到 GitHub。
> 2. GitHub 要求用户登录，然后询问"A 网站要求获得 xx 权限，你是否同意？"
> 3. 用户同意，GitHub 就会重定向回 A 网站，同时发回一个授权码。
> 4. A 网站使用授权码，向 GitHub 请求令牌。
> 5. GitHub 返回令牌.
> 6. A 网站使用令牌，向 GitHub 请求用户数据。



## 参看

- 《阮一峰的网络日志》
  - [OAuth 2.0 的一个简单解释](https://www.ruanyifeng.com/blog/2019/04/oauth_design.html) （以快递员问题举例解释）
  - [理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) （以“云冲印”谷歌照片为例解释；介绍一些概念和四种授权方式）
  - [OAuth 2.0 的四种方式 ](https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html) （更为直观的方式解释四种授权方式）
  - [GitHub OAuth 第三方登录示例教程](https://www.ruanyifeng.com/blog/2019/04/github-oauth.html)
- [RFC 6749](http://www.rfcreader.com/#rfc6749)
- [【分布式认证系统】OAuth2.0标准、SpringSecurity与SpringSecurityOAuth2之间的区别与联系](https://www.makeyourchoice.cn/archives/348/)
- [OAuth2案例应用（附参考开源代码）](https://blog.csdn.net/haponchang/article/details/94431147)


