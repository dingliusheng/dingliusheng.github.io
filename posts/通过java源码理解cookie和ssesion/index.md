# 通过Java源码理解cookie 和 ssesion


<!--more-->


### 一个网站怎么辨别你是否来过？
1、服务器给客户端一个特定cookie(存有身份认证信息的一个对象)
2、客户端下次浏览这个网站时，就会将这个cookie随请求发送到服务器。服务器就会根据cookie携带的信息来辨别你是否登入过这个网站
简单的理解就是：
	==cookie就是存储信息的一个容器。==
	抓取百度的包：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225090510757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
### 会话
#### 1、什么是会话？
打开会话：用户打开一个浏览器，进入一个网站如bilibili
结束会话：离开网站或退出登入
会话：就是你访问一个网站和它的网页的过程。
#### 2、cookie保存会话
客户端技术（响应，请求），保存信息

```java
package com.tin.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class CookieServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置中文编码
        req.setCharacterEncoding("utf-8");
        resp.setCharacterEncoding("utf-8");
        PrintWriter out = resp.getWriter();
        Cookie user = new Cookie("username", "123456");
        //获取浏览器的cookies(有多个cookie)
        Cookie[] cookies = req.getCookies();
        //如果是第一次进入网站就会给浏览器一个带有（username,123456）键值对信息的cookie
        //如果不是就输出网站所给的特殊标记
        //这里是通过判断cookie的name来进行区分用户身份
        int count;
        for(count=0;count<cookies.length;count++){
        	//用cookie携带信息认证你是否是网站用户
            if(cookies[count].getName().equals(user.getName())
                    &&cookies[count].getValue().equals(user.getValue())){
                out.println(user.getName()+""+user.getValue());
                break;
            }
        }
        if(count==cookies.length){
            resp.addCookie(user);
        }
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}

```

#### 3、session保存会话
服务端技术，保存信息
==Session==是唯一的

```java
package com.tin.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class SessionServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("utf-8");
        resp.setCharacterEncoding("utf-8");
        resp.setContentType("text/html");
        //得到Sesion
        HttpSession session = req.getSession();

        Object attribute = session.getAttribute("user");
        if(attribute==null){
            resp.getWriter().println("第一次登入");
            //用session存储用户信息
            session.setAttribute("user","123456");
        }else{
            resp.getWriter().println("欢迎再次浏览");
        }
    }
    }

```
cookie和session区别

 - cookie是把用户的数据写给用户浏览器，浏览器可以保存==多个cookie==，浏览器关闭，cookie就失效了
 - session是一个特殊的cookie,==一个浏览器只能保存一个==，服务端保存（保存重要信息，减少服务器资源的浪费），session是服务器创建的，当服务器关闭或者手动设置session的失效时间才能使session失效

session 应用场景：

 - 保存用户登入信息
 - 购物车信息
 - 在整个网站中经常会使用的数据


