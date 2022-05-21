# jsp基础语法和指令


<!--more-->



### Maven导入jsp依赖jar包
```xml
 <!--        Servlet依赖-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <!--        JSP依赖-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
        </dependency>
        <!--引入Javaee7开始-->
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
        </dependency>
        <!--引入Javaee7结束-->
        <!--引入JSTL开始-->
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
```
###  jsp 基础语法
jsp 表达式

```java
<%-- jsp 表达式
     作用：用来将程序的输出，输出到客户端
     格式：<%=变量或者表达式%>
--%>
<%=new java.util.Date()%>
```
jsp 脚本片段,可以和html任意嵌套

```java
<%-- jsp 脚本片段
     作用：就是一段java程序
     格式：<%一段java程序%>
--%>
<%
    int sum=0;
    for(int i=0;i<10;i++){
    >%
    <h2>循环</h2>
    <%
        sum+=i;
    }
    out.println("<h1>Sum="+sum+"</h1>");
%>
```
jsp 声明
```java
<%!
    //jsp 声明，定义全局变量（类变量）
    int a=0;
%>
```
本质上就是通过<%!java声明代码%>是放在XXX_jsp.java的类中，是全局的作用域
<%java代码%>是放在XXX_jsp.java下的_jspService方法中，是局部的作用域
### jsp指令
```xml
<%--指定页面格式--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%--指定发生错误时跳转的页面--%>
<%@ page errorPage="/WEB-INF/500.jsp" %>
<%--加载其他jsp的资源--%>
<%@ include file="hello.jsp"%>
```
### jsp 内置对象和作用域

 - PageContext  
 - Request   
 - Response
 - Session    
 - Application 【ServletContext】
 - config 【Servletconfig】
 - out
 - page
 - exception

PageContext：一般不用这个保存数据
Request:客户端向服务器发送请求，产生的数据，用户看完就没用了，比如：新闻
Session:客户端向服务器发送请求，产生的数据，用户看完后还可能用到，比如：购物车
Application:客户端向服务器发送请求，产生的数据，用户看完后其他用户还可能用到，比如：游戏中的聊天信息
```java
 <%  //保存的数据只在一个页面中有效
        pageContext.setAttribute("num1","1");
        //保存的数据只在一次请求中有效，请求转发会携带这个信息
        request.setAttribute("num2","2");
        //保存的数据只在服务器有效，从打开服务器到关闭服务器
        application.setAttribute("num3","3");
        //保存的数据只在一次会话有效，从打开浏览器到关闭浏览器
        session.setAttribute("num4","4");
        //从底层到高层的作用域：page-->request-->session-->application
		//从底层往高层找所需要的数据
		String  num3 =(String) pageContext.findAttribute("num3");
    %>
```
### EL表达式
格式：${ }
 **获取数据**
 **执行运算**
  **获取web常用对象**
### jsp标签
```xml
  <!--添加hello.jsp的内容-->
   <jsp:include page="hello.jsp"></jsp:include>
     <!--转发到hello.jsp-->
    <jsp:forward page="/hello.jsp">
        <!--设置跳转链接参数-->
        <jsp:param name="name" value="小明"/>
    </jsp:forward>
```
## jstl标签

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--
引用jstl核心标签库
-->
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form>
        <input type="text" name="user" value="${param.user}">
        <input type="submit" value="登入">
    </form>
    <！--
    测试登入的是否是admin
	c:if 就是java中的if逻辑判断如果为true，则执行
	c:out 输出内容
-->
    <c:if test="${param.user=='admin'}" var="isAdmin">
        <c:out value=" Welcome Admin    "></c:out>
    </c:if>
    <c:out value="${isAdmin}"></c:out>
    <！--
	c:forEach 遍历数组
	var 遍历的变量名 等同于int item=list[i];
	items 遍历的对象 就是list
	从0 开始计数
	begin end step 开始位置 结束位置 步长
-->
     <%
        ArrayList<String> list = new ArrayList<>();
        list.add("张三");
        list.add("李四");
        list.add("王五");
        list.add("赵六");
        list.add("宋七");
        request.setAttribute("list",list);
    %>
    <c:forEach items="${list}" var="item" begin="1" end="3" step="2" >
        <c:out value="${item}"></c:out>
    </c:forEach>
</body>
</html>
```


