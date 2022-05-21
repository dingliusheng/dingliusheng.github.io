# javaweb实例-订单管理系统（5）用户退出登入模块


<!--more-->



### 用户退出登入流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110181853934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
用户点击主页中的退出--》/jsp/logout.do 请求--》LogoutServlet 移除session用户信息
优化：当用户退出登入后不能再访问主页等资源页面，实现权限控制
访问主页等资源页面的请求--》过滤器--》判断用户信息是否存在--》存在就能访问--》不存在就跳转到error.jsp

### Logoutservlet

```java
package com.tin.servlet;

import com.tin.util.Constants;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class LogoutServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //移除session存储的用户信息
        req.getSession().removeAttribute(Constants.USER_SESSION);
        req.getRequestDispatcher("../login.jsp").forward(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```
### SysFilter

```java
package com.tin.filter;

import com.tin.pojo.UserInfo;
import com.tin.util.Constants;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class SysFilter implements Filter {
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request=(HttpServletRequest) servletRequest;
        HttpServletResponse response=(HttpServletResponse) servletResponse;
        UserInfo user = (UserInfo) request.getSession().getAttribute(Constants.USER_SESSION);
        //如果用户信息为空则转发到error.jsp
        if(user!=null){
            filterChain.doFilter(servletRequest,servletResponse);
        }else{
       		//这里是全路径
            response.sendRedirect("/ManageSystem_war/error.jsp");
        }
    }
    public void destroy() {

    }
}
```
### 在web.xml 注册

```xml
 <!--监听用户是否登录过滤器-->
  <filter>
    <filter-name>sysFilter</filter-name>
    <filter-class>com.tin.filter.SysFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>sysFilter</filter-name>
    <url-pattern>/jsp/*</url-pattern>
  </filter-mapping>
  <!--注册注销的logoutServlet-->
  <servlet>
    <servlet-name>logout</servlet-name>
    <servlet-class>com.tin.servlet.LogoutServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>logout</servlet-name>
    <url-pattern>/jsp/logout.do</url-pattern>
  </servlet-mapping>
```


