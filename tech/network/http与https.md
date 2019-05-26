## HTTP

### 1. HTTP 协议发展历史

#### 1.1 HTTP/0.9

- 只有一个命令 GET

- 没有 HEADER 等描述数据的信息

- 服务器发送完毕，就关闭 TCP 连接

#### 1.2 HTTP/1.0

- 增加了很多命令

- 增加 status code 和 header

- 多字符集支持、多部分发送、权限、缓存等


#### 1.4 HTTP/1.1

- 持久连接

- pipeline

- 增加 host 和 其他一些命令


#### 1.5 HTTP/2.0

- 所有数据以二进制传输

- 同一个连接里面发送多个请求不再需要按照顺序来

- 头信息压缩以及推送等提高效率的功能

#### 2. HTTP 相关知识

#### 2.1 GET 请求 和 POST 请求的区别

| 三个层面 | GET | POST |
|---|---|---|
|HTTP报文层面| GET 将请求信息放在URL | POST 将请求信息放在报文体中 |
|数据层面| GET 符合幂等性和安全性| POST 不符合|
|其他层面| GET 可以被缓存、被存储| POST 不行|

#### 2.2 Cookie 和 Session 的介绍及其区别

2.2.1 Cookie 简介

- 是否服务器发给客户端的特殊信息，以文本的形式存放在客户端

- 客户端再次请求的时候，会把 Cookie 回发

- 服务器收到后，会解析 Cookie 生成与客户端相对应的内容

2.2.2 Cookie 的设置以及发送过程

- 客户端 -> 服务端：HTTP Request

- 客户端 <- 服务端：HTTP Response + Set-Cookie

- 客户端 -> 服务端：HTTP Request + Cookie

- 客户端 <- 服务端：HTTP Response

2.2.3 Session 简介

- 服务端的机制，在服务器上保存的信息

- 解析客户端请求并操作 session id，按需保存状态信息

2.2.4 Session 的实现方式

- 使用 Cookie 来实现

- 使用 URL 回写来实现

Cookie 实现，如下：

- 客户端 -> 服务端：HTTP Request

- 客户端 <- 服务端：HTTP Response + Set-Cookie: JSESSIONID=xxxxx

- 客户端 -> 服务端：HTTP Request + Cookie: JSESSIONID=xxxxx

- 客户端 <- 服务端：HTTP Response

2.2.5 Cookie 和 Session 的区别

- Cookie 数据存放在客户的浏览器上，Session 数据放在服务器上

- Session 相对于 Cookie 更安全

- 若考虑减轻服务器负担，应当使用 Cookie



### 3. URI、URL 和 URN

#### 3.1 URI

- Uniform Resource Identifier / 统一资源标识符

- 用来唯一标识互联网上的信息资源

- 包括 URL 和 URN


#### 3.2 URL

- Uniform Resource Locator / 统一资源定位器

- http://user:pass@host.com:80/path?query=string#hash

- 此类格式都叫做 URL，比如 ftp协议



#### 3.3 URN

- 永久统一资源定位符

- 在资源移动之后还能被找到

- 目前还没有非常成熟的使用方案



### 4. HTTP 和 HTTPS 的区别

#### 4.1 HTTPS 简介
```
    HTTP               HTTPS

+------------+      +-----------+
|            |      |   HTTP    |
|    HTTP    |      |           |
|            |      +-----------+
+------------+      |SSL or TLS |
|            |      |           |
|    TCP     |      +-----------+
|            |      |    TCP    |
+------------+      |           |
|            |      +-----------+
|     IP     |      |    IP     |
|            |      |           |
+------------+      +-----------+

```

4.1.1 SSL（Security Sockets Layer，安全套接层）

- 为网络通信提高安全及数据完整性的一种安全协议

- 是操作系统对外的API， SSL 3.0 后更名为 TLS

- 采用身份验证和数据加密保证 网络通信的安全 和 数据的完整性

4.1.2 加密的方式

- 对称加密：加密和解密都使用同一个密钥

- 非对称加密：加密使用的密钥和解密使用的密钥是不相同的

- 哈希算法：将任意长度的信息转换为固定长度的值，算法不可逆

- 数字签名：证明某个消息或者文件是某人 发出 / 认同的

4.1.3 HTTPS 数据传输流程

- 浏览器将支持的加密算法信息发送诶服务器

- 服务器选择一套浏览器支持的加密算法，以证书的形式回发浏览器

- 浏览器验证证书合法性，并结合证书公钥加密信息发送给服务器

- 服务器使用私钥解密信息，验证哈希，加密响应消息回发浏览器

- 浏览器解密响应消息，并对消息进行验证，之后进行加密交互数据

![Alt Text](./images/HTTPS数据传输过程.png)

#### 4.2 HTTP 和 HTTPS 的区别

4.2.1 对比 HTTP

- HTTPS 需要到 CA 申请证书，HTTP 不需要

- HTTPS 密文传输，HTTP 明文传输

- 连接方式不同，HTTPS 默认使用 443 端口，HTTP 使用 80 端口

- HTTPS = HTTP + 加密 + 认证 + 完整性保护，较 HTTP 安全

#### 4.3 HTTPS 是否真的很安全

4.3.1 未必

- 浏览器默认填充 http://，请求需要进行跳转，有被劫持的风险

- 可以使用 HSTS（HTTP Strict Transport Security）优化

## 感谢

...