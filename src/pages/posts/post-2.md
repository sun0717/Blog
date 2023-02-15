# Web与HTTP

## 一些术语

- Web页：由一些对象组成

- 对象可以是HTML文件、JPEG图像、Java小程序、声音剪辑文件等

- Web页包含有一个基本的HTML文件，该基本HTML文件又包含若干对象的引用（链接）

- 通过URL对每个对象进行引用

  - 访问协议，用户名，口令名，端口等

- URL格式

  ```bash
  Prot://user:psw@www.someSchool.deu/someDept/pic.gif:port
  协议名  用户 口令    主机名            路径名             端口
  ```



## HTTP：超文本传输协议

- Web的应用层协议

- 客户/服务器模式

  - 客户：请求、接收和显示
  - 服务器：对请求进行响应，发送对象的Web服务器

- 使用TCP：

  - 客户发起一个与服务器的TCP连接（建立套接字），端口号为80
  - 服务器接受客户的TCP连接
  - 在浏览器（HTTP客户端）与Web服务器（HTTP服务器server）交换HTTP报文（应用层协议报文）
  - TCP连接关闭

- HTTP是无状态的

  - 服务器并不维护关于客户的任何信息

- 请求报文格式

  - 两种类型的HTTP报文：请求、响应

  - <strong style="color:red">HTTP请求报文：</strong>

    - ASCII

      首部行：

      ```http
      GET /somedir/page.html HTTP/1.1
      Host: www.someschool.edu
      User-agent: Mozilla/4.0
      Connection: close
      Accept-language: fr
      ```

      (一个额外的换行回车符)  ---> 执行回车符，表示报文结束

- 提交表单输入

  Post方式：
  	1. 网页通常包括表单输入

  ​	2. 包含在实体主题（entity body）中的输入被提交到服务器

​		URL方式：

​			1. 方法：GET

​			2. 输入请求行的URL字段上载

- 方法类型

  - HTTP/1.0

    - GET
    - POST
    - HEAD
      - 要求服务器在相应报文中不包含请求对象->故障跟踪

  - HTTP/1.1

    - GET,POST,HEAD
    - PUT
      - 将实体主体中的文件上载到URL字段规定的路径
    - DELETE
      - 删除URL字段规定的文件

  - HTTP响应报文

    - 首部行

      ```http
      HTTP/1.1 200 OK\r\n
      Connection close\r\n
      Date: Thr, 06 Aug 1998 12:00:15 GMT\r\n
      Server: Apache/1.3.0 (Unix) \r\n
      Last-Modified: Mon, 22 Jun 1998 ... \r\n
      Content-Length: 6821\r\n
      Content-Type: text/html\r\n
      \r\n
      数据：data data data....
      ```

      200 OK

      ​	请求成功，请求对象包含在响应报文的后续部分

      301 Moved Permanently

      ​	请求的对象已经被永久转移了，新的URL在响应报文的Location:首部行中指定

      400 Bad Request

      ​	一个通用的差错代码，表示该请求不能被服务器解读

      404 Not Found

      ​	请求的文档在该服务上没有找到

      505 HTTP Version Not Supported

### Cookie

​	<strong style="color:red">Cookies能带来什么:</strong>

- 用户验证
- 购物车
- 推荐
- 用户状态（Web e-mail）

​	Cookie会携带你太多的隐私。暴露你的网站访问行为，分析你。

### Web缓存（代理服务器）

- 缓存既是客户端又是服务器

- 通常缓存是由ISP安装（大学、公司、居民区ISP）

  <strong style="color:red">为什么使用Web缓存</strong>

  - 降低客户端的请求响应时间
  - 可以大大减少一个机构内部网络与Internet接入链路上的流量
  - 互联网大量采用了缓存：可以使较弱的ICP也能够有效提供内容

​		

​		 