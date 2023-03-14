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

### 四、TCP链接

### 五、发送HTTP请求

### 六、浏览器接收响应

### 七、页面渲染





