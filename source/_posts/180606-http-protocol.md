---
title:      "HTTP 协议"
date:       2018-06-06
author:     "Bowen"
tags:
    - 前端开发
    - 网络请求
---

## HTTP 三次握手

`HTTP` 自身没有和 `server` 端通信传输的功能，`HTTP` 本身只能发起和响应请求，并不传输请求。他是通过创建的 `TCP connection`（作为传输请求的通道）来实现数据传递功能。所有的 `HTTP` 请求创建时，都会创建一个 `TCP` 通道用于数据传输。

  ![http-tcp][http-tcp]

[http-tcp]:https://rawgit.com/lbwa/lbwa.github.io/dev/source/images/post/http-protocol/http-tcp.svg

- `HTTP 1.0` 时，在 `HTTP` 请求创建时，同样会创建一个 `TCP` 通道用于传输数据。在服务端响应请求后，`TCP` 通道就会关闭（非常驻）。

- `HTTP 1.1` 时，可 ***额外声明*** 让服务端响应请求后，`TCP` 仍保持通道开启（常驻状态）。此举用于避免多次请求时，不必要的 `三次握手` 性能开销。

    - 现阶段使用最为广泛的 `HTTP` 协议版本。

- `HTTP 2` 可并发请求，那么在保持 `TCP` 通道开启时，相同用户多次对同一服务器的并发请求可共用一个 `TCP` 通道。

    - `HTTP 2` 正在逐步推广中。 

### 三次握手

在 `HTTP` 通过 `TCP` 执行正式的请求之前，有 3 次预先请求发生在 `client` 和 `server` 端之间。

1. `client` 创建一个预请求以告知 `server`：`client` 即将发起一个正式 `TCP` 连接。此次请求包含标志位（`SYN=1,Seq=X`）。

2. `server` 响应 1 中的预请求，开启相应 `TCP` 端口，并返回一个响应数据包（`SYN=1, ACK=X+1, Seq=Y`）给 `client`。

    - 此次 `server` 返回数据表示 `server` 不仅能够正常接受 `client` 的请求，而且已开启相应端口准备接收即将到来的正式 `TCP` 连接。

    - 此时 `server` 端的 `TCP` 端口将保持开启至响应 `client` 请求（`client` 已正常接收的请求或关闭当前 `TCP` 连接的请求）。

3. `client` 在收到 `server` 端返回的允许创建 `TCP` 连接的请求之后，向 `server` 发送已正常接收到 2 中的响应数据的请求（`ACK=Y+1, Seq=Z`）。

    - 此次请求表示 `client` 能够正常接受 `server` 的响应数据。

此时，完成 `三次握手` 预请求，创建正式的 `TCP` 请求。

### 三次握手的意义

1. 若没有三次握手，直接请求，那么在 `server` 返回数据时，`server` 并不知道 `client` 是否能够正确的接受到请求，是否过程中有数据丢失，那么 `server` 就可能在错误的时机仍然保持 `TCP` 连接端口来等待 `client` 确认数据已接受的请求或关闭当前 `TCP` 连接的请求，这样将带来一系列不必要的 `server` 性能开销。在 `client` 等待时间内没有正确接收请求时，`client` 就会关闭 `TCP` 连接。那么此时 `server` 也就没有必要为为无用的数据连接继续保持开启相应 `TCP` 连接端口。

2. 在有了三次握手的策略后，在正式请求之前，就可以确保当前 `TCP` 通道是可用的，及时发现当前 `TCP` 的网络问题。避免因网络问题导致的无用的数据传输带来的 `server` 端口常驻的性能开销。

## URI/URL/URN

`URI`: Uniform Resource Identifier 统一资源标志符

  - 用于唯一标识互联网中的信息资源

  - 包含 `URL` 和 `URN`

