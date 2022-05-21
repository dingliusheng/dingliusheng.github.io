# http的简单介绍


<!--more-->


## http
## 什么是HTTP
> HTTP：超文本传输协议

是一个简单的==请求-响应==协议，通常运行在==tcp==上。

 - 超文本：图片、音频、视频、定位、地图。。。
 - 端口号：80
>HTTPS：传输层加密协议
 - 和HTTPS区别不大，https更安全
 - 端口号：443
## HTTP发展
 - http1.0
 		客户端与web服务器连接后，只能获得一个web资源
 - http1.1
		客户端与web服务器连接后，可以获得多个web资源
## http请求
 - 客户端 -->发送请求 -->服务器
 
百度为例：（在百度网站下打开开发者工具 Network ）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201215230720776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
## Http基本信息
状态码：
- 响应成功 200
- 请求重定向  3XX
- 找不到资源 404
- 服务器代码错误 500
- 网关错误502

百度为例：（在百度网站下打开开发者工具 Network ）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223181137529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

## http响应头部信息
 - 服务器-->响应-->客户端
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201215231724659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
http请求头部信息：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223181830585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

==这些信息都可以通过HttpServletResponse 和	HttpServletRequest对象进行修改==（博客的javaWeb文章会涉及）

