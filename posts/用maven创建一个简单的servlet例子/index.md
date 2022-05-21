# 初识servlet

<!--more-->

### 一、 Servlet介绍
 - servlet就是Sun公司开发动态网站的一门技术
 - Sun公司在这些API中提供一个接口叫做：servlet
 - 开发servlet程序步骤：
	  1、编写一个类，实现servlet接口
	  2 、把开发好的类部署到web服务器中
###  二、创建一个servlet实例
IDEA创建一个空的Maven工程（==用专业版Idea==,如何破解自行百度）
File->new->project
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219102740728.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
选择maven工程，这里不勾选模板
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219103148155.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
设置文件名，文件路径
GroupID是项目组织唯一的标识符，实际对应JAVA的包的结构，是main目录里java的目录结构。
ArtifactID就是项目的唯一的标识符，实际对应项目的名称，就是项目根目录的名称。
一般GroupID就是填com.test这样子
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219103457671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
Maven 空项目文件结构如下，删除src (==作为父项目管理依赖，具体功能由子模块实现==)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219104658660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
在Maven项目中添加webapp模块
选择项目的根目录->new ->Module
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219105101777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
创建webapp子模块
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219105406610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
设置子模块的文件名和文件路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219105625607.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
配置Maven和Maven仓库（可以自己建，也可以使用idea自带的Maven2或Maven3）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219105922105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
完成后，父项目的pom.xml文件下会生成：

```xml
<modules>
        <module>servlet</module>
    </modules>
```
子模块的pom.xml会生成：
```xml
<parent>
    <groupId>com.test</groupId>
    <artifactId>Test</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>
```
==如果没有需要手动添加，子模块能使用父项目导入的依赖（jar包），不需要再配置pom.xml文件==
在父项目的pom.xml导入JSP、Servlet依赖

```xml
    <dependencies>
        <!--Servlet依赖-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <!--JSP依赖-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
        </dependency>
    </dependencies>

```
在子模块中添加java文件和resource文件，java文件鼠标右键，将java设置为Source文件（这样才可以添加java代码）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219111526666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219111742746.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
在java文件下创建com.test.servlet包，在这个包下创建HelloServlet类
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219112216696.png#pic_center)
编写HelloServlet类
```java
package com.tin.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //输出流
        PrintWriter writer = resp.getWriter();
        writer.print("hello,welcome to servlet");
    }
}

```
编写servlet映射
servlet写的java程序，但是需要通过浏览器访问，而浏览器需要连接web服务器，所以需要在web服务中注册编写的servlet，还需要给一个浏览器能访问的路径：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219155426652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
在web.xml添加servlet信息

```xml
<!--注册servlet-->
  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.tin.servlet.HelloServlet</servlet-class>
  </servlet>
  <!--servlet 请求路径-->
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219155841761.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
配置tomcat server
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219160034173.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219160248108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
然后添加需要执行的webapp压缩包
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219160759104.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
完成配置后就可以执行项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219161010969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