`URL`: Uniform Resource Locator 统一资源定位器

  - 格式如下：

      `protocol://user:pass@host.com:80/path?query=string#hash`

      - `protocol` 协议。如 `https`、`http`、`ftp` 等。

      - `user:pass` 用户验证。因暴露用户账号密码不安全，故不推荐使用。

      - `host` 主机名。

      - `80` 主机端口，默认为 `80`。每个物理主机端口都存放着不同的 web 服务。

      - `path` 路由。
      
          1. `/` 表示当前 `web` 服务的根目录，而不是主机的根目录。
          
          2. `path` 路径默认情况下为 `web` 服务器下数据存放的路径。当数据库独立时，那么 `path` 仅表示数据的 ***存放地址***，并不能表示该数据在服务器磁盘上的路径。

          3. 故推荐在程序内部鉴别数据，而不是通过 URL 鉴别数据。

      - `query=string` 查询参数。常用于向 `server` 端传参。

      - `hash` 哈希值。定位某个资源的某一片段。如文章的锚点。

`URN`: Uniform Resource Name （永久）统一资源定位符

  - 用于永久性在网络中标识出资源，因限制过多，已逐渐被 `URI` 取代。（[extension][urn]）

[urn]:https://en.wikipedia.org/wiki/Uniform_Resource_Name

## HTTP 报文

`HTTP` 报文没有强约束，可自定义报文内容。

![http-bw][http-bw]

[http-bw]:https://rawgit.com/lbwa/lbwa.github.io/dev/source/images/post/http-protocol/http-bw.svg

## HTTP 方法

- 用来定义对于资源的操作

    - 常用方法有 `GET`、`POST`、`PUT`、`DELETE`。另外还有 `HEAD`、`OPTIONS`、`PATCH` 方法。

    - 应该从开发人员的使用方式来定义各自方法的语义。

## HTTP code

- 定义服务器对请求的处理结果。

    - 2XX - Success - 表示成功处理请求。如 200。

    - 3XX - Redirection - 需要重定向，浏览器直接跳转。

    - 4XX - Client Error - 客户端请求错误。

    - 5XX - Server Error - 服务端响应错误。

- 推荐 `server` 端正确配置 HTTP code，使得 HTTP code 语义化。好的 `HTTP` 服务应该可以通过 HTTP code 来判断请求结果。而不是只有 `200` 或 `500`。

## HTTP 客户端

能够发起 HTTP 请求，并能够接收返回数据的客户端都可称为 HTTP 客户端。如 `curl`、`XMLHttpRequest`、浏览器等。

除了在浏览器中可以观察 HTTP 请求的细节外，亦可使用 `curl` 命令行工具来观察。 

```bash
# -v 表示显示报文信息
curl -v www.baidu.com
```

返回数据如下：

```bash
* Rebuilt URL to: www.google.com/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*
  Trying 172.217.10.132...
* TCP_NODELAY set
* Connected to www.google.com (172.217.10.132) port 80 (#0)
# 请求报文
  # 起始行
> GET / HTTP/1.1
  # 首部
> Host: www.google.com
> User-Agent: curl/7.57.0
> Accept: */*
> # 此处有一空行
# 响应报文
  # 起始行
< HTTP/1.1 200 OK
  # 首部
< Date: Thu, 07 Jun 2018 14:28:45 GMT
< Expires: -1
< Cache-Control: private, max-age=0
< Content-Type: text/html; charset=ISO-8859-1
# 省略一些信息
# ...
< Transfer-Encoding: chunked
<
{ [759 bytes data]
100  3555    0  3555    0     0   3555      0 --:--:--  0:00:01 --:--:--  1834
# 以下是响应报文的主体内容区域
# ...
<!doctype html><html
```

## HTTP 响应首部

HTTP 响应首部即 `Response Headers`。一般用于在 `server` 端配置合法的 `CORS` 请求信息。

### Access-Control-Allow-Origin

- 常用于 HTTP 请求跨域解决方案之一 —— `CORS` 。表示指定了该响应资源只允许被给定的 `Origin` 共享。该值设置为 `*` 时，表示允许所有源都具有访问该资源的权限（[source][access-control-allow-origin]）。

- 该属性只能指定一个 ***唯一值***，不接受多个值。

    - 若有多个源需要通过 CORS 跨域，那么可配置一个模块。该模块在 `server` 端设置该头部前配置筛选出 URL 是否为白名单内源，若是白名单内源，那么就配置头部 `Access-Control-Allow-Origin`，否则不配置该头部。

