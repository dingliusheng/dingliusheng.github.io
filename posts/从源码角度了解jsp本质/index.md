# 从源码角度了解jsp本质


<!--more-->

### 什么是JSP
java server pages:java服务端页面，和servlet一样，用于动态web技术。
最大的特点：
	==完全兼容Html和嵌入java代码==
### JSP原理
jsp在服务器内部运行（以tomcat为例）：
index.jsp

```html
<html>
<body>
<h2>Hello World!</h2>
</body>
</html>
```
在idea 运行javaweb程序，会在Output输出中找到==jsp==转为==.java==的文件路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201231204400241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

找到javaweb在idea运行生成的项目文件后，在下面这个路径可以找到javaweb的jsp转java文件
==\work\Catalina\localhost\Servlet_war\org\apache\jsp==
先观察hello.jsp文件
```xml
<%--
  Created by IntelliJ IDEA.
  User: admin
  Date: 2020/12/28
  Time: 23:13
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h2>Hello World!</h2>
</body>
</html>
```
hello_jsp.java 
```java
public final class hello_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent,
                 org.apache.jasper.runtime.JspSourceImports 
```
```java
public abstract class HttpJspBase extends HttpServlet
  implements HttpJspPage
```
hello_jsp的父类是HttpJspBase，HttpJspBase又是继承 HttpServlet

==jsp文件本质上就是Servlet==
hello_jsp.java主要方法
```java
//初始化
public void _jspInit() {
  }
//销毁  
public void _jspDestroy() {
  }
//jsp服务，参数为request、response
public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
      throws java.io.IOException, javax.servlet.ServletException {}
```
hello_jsp.java 的_jspService方法

```java
 //参数是request和response
 public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
      throws java.io.IOException, javax.servlet.ServletException {
//判断请求方法
    final java.lang.String _jspx_method = request.getMethod();
    if (!"GET".equals(_jspx_method) && !"POST".equals(_jspx_method) && !"HEAD".equals(_jspx_method) && !javax.servlet.DispatcherType.ERROR.equals(request.getDispatcherType())) {
      response.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, "JSPs only permit GET POST or HEAD");
      return;
    }
//内置了多个对象
	// pageContext 页面上下文
    final javax.servlet.jsp.PageContext pageContext;
    // session 会话信息
    javax.servlet.http.HttpSession session = null;
    // application 就是ServletContext 
    final javax.servlet.ServletContext application;
    //config 配置
    final javax.servlet.ServletConfig config;
    //out 输出对象 等同于response.getWriter()
    javax.servlet.jsp.JspWriter out = null;
    // page 当前页面
    final java.lang.Object page = this;
    //加上方法参数总共九大对象
    //final javax.servlet.http.HttpServletRequest request
    //final javax.servlet.http.HttpServletResponse response
    //下面不需要了解
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;

    try {
      //设置响应的页面类型
      response.setContentType("text/html;charset=UTF-8");
      //以下这些对象我们可以在jsp java 代码中直接使用
      pageContext = _jspxFactory.getPageContext(this, request, response,
      			null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;
	  //输出jsp内容
      out.write("\r\n");
      out.write("\r\n");
      out.write("<html>\r\n");
      out.write("<head>\r\n");
      out.write("    <title>Title</title>\r\n");
      out.write("</head>\r\n");
      out.write("<body>\r\n");
      out.write("<h2>Hello World!</h2>\r\n");
      out.write("</body>\r\n");
      out.write("</html>\r\n");
      //异常处理
    } catch (java.lang.Throwable t) {
      if (!(t instanceof javax.servlet.jsp.SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          try {
            if (response.isCommitted()) {
              out.flush();
            } else {
              out.clearBuffer();
            }
          } catch (java.io.IOException e) {}
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
        else throw new ServletException(t);
      }
    } finally {
      _jspxFactory.releasePageContext(_jspx_page_context);
    }
  }
}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020123121425832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


