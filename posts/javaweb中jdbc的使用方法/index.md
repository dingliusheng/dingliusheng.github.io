# javaweb中jdbc使用方法


<!--more-->

# 什么是jdbc
jdbc: java database connect  java数据库连接
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103085213378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
在pom.xml配置文件中导入jar 包

```xml
  <!--导入mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.25</version>
        </dependency>
```
# idea连接数据库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103090406417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
配置数据连接信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103090806940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
设置时区  Asia/Shanghai
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103091103133.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
### 用java 连接数据库
jdbc 固定步骤：

 1. 加载驱动
 2. 连接数据库
 3. 获取向数据库发送SQL语句的对象
 4. 编写Sql 语句
 5. 执行SQL语句
 6. 获取SQL执行结果
 7. 关闭连接

```java
package com.tin.jdbc;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class MysqlJDBC {
    // Class.forName() 要抛出ClassNotFoundException 
    //DriverManager.getConnection() 要抛出SQLException
    public static void main(String[] args) throws Exception {
        //设置配置信息
        String url="jdbc:mysql://127.0.0.1:3306/smbms?characterEncoding=UTF-8";
        String user="root";
        String password="root";
        //1、加载驱动
        Class.forName("com.mysql.jdbc.Driver");
        //2、连接数据库，connection 代表数据库，DriverManager驱动管理
        Connection connection = DriverManager.getConnection(url, user, password);
        //3、向数据库发送SQL的对象，用它进行增删改查
        Statement statement = connection.createStatement();
        //4、编写查询SQL语句
        String sql="select * from smbms_user";
        //5、执行查询SQL，返回一个ResultSet 结果集，是一个链表
        ResultSet resultSet = statement.executeQuery(sql);
        //6、获取查询结果
        while(resultSet.next()){
            //通过列名获取值resultSet.getObject
            System.out.println("id:"+resultSet.getObject("id"));
            System.out.println("userName:"+resultSet.getObject("userName"));
            System.out.println("userRole:"+resultSet.getObject("userRole"));
        }
        //7、关闭连接，释放资源
        resultSet.close();
        statement.close();
        connection.close();

    }
}

```
# 预编译SQL
```java
package com.tin.jdbc;

import java.sql.*;

public class MysqlJDBC {
    // Class.forName() 要抛出ClassNotFoundException 否则报错
    //DriverManager.getConnection() 要抛出SQLException
    public static void main(String[] args) throws Exception {
        //设置配置信息
        String url="jdbc:mysql://127.0.0.1:3306/smbms?characterEncoding=UTF-8";
        String user="root";
        String password="root";
        //1、加载驱动
        Class.forName("com.mysql.jdbc.Driver");
        //2、连接数据库
        Connection connection = DriverManager.getConnection(url, user, password);
        //3、编写查询SQL语句，？占位符
        String sql="INSERT INTO test (id,userName) VALUES (?,?);";
        //4、预编译sql
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //5、插入多行数据，给？赋值
        for(int i=0;i<10;i++){
            // 给第一个占位符 ？ 赋值
            preparedStatement.setInt(1,i);
            //给第二个占位符 ？ 赋值
            preparedStatement.setString(2,"测试");
            //6、执行SQL
            int res = preparedStatement.executeUpdate();
            if(res==1){
                System.out.println("插入成功");
            }else{
                break;
            }
        }

        //7、关闭连接
        preparedStatement.close();
        connection.close();

    }
}

```
# jdbc事务
事务可以理解为一段SQL语句，事务要么都成功，要么都失败
>开启事务
>事务提交 commit()
>事务回滚 rockback()
>关闭事务
>比如 A 给B转账100元
>1、A的钱-100元
>2、B的钱+100元
>步骤1和2就必须同时成功或者同时失败
>一旦步骤1执行步骤2没执行就会出现业务错误