详见我的另一篇博文👉[客户端跨域解决方案][客户端跨域解决方案]

[access-control-allow-origin]:https://fetch.spec.whatwg.org/#http-access-control-allow-origin

[客户端跨域解决方案]:http://lbwa.github.io/2018/04/19/180419-Cross-domain-solution/

### Access-Control-Allow-Headers

- 常用于标记超出 `CORS` 限定配置情况下的 `request headers` 是否合法。表示指定在 `CORS` 请求中除限定配置外额外被允许的请求头（[source][access-control-allow-headers]）。

1. CORS 请求限制

    - 默认允许的 `CORS` 请求方法（[source][CORS-methods]）

    只允许 `GET`、`POST`、`HEAD` 方法。使用其他请求方法都需要经过 `CORS` 预请求。

    - 默认允许的 `CORS` 请求头（[source][cors-safelisted-request-header]）

        - `Accept`
        - `Accept-Language`
        - `Content-Language`
        - `Content-Type` 中仅包含 `text/plain`、`multipart/form-data`、`application/x-www-form-urlencoded` 三种 `MIME` 类型值。

    - 其他限制

        1. `XMLHttpRequestUpload` 对象均没有注册任何事件监听程序。

        2. 请求中没有使用 `ReadableStream` 对象。

***总结***:使用其他超出以上 `CORS` 请求所限定的配置都将需要经过 `CORS` 预请求检测 `CORS` 请求头的合法性。

2. CORS 预请求

`CORS` 预请求的 `Request Method` 值为 `OPTIONS`。

在浏览器即将发起超过 1 中限定配置的 `CORS` 请求时，将触发浏览器 `CORS` 预请求策略。该策略用于在发起正式的 `CORS` 请求之前确认 `CORS` 请求中超出限定配置的部分是否合法。仅当超出默认配置的默认配置被 `server` 端认可时，浏览器才会真正 ***解析*** CORS 正式请求返回的数据。

  - 不论 `CORS` 预请求是否合法，浏览器均会发出正式的 `CORS` 请求，合法性检测的意义在于浏览器 ***是否解析*** 返回的数据。

  ```js
  // server1.js
  const http = require('http')
  const fs = require('fs')

  http.createServer(function (request, response) {
    console.log('request.url :', request.url)

    const html = fs.readFileSync('cross-domain-solution.html', 'utf8')
    response.writeHead(200, {
      'Content-type': 'text/html',
    })
    response.end(html)
  }).listen(8888)

  console.info('server listening at port 8888')

  // client 跨域请求 server2 数据
  fetch('http://127.0.0.1:8800', {
    method: 'POST',
    headers: {
      // 请求头类型不在 CORS 请求限定配置内，触发 CORS 预请求检测该请求头合法性
      'X-Test-Cors': 'test custom headers in CORS preflight'
    }
  })
    .then(res => {
      target.innerText = 'check your network tag in console drawer'
    })
    .catch(err => console.error(err))
  // 不论 CORS 预请求是否合法，client 均会发起 CORS 正式请求。
  ```

  当被请求的 `server2` 没有配置 `Access-Control-Allow-Headers` 或目标值不在该值中时，`client` 将在预请求响应后报错，但仍发起正式 `CORS` 请求，但拒绝解析正式 `CORS` 请求返回的数据。

  ```js
  // server2.js
  const http = require('http')

  http.createServer(function (request, response) {
    console.log('request.url :', request.url)

    response.writeHead(200, {
      // 允许跨域请求
      'Access-Control-Allow-Origin': '*',
      // 允许除限定配置外额外的合法请求头的值
      'Access-Control-Allow-Headers': 'X-Test-Cors'
    })
    response.end('server response')
  }).listen(8800)

  console.log('server listening at port 8800')
  ```

[CORS-methods]:https://fetch.spec.whatwg.org/#methods

[cors-safelisted-request-header]:https://fetch.spec.whatwg.org/#cors-safelisted-request-header

