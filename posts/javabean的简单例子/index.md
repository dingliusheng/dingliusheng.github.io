# 一个 javabean 的简单例子


<!--more-->



### javaBean
javaBean 特定写法

 - 必须有一个无参数构造
 - 属性必须私有化
 - 必须有对应的get/set方法
 
== 一般用来和数据库字段作映射==
 
 - 表--->类
 - 字段--->属性
 - 行记录--->对象
模拟一张student 表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210102193606602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
构造对应的类

```java
package com.tin.pojo;

public class Student {
    private int id;
    private String name;
    private int age;

    public Student(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public Student() {
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}

```
在jsp使用JavaBean
```xml
<%@ page import="com.tin.pojo.Student" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<!--
    等同于：
    Student student = new Student();
    student.setName("张三");
    student.setAge(22);
    
    student.getName();
    student.getAge();
!-->
    <jsp:useBean id="student" class="com.tin.pojo.Student" scope="page" />
    <jsp:setProperty name="student" property="id" value="1" />
    <jsp:setProperty name="student" property="name" value="张三" />
    <jsp:setProperty name="student" property="age" value="22" />

    姓名：<jsp:getProperty name="student" property="name"/>
    年龄：<jsp:getProperty name="student" property="age"/>


</body>
</html>

```


