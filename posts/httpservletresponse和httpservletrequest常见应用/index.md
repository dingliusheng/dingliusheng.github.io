# HttpServletResponse和HttpServletRequest常见应用

<!--more-->


### HttpServletResponse和HttpServletRequest
web服务器接收到客户端请求，针对这个请求，分别创建一个代表请求的HttpServletRequest，代表响应的HttpServletResponse

 - 如果要获取客户端请求过来的参数，就用HttpServletRequest对象
 - 如果要给客户端响应一些信息，就用HttpServletResponse对象
### HttpServletResponse的接口
1、向浏览器发送数据的方法

```java
 ServletOutputStream getOutputStream() throws IOException;

 PrintWriter getWriter() throws IOException;
```
2、负责向浏览器发送响应头的方法

```java

    void setCharacterEncoding(String var1);
    void setContentType(String var1);
    void setHeader(String var1, String var2);
    void addHeader(String var1, String var2);
```
3、响应状态码：

```java
//响应成功 200
int SC_OK = 200;
//请求重定向  3XX
int SC_MULTIPLE_CHOICES = 300;
//找不到资源 404
 int SC_NOT_FOUND = 404;
//服务器代码错误 500
int SC_INTERNAL_SERVER_ERROR = 500;
//网关错误
 int SC_BAD_GATEWAY = 502;
```
### HttpServletResponse的常见应用
1、向浏览器输出消息

```java
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        resp.getWriter().print("你好，小明");

    }
```
2、下载文件
```java
package com.tin.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.FileInputStream;
import java.io.IOException;
import java.net.URLEncoder;

public class Servlet_1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       //获取下载文件的路径
        String realPath = "E:\\javaWebTest\\Servlet\\src\\main\\resources\\风景.png";
        //System.out.print(realPath);
        //获取下载文件名
        String filename = realPath.substring(realPath.lastIndexOf("\\") + 1);
        //设置响应的消息头
        resp.setContentType("text/html;charset=UTF-8");
        //设置响应类型中包含文件附件
        resp.setHeader("Content-Disposition", "attachment; " +
                "filename="+URLEncoder.encode(filename,"UTF-8"));
        //设置文件流
        FileInputStream fis=new FileInputStream(realPath);
        //创建缓存区
        int len;
        byte[] buff=new byte[1024];
        //获取outputStream对象
        ServletOutputStream outputStream = resp.getOutputStream();
        //IO流,下载文件
        while((len=fis.read())!=-1){
            outputStream.write(buff,0,len);
        }
        outputStream.close();
        fis.close();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}

```

 3、重定向
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223124404432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       // '/'表示根目录，如localhost：8080,需要加入具体的项目名和请求
       resp.sendRedirect("/Servlet_war/test");
       
    }
```
等同于

```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置路径和状态码
        // '/'表示根目录，如localhost：8080,需要加入具体的项目名和请求
        resp.setHeader("location","/Servlet_war/test");
        resp.setStatus(302);

    }
```
重定向和请求转发的区别：

 **相同点：**
 		1、 都实现页面跳转
 		
**不同点：**
 		1、请求转发，url地址栏不会改变
 		2、重定向，url地址栏会发生改变
### HttpServletRequest常见应用
**获取参数**
login.jsp 模拟用户登入页面
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>login</title>
</head>
<body>
    <div style="text-align: center;">
    <%--${pageContext.request.contextPath} 表示webapp运行的路径，如：http://localhost:8080/Servlet_war --%>
        <form action="${pageContext.request.contextPath}/login" method="post">
            用户名：<input type="text" name="username"><br>
            密码：<input type="password" name="password"><br>
            爱好：
            <input type="checkbox" name="hobby" value="运动">运动
            <input type="checkbox" name="hobby" value="美食">美食
            <input type="checkbox" name="hobby" value="电影">电影
            <input type="checkbox" name="hobby" value="电子游戏">电子游戏
            <br>
            <input type="submit">
        </form>
    </div>

</body>
</html>

```
LoginServlet 对login.jsp提交的信息进行处理，然后跳转到index.jsp

```java
package com.tin.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Array;
import java.util.Arrays;

public class LoginServlet  extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        //获取单个参数
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        //获取参数数组
        String[] hobbies = req.getParameterValues("hobby");
        //在这里可以进行用户登入判断，用户是否存在，密码是否正确
        //输出参数信息
        System.out.println(username+":"+password+ Arrays.toString(hobbies));
        //设置客户端浏览器以UTF-8编码解析数据
        //resp.setCharacterEncoding("UTF-8");
        //resp.setContentType("text/html;charset=UTF-8");
        //resp.getWriter().print(username+":"+password+ Arrays.toString(hobbies));
        //重定向到index.jsp
        resp.sendRedirect("/Servlet_war/index.jsp");

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}

```