[access-control-allow-headers]:https://fetch.spec.whatwg.org/#http-access-control-allow-headers

### Access-Control-Allow-Methods

该响应头的使用方法与原理于 `Access-Control-Allow-Headers` 相似。

- 常用于标记超出 `CORS` 限定配置情况下的 `Request Method` 是否合法（[source][access-control-allow-methods]）。

  ```js
  response.writeHead(200, {
    // 允许跨域请求
    'Access-Control-Allow-Origin': '*',
    // 允许除限定配置外额外的合法 `Request Method` 的值
    'Access-Control-Allow-Methods': 'PUT, DELETE'
  })
  ```

[access-control-allow-methods]:https://fetch.spec.whatwg.org/#http-access-control-allow-methods

### Access-Control-Max-Age

- 表示当次预请求检测 `Access-Control-Allow-Methods` 和 `Access-Control-Allow-Headers` 的缓存有效期，即在有效期内，即使有超出限定配置的 `CORS` 请求也不需要再进行 `CORS` 预请求来检测其合法性（[source][access-control-max-age]）。

[access-control-max-age]:https://fetch.spec.whatwg.org/#http-access-control-max-age

## HTTP 请求首部

### Cache-Control

- 用于指定在 `request` 或 `response` 链中缓存当前请求数据，该指令是单向指令（[source][http1.1-cache-control]）。

- 可缓存性

    1. `public` 表示响应链中所有缓存都可存储当前响应数据，如发送客户端，中转服务器等。

    2. `private` 表示当前响应数据只能单个用户缓存，即中转服务器不能缓存该响应数据。

    3. `no-cache` 表示在使用缓存之前，必须先请求原 `server` 端验证当前缓存的数据是否可用。

    ![cache-control][img-cache-control]

    [img-cache-control]:https://rawgit.com/lbwa/lbwa.github.io/dev/source/images/post/http-protocol/cache-control.svg

- 缓存有效期

    1. `max-age=<seconds>` 于 `server` 端设置响应数据在 `client` 端的缓存有效期，始于请求时间点。在有效期内，`client` 将读取缓存数据而不是请求数据。即使在 `server` 端该数据已经被更新，也不会改变 `client` 在有效期内读取缓存的策略，因为 `client` 在有效期内当前请求 URL 未改变的情况下就不会去请求该数据，所以 `client` 并不知道该数据已经在 `server` 端被更新了。

        ```js
        response.writeHead(200, {
          'Content-type': 'text/javascript',
          'Cache-Control': 'max-age=200, public' // 以秒为单位
        })
        response.end('console.log("script loaded")')
        ```

        - ***拓展应用***：根据静态资源的 ***内容*** 打包生成的 `contentHash` 码来命名常缓存文件。只要 `server` 端该静态资源文件被更新，那么该资源的 `contentHash` 一定变化，即请求 URL 改变，那么 `client` 知晓当前静态资源请求 URL 改变后，即使在缓存有效期内，也会重新请求该资源。这样做的目的是最大限度使用缓存文件，且规避在有效期内即使 `server` 端数据被更新但仍使用缓存文件的问题。

    2. `s-maxage=<seconds>` 覆盖 `max-age=<seconds>`，只在共享缓存中（如中转服务器）有效。

    3. `max-stale[=<seconds>]` 表示即使缓存过期，仍可接受一个（在指定时间内）已过期资源，只在发起端设置才有效，在 `server` 端响应数据中设置是无效的。

- 验证

    1. `must-revalidate` 在使用之前的旧资源时，必须请求原 `server` 端来验证当前旧资源是否已经过期。

    2. `proxy-revalidate` 与 `must-revalidate` 作用相同，但仅适用于共享缓存，如中转服务器。

- 其他

    1. `no-store` 表示所有的链中节点的缓存都不可存储当前响应数据。

    2. `no-transform` 表示不能对当前响应数据进行转换或变化。

***注***：以上所有指令都不具有强制力，仅表示一种约束期望。

[http1.1-cache-control]:https://tools.ietf.org/html/rfc7234#section-5.2