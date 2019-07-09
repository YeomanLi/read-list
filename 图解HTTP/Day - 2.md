|         title         |    date    |      tags      |
| :-------------------: | :--------: | :------------: |
| 图解HTTP - 打卡第二天 | 2019-07-09 | 第三章、第四章 |

# 图解 HTTP

# Chapter - 3

## 报文结构

<!-- more -->

![报文结构 - 1](https://s2.ax1x.com/2019/07/09/ZyRpeH.png)

![报文结构 示例](https://s2.ax1x.com/2019/07/09/ZyRNm4.png)

一般有4种首部：通用首部、请求首部、响应首部、实体首部

### 报文 vs 实体

报文是 HTTP 通信中的基本单位，用于传输请求或响应的实体主体。



## 内容编码

内容编码是指明应用在实体内容上的编码格式，并保持实体信息被原样压缩。内容编码后的实体内容由客户端接收并负责编码。

常用的内容编码有以下几种：

- gzip（GNU zip）
- compress（UNIX 系统的标准压缩）
- deflate（zlib）
- identity（不进行编码）



## 获取部分内容的范围请求（_Range Request_）

使用首部字段 Range 来指定资源的 byte 范围。

指定形式如下：

- 5001~10000字节

  `Range：bytes=5001-10000`

  

- 从5001字节之后全部的

  `Range：bytes=5001-`

  

- 从一开始到3000字节和5000~7000字节的多重范围

  `Range：bytes=-3000, 5000-7000`



针对范围请求，响应会返回状态码为 206 Partial Content 的响应报文。

另外，对于多重范围的范围请求，响应会在首部字段 Content-Type 标明 multipart/byterages 后返回响应报文。

如果服务端无法响应范围请求，则会返回状态码 200 OK 和完整的实体内容。

![举例](https://s2.ax1x.com/2019/07/09/Zyfyee.png)



## 内容协商

客户端和服务器端就响应的资源内容进行交涉，然后提供给客户端最适合的资源。以响应资源的语言、字符集、编码方式等作为判断的基准。

包含在请求报文中的某些首部字段就是判断的基准：

- Accept
- Accept-Charset
- Accept-Encoding
- Accept-Language
- Content-Language



# Chapter - 4

## 返回结果的状态码

![状态码的类别](https://s2.ax1x.com/2019/07/09/Zy4YCR.png)

`2xx` 成功

- 200 OK
- 204 No Content
- 206 Partial Content



`3xx` 重定向

表明浏览器需要执行某些特殊的处理以正确处理请求。

- 301 Moved Permanently

- 302 Moved Temporarily

- 303 See Other 和 302 有着相同的功能，但是 303 状态码明确表示客户端应该采用 GET 方法获取资源。

- 304 Not Modified

  ![304 状态码](https://s2.ax1x.com/2019/07/09/ZyTvlR.png)



`4xx` 客户端错误

- 400 Bad Request 请求报文中存在语法错误，需要修改请求的内容后再次发送请求。
- 401 Unauthorized 认证失败
- 403 Forbidden 禁止访问
- 404 Not Found 除了表明服务器无法找到请求的资源外，也可以在服务端拒绝请求并且不想说明理由时使用。



`5xx` 服务器错误

- 500 Internet Server Error 服务端在执行请求时发生错误。
- 503 Service Unavailable 服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。