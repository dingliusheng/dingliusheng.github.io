# javaweb实例-订单管理系统（3）公共配置


<!--more-->


###  设置数据库配置信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103215552267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
在resource 目录下编写数据库配置文件 db.properties
```properties
drive=com.mysql.jdbc.Driver
url=jdbc:mysql://127.0.0.1:3306/smbms?characterEncoding=UTF-8
userName=root
password=root
```
### 创建操作数据库的工具类

```java
package com.tin.util;

import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

//操作数据库的公共类
public class BaseDao {
    private static String driver;
    private static String url;
    private static String userName;
    private static String password;
    //静态代码块类加载的时候就执行,获取数据库配置信息
    static {
        //通过类加载器读取对应的资源
        InputStream is = BaseDao.class.getClassLoader().getResourceAsStream("db.properties");
        Properties properties = new Properties();
        try {
            properties.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
        driver=properties.getProperty("driver");
        url=properties.getProperty("url");
        userName=properties.getProperty("userName");
        password=properties.getProperty("password");

        try {
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    //获取数据库链接
    public static Connection getconnection() throws Exception {
        Connection connection=null;
        //加载驱动
        Class.forName(driver);
        //获取数据库连接
        connection= DriverManager.getConnection(url,userName,password);
        return connection;
    }

    //编写查询公共方法
    public static ResultSet excuteQuery(Connection connection,String sql,Object[] parms) throws Exception {
        ResultSet resultSet=null;
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < parms.length; i++) {
            preparedStatement.setObject(i+1,parms[i]);
        }
        resultSet=preparedStatement.executeQuery(sql);
        return  resultSet;
    }

    //编写增删改公共方法
    public static int  excuteUpdate(Connection connection,String sql,Object[] parms) throws Exception {
        int updateRows;
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < parms.length; i++) {
            preparedStatement.setObject(i+1,parms[i]);
        }
        updateRows=preparedStatement.executeUpdate(sql);
        return  updateRows;
    }

    //关闭资源
    public static boolean closeResource(Connection connection,PreparedStatement preparedStatement,ResultSet resultSet){
        boolean flag=true;
        if(connection!=null){
            try {
                connection.close();
                connection=null;
            }catch (Exception e){
                e.printStackTrace();
                flag=false;
            }
        }
        if(connection!=preparedStatement){
            try {
                preparedStatement.close();
                preparedStatement=null;
            }catch (Exception e){
                e.printStackTrace();
                flag=false;
            }
        }
        if(resultSet!=null){
            try {
                resultSet.close();
                resultSet=null;
            }catch (Exception e){
                e.printStackTrace();
                flag=false;
            }
        }

        return flag;
    }
}

```
### 用过滤器Filter 设置网页中文编码

```java
package com.tin.filter;

import javax.servlet.*;
import java.io.IOException;

public class EncodingFilter implements Filter {
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //设置中文编码
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html");
        servletResponse.setCharacterEncoding("utf-8");
        filterChain.doFilter(servletRequest,servletResponse);
    }

    public void destroy() {

    }
}
```
注册过滤器

```xml
<filter>
    <filter-name>Encoding</filter-name>
    <filter-class>com.tin.filter.EncodingFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>Encoding</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```


