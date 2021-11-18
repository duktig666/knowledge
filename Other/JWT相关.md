## 什么是JWT？

JWT简称JSON Web Token,也就是通过JSON形式作为Web应用中的令牌,用于在各方之间安全地将信息作为JSON对象传输。在数据传输过程中还可以完成数据加密、签名等相关处理。

它最重要的特性就是，为了确认它是否有效，我们只需要看JWT本身的内容，而不需要借助于第三方服务或者在多个请求之间将其保存在内存中-这是因为它本身携带了信息验证码MAC(Message Authentication Code)。

## JWT的组成

1. 标头(Header)

2. 有效载荷(Payload)

3. 签名(Signature)

JWT通常如下所示:xxxxx.yyyyy.zzzzz  Header.Payload.Signature

### Header

- 标头通常由两部分组成：令牌的类型（即JWT）和所使用的签名算法，例如HMAC SHA256或RSA。它会使用 Base64 编码组成 JWT 结构的第一部分。 
-  注意:Base64是一种编码，也就是说，它是可以被翻译回原来的样子来的。它并不是一种加密过程。 

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

令牌的第二部分是有效负载，其中包含声明。声明是有关实体（通常是用户）和其他数据的声明。同样的，它会使用 Base64 编码组成 JWT 结构的第二部分

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

```java
iss (issuer)：表示签发人
exp (expiration time)：表示token过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

JWT的Payload理论上可以存放任何内容，不一定是用户身份信息，只不过使用JWT作为认证是最常用的方式。

### Signature

前面两部分都是使用 Base64 进行编码的，即可以解开知道里面的信息。Signature 需要使用编码后的 header 和 payload 以及我们提供的一个密钥，然后使用 header 中指定的签名算法（HS256）进行签名。签名的作用是保证 JWT 没有被篡改过 。如:

HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload),secret); 

最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。

## JWT的优缺点

### 优点

- 简洁(Compact): 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快
- 自包含(Self-contained)：负载中包含了所有用户所需要的信息，避免了多次查询数据库

- 因为Token是以JSON加密的形式保存在客户端的，所以JWT是跨语言的，原则上任何web形式都支持。
- 不需要在服务端保存会话信息，特别适用于分布式微服务。

### 缺点

注销登录等场景下 token 还有效

token 的续签问题

## **JWT的目标是让服务器无状态?**

JWT真正的好处是让认证服务器和校验JWT token的应用服务器可以完全分开，而让服务器无状态化只是它的一个副作用罢了。这意味着应用服务器只需要最简单的认证逻辑-校验JWT！我们可以将整个应用集群的登录/注册委托给一个单独的认证服务器。这也意味着应用服务器更简单更安全，因为更多的认证功能集中部署在认证服务器，可以被跨应用使用。

