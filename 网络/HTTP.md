# HTTP

<!-- TOP -->
- [一、URI和URL](#一URI和URL)
  - [统一资源标识符](#统一资源标识符)
  - [URI格式](#URI格式)
- [二、简单的HTTP协议](#二简单的HTTP协议)
- [三、HTTP方法](#三HTTP方法)
  - [GET](#GET获取资源)
  - [POST](#POST传输实体主体)
  - [PUT](#PUT传输文件)
  - [HEAD](#HEAD获得报文首部)
  - [DELETE](#DELETE删除文件)
  - [OPTIONS](#OPTIONS询问支持的方法)
  - [TRACE](#TRACE追踪路径)
  - [CONNECT](#CONNECT要求用隧道协议连接代理)
- [参考资料](#参考资料)
<!-- TOP -->

## 一、URI和URL

与URI相比，我们更熟悉URL(Uniform Resource Locator，统一资源定位符)。
URL就是Web浏览器等访问Web页面时需要输入的网页地址(https://github.com)。

### 统一资源标识符

URI是 Uniform Resource Identifier 的缩写。
==Uniform==
规定统一的格式，可以方便处理多种不同类型的资源，而不用根据上下文环境来识别资源指定的访问方式。
==Resource==
资源的定义是 “可标识的任何东西”。
==Identifier==
表示可标识的对象，也称为标识符。

URI用字符串标示某一互联网资源，而URL表示资源的地点(互联网上所处的位置)。可见URL是URI的子集。

### URI格式

表示指定的URI，要使用涵盖全部必要信息的绝对URI、绝对URL以及相对URL。

## 二、简单的HTTP协议

- HTTP协议用于客户端和服务器端之间的通信
  - 请求访问资源的一端称为客户端
  - 提供资源响应的一端称为服务器端

- 通过请求和响应的交换达成通信
  - 请求从客户端发出，最后服务器端响应该请求并返回
  - 服务器端没有接收到请求之前不会发送响应

- HTTP是不保存的协议状态
  - HTTP是一种不保存状态，即无状态(stateless)协议
  - HTTP协议自身不对请求和响应之间的通信状态进行保存
  - 这是为了更快地处理大量事务，确保协议的可伸缩性
  - 使用Cookie技术来保存管理状态
zhuizonglujing
## 三、HTTP方法

### GET：获取资源

***GET方法用于请求访问已被URL识别的资源***

请求：

```shell
GET /index.html HTTP/1.1
Host: example.com
```

### POST：传输实体主体

***POST 主要传输数据，GET 主要获取数据***

请求：

```shell
POST /submit.cgi HTTP/1.1
Content-Length: 1560
```

### PUT：传输文件

***自身不带验证机制，任何人都可以上传文件，存在安全性的问题，因此一般不用该方法***

```shell
PUT /example.html HTTP/1.1
Host: example.com
Content-Type: text/html
Content-Length: 1560
```

### HEAD：获得报文首部

***和GET方法一样，但是不返回报文主体部分***
***用于确认URI的有效性及资源更新的日期时间等***

```shell
HEAD /index.html HTTP/1.1
Host: example.com
```

### DELETE：删除文件

***DELETE是用于删除文件，和PUT相反***

```shell
DELETE /example.html HTTP/1.1
Host: example.com
```

### OPTIONS：询问支持的方法

***查询针对请求URI指定的资源的支持的方法***
会返回`Allow: GET, POST, HEAD, OPTIONS`(服务器支持的方法)

```shell
OPTIONS * HTTP/1.1
Host: example.com
```

### TRACE：追踪路径

***服务器会将通信路径返回给客户端***

> 当发送请求时，在`Max-Forwards`首部字段中填入数值，每经过一个服务器端就将这个数字减1，当数值刚好为0时，就停止继续传输，最后接收到请求的服务器端则返回状态码`200 OK`的响应。
> 但是，通常不会使用`TRACE`方法，它容易受到`XST(Cross-Site Tracing, 跨站追踪)`攻击。

### CONNECT：要求用隧道协议连接代理

要求在与代理服务器通信时建立隧道，实现用隧道协议进行TCP通信
主要使用`SSL(Secure Sockets Layer，安全套接层)`和`TSL(Transport Layer Security，传输层安全)` 协议把通信内容加密后经过网络隧道传输。

```shell
CONNECT www.example.com:433 HTTP/1.1
```

## 参考资料

- 图解HTTP· 上野 宣 著，于均良 译