# javaweb实例-订单管理系统（1）项目搭建


<!--more-->




### 一、 搭建一个maven web项目
选择maven,然后勾选，选择webapp模板
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103172225339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
只需设置项目名称
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103172416867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
选好Maven路径、maven\conf\setting 路径、maven仓库 （可以选择idea自带的maven，则不需要修改），点击完成
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103172754854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
### 二、配置tomcat 服务器
说明：专业版的idea才自带tomcat的服务，如果不是请自行百度解决
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103173703197.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103173948673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103174411139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
配置tomcat
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103174438855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
添加tomcat 运行的web项目（war 包就是web 的压缩文件），点下面的+号
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103174548902.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103174624320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
配置完成后，测试运行javaweb ,能访问这个页面就代表项目配置成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103174741595.png#pic_center)

### 三、导入项目依赖的jar 包
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103173255686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
打开pom.xml 文件添加项目所需的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.tin</groupId>
  <artifactId>ManageSystem</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <dependencies>
    <!--单元测试-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--导入servlet-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>
    <!--导入jsp-->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.3</version>
    </dependency>
    <!--导入jstl表达式的依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
    <!--导入standard标签库-->
    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
    </dependency>
    <!--导入mysql驱动-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.25</version>
    </dependency>
  </dependencies>
</project>

```
### 创建项目包结构
按图创建文件结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103180203758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)





