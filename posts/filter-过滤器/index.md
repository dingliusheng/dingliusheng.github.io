# 简单理解Filter过滤器


<!--more-->

# 什么是过滤器
Filter :过滤器，用来过滤网站数据

 - 处理中文乱码
 - 拦截不需要的请求
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210102230556939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
### 创建一个过滤器
注意要先导入servlet 依赖，选对Filter 接口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210102233055251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
创建过滤器实现中文编码
```java
package com.tin.filter;

import javax.servlet.*;
import java.io.IOException;

public class CharacterEncodingFilter implements Filter {

    @Override
    //初始化，服务器启动时就完成初始化
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //设置中文编码
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html");
        servletResponse.setCharacterEncoding("utf-8");
        //chain:链 过滤器的代码在过滤特定请求时都会执行（满足filter注册映射的url请求路径时）
        //必须执行下面这条代码，让请求转接到servlet，否则就会拦截这个请求
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    //销毁 服务器关闭时执行
    public void destroy() {

    }
}

```
在web.xml注册过滤器

```xml
<filter>
    <filter-name>Encoding</filter-name>
    <filter-class>com.tin.filter.CharacterEncodingFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>Encoding</filter-name>
    <!--设置过滤器适用的请求路径-->
    <url-pattern>/session/*</url-pattern>
 </filter-mapping>
```


