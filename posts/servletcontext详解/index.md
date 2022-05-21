# ServletContext详解

<!--more-->




## 一、ServletContext介绍
ServletContext官方叫servlet上下文。服务器会为每一个工程创建一个对象，这个对象就是ServletContext对象。这个对象全局唯一，而且工程内部的所有servlet都共享这个对象。所以叫全局应用程序共享对象。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201220225727912.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
## 功能介绍
### 1、共享数据

	在Servlet_1中上传数据
```java
package com.tin.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class Servlet_1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //ServletContext 上下文
        ServletContext context = this.getServletContext();
        //数据
        String username="小明";
        //将数据保存在ServletContext中，参数为键值对(名，对象）
        context.setAttribute("username",username);


    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }


}

```
>在Servlet_2中读数据

```java
package com.tin.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class Servlet_2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        //获取数据，类似键值对，需要通过键名获取
        String username=(String) context.getAttribute("username");
        //设置编码格式
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        //输出数据
        resp.getWriter().print(username);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }

}

```
需要在web.xml文件注册servlet和servlet-mapping

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
  <servlet>
    <servlet-name>test</servlet-name>
    <servlet-class>com.tin.servlet.Servlet_1</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>test</servlet-name>
    <url-pattern>/test</url-pattern>
  </servlet-mapping>
  <servlet>
    <servlet-name>test1</servlet-name>
    <servlet-class>com.tin.servlet.Servlet_2</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>test1</servlet-name>
    <url-pattern>/test1</url-pattern>
  </servlet-mapping>
</web-app>
```
测试结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122112394567.png#pic_center)
### 2、获取初始化参数
在web.xml设置参数

```xml
 <context-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql://localhost:3306</param-value>
  </context-param>
```
在java程序中获取参数

```java
package com.tin.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class Servlet_1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //ServletContext 上下文
        ServletContext context = this.getServletContext();
        //获取初始化参数
        String url = context.getInitParameter("url");
         //设置编码格式
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        //输出参数值
        resp.getWriter().print(url);
        
    }
}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221125908392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

### 3、请求转发
在web客户端发出请求，ServletB收到请求后转发给ServletC,将ServletC处理后的响应传给ServletB,ServletB再响应web客户端的请求。web客户端始终是与ServletB进行交互。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222124342747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

将原来servlet_1的请求是/test1，但转发得到请求路径 /test 的结果
```java
package com.tin.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class Servlet_2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        //获取数据，类似键值对，需要通过键名获取
        String username=(String) context.getAttribute("username");
        // ‘/’表示web的根目录 ,修改转发的请求路径
        //forward()实现请求转发
        context.getRequestDispatcher("/test").forward(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }

}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221125821408.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

### 读取资源文件
资源无法导出解决方案：

```xml
<build>
    <resources>
        <resource>
        	<!--添加java文件夹下的xml和properties文件资源-->
            <directory>src/main/java</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```
在resource创建一个资源文件，如test.properties 所以的资源都会放在classes文件夹下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222130153916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
在test.properties 中写入

```yaml
username=root
password=123456
```
在Servlet 中读取资源文件内容

```java
package com.tin.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class Servlet_2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        //用相对资源定位，获取文件流
        InputStream is=context.getResourceAsStream("/WEB-INF/classes/test.properties");
        //加载配置文件
        Properties properties = new Properties();
        properties.load(is);
        //获取配置值
        String username = properties.getProperty("username");
        String password = properties.getProperty("password");
        //设置编码格式
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        resp.getWriter().print(username+":"+password);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }

}

```
运行后，测试：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222190532488.png#pic_center)


