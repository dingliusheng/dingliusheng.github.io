# 简单理解MVC三层模型


<!--more-->

# 什么是MVC三层模型
Model 模型、View 视图、Controller 控制器
 前期模型：![在这里插入图片描述](https://img-blog.csdnimg.cn/2021010222032311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
客户端直接访问控制层，控制层可以直接操作数据库
>servlet--->jdbc--->数据库
>不足之处：程序耦合度高，不利于维护
>servlet代码中需要处理：
>	请求、响应、视图跳转、JDBC、业务代码、逻辑代码（功能复杂）

改进：
>架构：将业务逻辑处理封装成模块（多加一层来实现JDBC、业务代码、逻辑代码），控制层servlet 只需要调用这个模块
>拓展：架构的基本思想就是将功能拆分模块化，例如JDBC,我们不需要去掌握如何用java驱动mysql、oracle、sqlserver等各种不同服务器的底层实现方法，只需要去调用JDBC这一层模块

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210102223553835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
Model

 - 业务处理：业务逻辑service
 - 数据持久层：数据的增删改查 Dao

View

 - 展示数据; 
 - 提供链接发起Servlet请求;如超链接、表单提交、鼠标点击跳转等

 Controller
 

 - 接收用户的请求（req:请求参数、Session信息）
 - 交给业务层处理对应的请求
 - 控制视图的跳转
 
 >用户登入--->接收用户的登入请求--->处理用户的请求（获取用户登入的账号和密码）--->交给业务层处理登入业务（判断用户密码是否正确）--->Dao层查询用户名和密码是否正确--->访问数据库的数据


