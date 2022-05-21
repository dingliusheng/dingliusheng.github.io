# javaweb实例-订单管理系统（4）用户登入模块


<!--more-->




### 前言
本来想尝试一下自己做demo，但弄起来很麻烦，javaweb所学部分就是看了B站up主遇见狂神说的教学视频，详细资源自己在b站搜索一下
### 前端页面
资源链接：
[网站链接](https://gitee.com/babobenNB/smbms?_from=gitee_search)
获取到login.jsp 和 对应的images、css 、jsp资源
文件位置如图放置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107192638792.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
2、在web.xml设置login.jsp为服务器首页

```xml
<!--设置网站首页-->
  <welcome-file-list>
    <welcome-file>login.jsp</welcome-file>
  </welcome-file-list>
```
### 登入模块流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210109214354150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

1、数据库中存储用户信息的表
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021010910373543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
2、JavaBean 对应这个表的实体类（加无参构造，get/set方法）

```java
package com.tin.pojo;

public class UserInfo {
    private int userID;
    private String userName;
    private String userPassword;
    private int roleID;
    private String phone;
    private String address;
    //通过联合查询获取用户的角色名称
    private String userRoleName;
	//再添加get/set属性的方法和无参构造 
}
```
3、构建java 加载mysql 的共有方法的工具类BaseDao
获取mysql 加载需要的配置信息  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210109104422423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
```java
 	private static String driver;//mysql 驱动包
    private static String url;
    private static String userName;
    private static String password;
    private static PreparedStatement preparedStatement;
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
        driver=properties.getProperty("a");
        url=properties.getProperty("url");
        userName=properties.getProperty("userName");
        password=properties.getProperty("password");

        try {
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```
获取操作数据库的connection 对象

```java
 //获取数据库链接
    public static Connection getconnection() throws Exception {
        Connection connection=null;
        //加载驱动
        Class.forName(driver);
        //获取数据库连接
        connection= DriverManager.getConnection(url,userName,password);
        return connection;
    }
```
封装SQL查询的类方法

```java
    //编写查询公共方法
    //connecton 操作数据库对象
    //sql  要执行的SQL语句
    // parms SQL预编译待填的参数
    // resultSet 存储返回的结果，类似一张数据表
    public static ResultSet excuteQuery(Connection connection,String sql,Object[] parms) throws Exception {
        ResultSet resultSet=null;
        preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < parms.length; i++) {
            preparedStatement.setObject(i+1,parms[i]);
        }
        resultSet = preparedStatement.executeQuery();
        return  resultSet;
    }
```
封装增删改查的类方法

```java
//编写增删改公共方法
//返回影响的数据表行数
    public static int  excuteUpdate(Connection connection,String sql,Object[] parms) throws Exception {
        int updateRows;
        preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < parms.length; i++) {
            preparedStatement.setObject(i+1,parms[i]);
        }
        updateRows = preparedStatement.executeUpdate();
        return  updateRows;
    }
```
封装关闭资源的类方法

```java
 //关闭资源
 //基本原则：谁打开资源就由谁来关闭
    public static boolean closeResource(Connection connection,ResultSet resultSet){
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
        if(preparedStatement!=null){
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
```
4、定义获取用户信息的接口

```java
package com.tin.Dao.user;

import com.tin.pojo.UserInfo;

public interface UserDao {
    //得到登入的用户对象（前面说的对应userinfo表对应的实体类）
    public UserInfo getLoginUser( String userName,String password);
}
```
5、接口的实现类

```java
package com.tin.Dao.user;

import com.tin.pojo.UserInfo;
import com.tin.util.BaseDao;
import org.junit.jupiter.api.Test;

import java.sql.Connection;
import java.sql.ResultSet;

public class UserDaoImpl implements UserDao{
    public UserInfo getLoginUser(String userName, String password) throws Exception {
        Connection connection = BaseDao.getconnection();
        String sql = "select * from userInfo where userName = ? and userPassword = ?";
        Object[] params={userName,password};
        ResultSet resultSet = BaseDao.excuteQuery(connection, sql, params);
        UserInfo userInfo = new UserInfo();
        if(resultSet.next()){
            userInfo.setUserID(resultSet.getInt("userID"));
            userInfo.setUserName(resultSet.getString("userName"));
            userInfo.setUserPassword(resultSet.getString("userPassword"));
            userInfo.setPhone(resultSet.getString("phone"));
            userInfo.setAddress(resultSet.getString("address"));
            userInfo.setRoleID(resultSet.getInt("roleID"));
        }
        BaseDao.closeResource(connection,resultSet);
        return userInfo;
    }
}

```
6、定义login业务处理的接口

```java
package com.tin.service;

import com.tin.pojo.UserInfo;

public interface LoginService {
    //获取用户登入的对象，如果用户或密码不正确就返回null
    public UserInfo login(String userName,String password);
}
```
7、login业务接口实现类

```java
package com.tin.service;

import com.tin.Dao.user.UserDao;
import com.tin.Dao.user.UserDaoImpl;
import com.tin.pojo.UserInfo;
import com.tin.util.BaseDao;
import org.junit.jupiter.api.Test;

import java.sql.Connection;

public class LoginServiceImpl implements LoginService{
    //业务层都会调用dao层，引入dao层
    private UserDao userDao;
    public LoginServiceImpl() {
        userDao=new UserDaoImpl();
    }
    
    public UserInfo login(String userName, String password) {
        UserInfo user=null;
        try {
            user = userDao.getLoginUser(userName, password);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println(user.getUserName());
        return user;
    }
}

```
8、LoginServlet 

```java
package com.tin.servlet;

import com.tin.pojo.UserInfo;
import com.tin.service.LoginService;
import com.tin.service.LoginServiceImpl;
import com.tin.util.Constants;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取输入的用户名和密码,通过input 中name
        String userName=req.getParameter("userName");
        String userPassword=req.getParameter("userPassword");
        //调用业务接口，获取用户对象
        LoginService loginService=new LoginServiceImpl();
        UserInfo user = loginService.login(userName, userPassword);
       // System.out.println(user.getUserName());
        //如果对象为空就是密码错误重定向到当前页面，提示用户或密码错误
        if(user.getUserName()!=null){
            //登入成功重定向到网站主页
            //用session保存用户对象
            req.getSession().setAttribute(Constants.USER_SESSION,user);
            req.getRequestDispatcher("/jsp/frame.jsp").forward(req,resp);
        }else{
            req.setAttribute("error","用户或密码不正确");
            req.getRequestDispatcher("login.jsp").forward(req,resp);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```


