# 计算机网络编程--基础概念


<!--more-->



# 网络编程

### 一、概述

网络编程的目的：传播信息、数据交换

实现条件：

1. 如何准确的定位网络上的一台主机，利用==ip:端口==

2. 找到了这个主机，如何传输数据： 传输协议：==tcp\udp\ftp==等

### 二、网络模型
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210192326118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

### 三、IP地址

IP地址：唯一定位一台网络上计算机

IP地址分类：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210192348129.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

**IP私有地址：**

在IP地址3种主要类型里，各保留了3个区域作为私有地址，其地址范围如下： 

A类地址：10.0.0.0～10.255.255.255 

B类地址：172.16.0.0～172.31.255.255 

C类地址：192.168.0.0～192.168.255.255

### 四、端口

端口：表示计算机一个程序的进程

==不同的进程有不同的端口，用来区分软件==

```git
netstat -a -n   //window查看端口信息
```

​	被规定 0-65535，==TCP、UDP各有65535个==，单个协议下，端口号不能冲突

端口划分：

- 公有端口：0-1023 ，==不建议占用==
  - http:80
  - https:443
  - ftp:21
  - Telent:23
- 程序注册端口：1024-49151，分配给用户或者程序
  - Tomcat:8080
  - MySql:3306
  - Oracle:1521
- 动态、私有端口：49152-65535 ，==不建议使用==

### 五、通信协议

协议：数据传输约定
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210192428247.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
思路：以大化小，分层

- TCP
  - 建立可靠连接，确定双方都建立连接后数据传输，可靠
  - ==三次握手，四次挥手==
  - 效率低
- UDP
  -   不建立连接，数据发送方只需发送数据，不可靠
  - 效率高




