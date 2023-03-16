---
title: 从输入URL到页面展示发生了什么
date: 2023-03-11 21:56:01
tags:
    - 前端面试
categories: 前端面试
# no_toc: true

---

在浏览器地址栏输入网址，如：[www.baidu.com](https://www.baidu.com)后浏览器是怎么把最终的页面呈现出来的呢？
<!--more-->

1. 浏览器输入URL地址按下回车
2. 浏览器查找当前URL是否存在缓存，并比较缓存是否过期
3. DNS解析URL对应的IP
4. 根据IP建立TCP链接（三次握手）
5. HTTP发起请求
6. 服务器处理请求，浏览器接收HTTP响应
7. 渲染页面，构建DOM树
8. 关闭TCP链接（四次握手）

### 一、URL

URL（Uniform Resource Locator），统一资源定位符，用于定位互联网上资源，俗称网址。URL中的所有字符都是*ASCII*字符集，如果出现非*ASCII*字符，比如中文，浏览器会进行编码再进行传输

遵守以下的语法规则：

```js
scheme://host:port/path/filename
https://www.baidu.com:443/img?name=123#anchor
```

- scheme（协议）：因特网服务的类型。常见的协议有 http、https、ftp、file。http最常见，https 是进行加密的网络传输。

- host（主机名，域名，IP）

- port（端口号）：整数，可选范围0-65535，http默认为80，https默认为443。

- path（查找路径）：用来表示主机上的一个目录或文件地址。

- query（查询参数）：用于给[动态网页](https://baike.baidu.com/item/动态网页?fromModule=lemma_inlink)（如使用CGI、ISAPI、PHP/JSP/ASP/ASP.NET等技术制作的网页）传递参数，可有多个参数，用“&”符号隔开，每个参数的名和值用“=”符号隔开。

- fragment（#锚点）：在 URI 的末尾通过 hash mark（#）作为 fragment 的开头，其中 # 不属于 fragment 的值。

  ```jsp
  https://www.baidu.com/index#wang18
  1.# 有别于 ?，? 后面的查询字符串会被网络请求带上服务器，而 fragment 不会被发送的服务器
  2.fragment 的改变不会触发浏览器刷新页面，但是会生成浏览历史
  3.fragment 会被浏览器根据文件媒体类型（MIME type）进行对应的处理
  4.Google 的搜索引擎会忽略 # 及其后面的字符串。
  
  ```

  ```js
  针对1，2条特性，用于单页面应用跳转页面，修改 location.hash 值，触发 hashchange 事件，JS 处理对应的逻辑，改变页面 UI 实现页面的跳转，并在浏览器中产生历史记录。
  HTML锚点，页面中寻找wang18这个锚点，并将页面滚动到该锚点的位置， URI 中是空的 fragment，则浏览器默认显示页面的最顶端。
  fragment 会被 Google 搜索引擎忽略掉，不利于SEO。Google 给了一个方案，就是在 # 紧跟一个 ! ，这样Google 搜索引擎就会将这个 URI 进行转换，如https://www.baidu.com/index#wang18转换后就成为了 https://www.baidu.com/index.html?_escaped_fragment_=wang18
  ```

### 二、缓存

- 浏览器每次发起请求，都会先在浏览器缓存中查找该请求的结果以及缓存标识
- 浏览器每次拿到返回的请求结果都会将该结果和缓存标识存入浏览器缓存中

![](https://pic.imgdb.cn/item/64108a80ebf10e5d53a39216.png)

> 强制缓存

强制缓存就是向浏览器缓存查找该请求结果，并根据该结果的缓存规则来决定是否使用该缓存结果的过程

* 不存在该缓存结果和缓存标识，强制缓存失效，则直接向服务器发起请求
* 存在该缓存结果和缓存标识，但该结果已失效，强制缓存失效，则使用协商缓存(暂不分析)
* 存在该缓存结果和缓存标识，且该结果尚未失效，强制缓存生效，直接返回该结果

> 强制缓存的缓存规则是什么？

当浏览器向服务器发起请求时，服务器会将缓存规则放入HTTP响应报文的HTTP头中和请求结果一起返回给浏览器，控制强制缓存的字段分别是Expires和Cache-Control，其中Cache-Control优先级比Expires高。

Expires是HTTP/1.0控制网页缓存的字段，其值为服务器返回该请求结果缓存的到期时间，即再次发起该请求时，如果客户端的时间小于Expires的值时，直接使用缓存结果。

在HTTP/1.1中，Cache-Control是最重要的规则，主要用于控制网页缓存，主要取值为：

- public：所有内容都将被缓存（客户端和代理服务器都可缓存）
- private：所有内容只有客户端可以缓存，Cache-Control的默认取值
- no-cache：客户端缓存内容，但是是否使用缓存则需要经过协商缓存来验证决定
- no-store：所有内容都不会被缓存，即不使用强制缓存，也不使用协商缓存
- max-age=xxx (xxx is numeric)：缓存内容将在xxx秒后失效

> 内存缓存和硬盘缓存

内存缓存(from memory cache)和硬盘缓存(from disk cache)，如下:

- 内存缓存(from memory cache)：内存缓存具有两个特点，分别是快速读取和时效性：
- 快速读取：内存缓存会将编译解析后的文件，直接存入该进程的内存中，占据该进程一定的内存资源，以方便下次运行使用时的快速读取。
- 时效性：一旦该进程关闭，则该进程的内存则会清空。
- 硬盘缓存(from disk cache)：硬盘缓存则是直接将缓存写入硬盘文件中，读取缓存需要对该缓存存放的硬盘文件进行I/O操作，然后重新解析该缓存内容，读取复杂，速度比内存缓存慢。

在浏览器中，浏览器会在js和图片等文件解析执行后直接存入内存缓存中，那么当刷新页面时只需直接从内存缓存中读取(from memory cache)；而css文件则会存入硬盘文件中，所以每次渲染页面都需要从硬盘读取缓存(from disk cache)。

> 协商缓存

协商缓存就是强制缓存失效后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否使用缓存的过程，主要有以下两种情况：

- 协商缓存生效，返回304

- 协商缓存失效，返回200和请求结果结果

> 总结

强制缓存优先于协商缓存进行，若强制缓存(Expires和Cache-Control)生效则直接使用缓存，若不生效则进行协商缓存(Last-Modified / If-Modified-Since和Etag / If-None-Match)，协商缓存由服务器决定是否使用缓存，若协商缓存失效，那么代表该请求的缓存失效，重新获取请求结果，再存入浏览器缓存中；生效则返回304，继续使用缓存。

### 三、DNS解析

> 为什么需要DNS系统

当客户端访问[www.baidu.com](www.baidu.com)时计算机是识别不了的，需要转换成IP地址，计算机找到IP就能访问到web，就可以访问到对应的百度服务器。win+R输入cmd，输入ping[www.baidu.com](www.baidu.com)可以看到对应的IP地址为112.80.248.76。

更多内容可以查看[DNS域名解析CSDN博客](https://blog.csdn.net/L2111533547/article/details/124180225)

### 四、TCP链接

传输控制协议（TCP，Transmission Control Protocol）是为了在不可靠的[互联网络](https://baike.baidu.com/item/互联网络/6908194?fromModule=lemma_inlink)上提供可靠的[端到端](https://baike.baidu.com/item/端到端/8851783?fromModule=lemma_inlink)字节流而专门设计的一个[传输协议](https://baike.baidu.com/item/传输协议/8048821?fromModule=lemma_inlink)。

![](https://pic.imgdb.cn/item/6411dc39ebf10e5d53ee7408.png)

### 五、发送HTTP请求

指从[客户端](https://baike.baidu.com/item/客户端?fromModule=lemma_inlink)到[服务器端](https://baike.baidu.com/item/服务器端/3369401?fromModule=lemma_inlink)的请求消息。HTTPS由两部分组成HTTP+SSL/TLS，在http上加了一层处理加密信息的模块。服务端和客户端的信息传输都会通过TLS加密，传输的数据自然也是加密后的数据。

HTTP协议：[超文本传输协议](https://baike.baidu.com/item/超文本传输协议)（HTTP，HyperText Transfer Protocol)是[互联网](https://baike.baidu.com/item/互联网)上应用最为广泛的一种[网络协议](https://baike.baidu.com/item/网络协议)。

HTTPS=HTTP+加密+认证+完整性保护

发送http请求会对TCP连接进行处理，对HTTP协议进行解析，并按照报文格式进一步封装成HTTP Request对象。

Web服务器有Tomcat, Nginx和Apach

`http请求报文：请求行，请求头，空行，请求数据`

> 请求行

请求行由请求方法、URL和HTTP协议版本3个字段组成，它们用空格分隔。比如 GET /data/info.html HTTP/1.1

方法字段就是HTTP使用的请求方法，比如常见的GET/POST

其中HTTP协议版本有两种：HTTP1.0/HTTP1.1 可以这样区别：

HTTP1.0对于每个连接都只能传送一个请求和响应，请求就会关闭，HTTP1.0没有Host字段；而HTTP1.1在同一个连接中可以传送多个请求和响应，多个请求可以重叠和同时进行，HTTP1.1必须有Host字段。

> 请求头

```
HTTP Request Header 请求头
Accept：指定客户端能够接收的内容类型。
Accept-Charset：浏览器可以接受的字符编码集。
Accept-Encoding：指定浏览器可以支持的web服务器返回内容压缩编码类型。
Accept-Language：浏览器可接受的语言。
Accept-Ranges：可以请求网页实体的一个或者多个子范围字段。
AuthorizationHTTP：授权的授权证书。
Cache-Control：指定请求和响应遵循的缓存机制。
Connection：表示是否需要持久连接。（HTTP 1.1默认进行持久连接）
CookieHTTP：请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。
Content-Length：请求的内容长度。
Content-Type：请求的与实体对应的MIME信息。
Date：请求发送的日期和时间。
Expect：请求的特定的服务器行为。
From：发出请求的用户的Email。
Host：指定请求的服务器的域名和端口号。
If-Match：只有请求内容与实体相匹配才有效。
If-Modified-Since：如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码。
If-None-Match：如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变。
If-Range：如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。
If-Unmodified-Since：只在实体在指定时间之后未被修改才请求成功。
Max-Forwards：限制信息通过代理和网关传送的时间。
Pragma：用来包含实现特定的指令。
Proxy-Authorization：连接到代理的授权证书。
Range：只请求实体的一部分，指定范围。
Referer：先前网页的地址，当前请求网页紧随其后,即来路。
TE：客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息。
Upgrade：向服务器指定某种传输协议以便服务器进行转换（如果支持。
User-AgentUser-Agent：的内容包含发出请求的用户信息。
Via：通知中间网关或代理服务器地址，通信协议。
Warning：关于消息实体的警告信息
```

> 空行

作用是通过一个空行，告诉服务器请求头部到此为止。

> 请求数据

若方法字段是GET，参数直接置于请求行URL中，报文体则为空

若方法字段是POST，则通常来说此处放置的就是要提交的数据

> 请求方式

| 请求方式 |                             描述                             |
| :------: | :----------------------------------------------------------: |
|   GET    |              请求指定的页面信息，并返回实体主体              |
|   POST   | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改 |
|   PUT    |        从客户端向服务器传送的数据取代指定的文档的内容        |
|  DELETE  |                   请求服务器删除指定的页面                   |
| OPTIONS  |                  允许客户端查看服务器的性能                  |
|   HEAD   | 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头 |
| CONNECT  |    HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器    |
|  TRACE   |           回显服务器收到的请求，主要用于测试或诊断           |

### 六、浏览器接收响应

`http响应报文：状态码、响应头、响应报文`

> 状态码

常见状态码：200, 204, 301, 302, 304, 400, 401, 403, 404, 422, 500

```
1xx：指示信息–表示请求已接收，继续处理。
2xx：成功–表示请求已被成功接收、理解、接受。
3xx：重定向–要完成请求必须进行更进一步的操作。
4xx：客户端错误–请求有语法错误或请求无法实现。
5xx：服务器端错误–服务器未能实现合法的请求
```

> 响应头

```
HTTP Responses Header 响应头
Accept-Ranges：表明服务器是否支持指定范围请求及哪种类型的分段请求。
Age：从原始服务器到代理缓存形成的估算时间（以秒计，非负）。
Allow：对某网络资源的有效的请求行为，不允许则返回405。
Cache-Control：告诉所有的缓存机制是否可以缓存及哪种类型。
Content-Encodingweb：服务器支持的返回内容压缩编码类型。
Content-Language：响应体的语言。
Content-Length：响应体的长度。
Content-Location：请求资源可替代的备用的另一地址。
Content-MD5：返回资源的MD5校验值。
Content-Range：在整个返回体中本部分的字节位置。
Content-Type：返回内容的MIME类型。
Date：原始服务器消息发出的时间。
ETag：请求变量的实体标签的当前值。
Expires：响应过期的日期和时间。
Last-Modified：请求资源的最后修改时间。
Location：用来重定向接收方到非请求URL的位置来完成请求或标识新的资源。
Pragma：包括实现特定的指令，它可应用到响应链上的任何接收方。
Proxy-Authenticate：它指出认证方案和可应用到代理的该URL上的参数。
refresh：应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持）
Retry-After：如果实体暂时不可取，通知客户端在指定时间之后再次尝试。
Serverweb：服务器软件名称。
Set-Cookie：设置Http Cookie。
Trailer：指出头域在分块传输编码的尾部存在。
Transfer-Encoding：文件传输编码。
Vary：告诉下游代理是使用缓存响应还是从原始服务器请求。
Via：告知代理客户端响应是通过哪里发送的。
Warning：警告实体可能存在的问题。
WWW-Authenticate：表明客户端请求实体应该使用的授权方案。
```

> 响应报文

响应报文就是响应的消息体，如果是纯数据就是返回纯数据，如果请求的是HTML页面，那么返回的就是HTML代码，如果是JS就是JS代码

### 七、浏览器解析渲染页面

浏览器解析渲染页面是一个从上到下的过程。
 1.解析HTML标签，构建DOM树 。
 2.解析CSS，构建CSSOM树。
 3.把DOM和CSSOM组合成渲染树（render tree）。
 4.在渲染树的基础上进行布局，计算每个节点的几何结构。
 5.把每个节点绘制到屏幕上（painting）。

在构建页面的过程中，不可避免地会遇到浏览器要重新计算每个元素的大小位置样式。这里会遇到reflow和repaint。
 **Reflow**（回流）Relayout（重新布局）
 重新计算元素的几何尺寸，位置。
 假设通过JS或是别的什么方法把一些元素消除了，浏览器又要重新计算每个元素的位置，渲染树，几何结构。

**Repaint**（重绘）
 重新绘制界面发生变化的部分。
 通过JS修改元素的颜色，其他元素的几何结构位置没变。
 只是要改变样式颜色。重绘。

要尽量减少这两个东西的发生。尽量一次性修改样式，给动画使用绝对定位可以减少reflow次数，DOM离线后修改。都是可行的办法。

由于会发生 阻塞解析和阻塞渲染，还是尽量让CSSlink样式表存在于head标签内，JS的加载使用无顺序的async异步加载或者有顺序的defer加载方式。
