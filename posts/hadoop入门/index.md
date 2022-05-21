# hadoop入门文档


<!--more-->







# Hadoop入门

<nav>
<a href="#一概念">一、概念</a><br/>
<a href="#二环境准备">二、环境准备</a><br/>
<a href="#三Hadoop生产集群搭建">三、Hadoop生产集群搭建</a><br/>
</nav>

## 一、概念

### 1.1、Hadoop是什么

1.   `Hadoop` 是一个由Apache基金会所开发的`分布式系统基础架构` 。
2.   主要解决海量数据的`存储`和海量数据的`分析计算问题`
3.   广义上来说，Hadoop 通常是指一个更广泛的概念———`Hadoop生态圈`(HBase、Hive等)

### 1.2 、Hadoop发展历史

1. Hadoop创始人 `Doug Cutting` ，为了实现与 Google类似的全文搜索功能，在`Lucene`框架基础上进行优化升级，查询引擎和索引引擎。
2. 2001年年底Lucene成为`Apache基金会`的一个子项目。
3. 对于海量数据场景，Lucene框架面对与Google同样的困难，**存储海量数据困难，检索海量数据慢**。
4. 学习和模仿Google解决这些问题的办法：微型版 Nutch。
5. 参考Google在大数据的三篇论文（GFS--->HDFS ，Map-Reduce--->MR  ，BigTable--->HBase ）。
6. 2003-2004年，Google公开了部分GFS和MapReduce思想的细节，以此为基础`Doug Cutting`等人用了**2年业余时间**实现了DFS和MapReduce机制，使Nutch性能飙升。
7. 2005年Hadoop作为Lucene的子项目Nutch的一部分正式引入Apache基金会。
8. 2006年3月，Map-Reduce和Nutch Distributed File System (NDFS) 分别纳入到Hadoop项目中，Hadoop就此正式诞生，标志着大数据时代来临。
9. logo来源于`Doug Cutting`儿子的玩具大象

### 1.3 Hadoop三大发行版本

Hadoop 三大发行版本： Apache 、Cloudera 、Hortonworks 。

- Apache 版本最原始（最基础)的版本，对入门学习最好（2006年）。
- Cloudera 在Apache版本基础上集成了很多大数据框架，对应的产品CDH（2008年）。
- Hortonworks 文档较好，对应产品HDP（2011年）。
- Hortonworks 被Cloudera 收费，推出CDP（2018年）。
- 两家公司都是在Apache的基础上进行集成功能，提高用户体验，借此收费，最终合并联手。

### 1.4、Hadoop优势

1. **高可靠性**：Hadoop 底层数据有多个备份，所以即使Hadoop某个数据存储或计算出现故障，也不会导致数据丢失。
2. **高扩展性**：动态增加、减少服务器，在集群间分配任务数据，可方面的扩展数以千计的节点。
3. **高效性**：在MapReduce 的思想下，Hadoop是==并行工作==的，加快任务处理速度。
4. **高容错性**：能自动将失败的任务重新分配。

### 1.5、Hadoop组成（面试重点）

#### 1.5.1、Hadoop1.x、2.x、3.x 区别

- Hadoop1.x 组成： Common(辅助工具)   + HDFS（数据存储） + ==MapReduce(计算+资源调度)==
- Hadoop2.x 组成： Common(辅助工具)   + HDFS（数据存储） + ==Yarn(资源调度) + MapReduce(计算)==
- Hadoop3.x 组成和Hadoop2.x 没有区别

#### 1.5.2、HDFS架构概述

​	`Hadoop Distributed File System` ， 简称`HDFS`，是一个分布式文件系统。

- **NameNode**: 存储文件的**元数据**，如**文件名、文件目录结构、文件属性**（生成时间、副本数、文件权限），以及每个文件的**块列表**和**块所在的DataNode**等。

- **DataNode**：在本地文件系统**存储文件块数据**，以及**块数据的校验和**

- Secondary NameNode (**2NN**)：**每隔一段时间对NameNode元数据备份**

  理解：NameNode记录了数据存储信息，DataNode就是数据存储块，2NN是对NameNode的备份，以防止NameNode的丢失情况

#### 1.5.3、YARN架构概述

`YetAnother Resource Negotiator` 简称`YARN`，另一种资源协调者，是Hadoop的资源管理器。

- **ResourceManager**（RM）：整个集群**资源（内存、cpu等）**的管理者

- **NodeManager**（NM）：单个节点服务器资源的管理者

- **ApplicationMaster**（AM）：单个任务运行的管理者

- **Container**：容器，相当于一台独立的服务器，里面封装了任务运行所需要的资源，如**内存，CPU、磁盘、网络**等（将一个计算资源虚拟划分多个容器，并分配好内存、CPU、磁盘、网络等）

  说明：客户端可以有多个，集群上可以运行多个ApplicationMaster，每个NodeManager上可以有多个Container

  理解：**ApplicationMaster**根据任务运行资源需要向**ResourceManager**申请对应的资源，**ResourceManager**分配一个**NodeManager**并在**NodeManager**划分出满足任务资源需要的容器，任务完成后，删除容器，释放对应资源，能灵活分配集群资源

#### 1.5.4、MapReduce 架构概述

  **Map阶段**：并行处理输入数据

  **Reduce阶段**：对Map结果进行汇总

  理解：将待分析的数据划分成N分，分配给多个服务器进行计算，再将各个计算结果进行汇总

#### 1.5.5、HDFS 、YARN、MapReduce 三者关系

首先，通过数据采集获取海量数据需要HDFS来存储，当查询或计算这些海量数据时，需要先给查询或计算任务分配CPU、内存等资源，并将大量的输入数据进行拆分，分给多个服务器并行计算，最终将结果汇总。

#### 1.6、大数据生态体系

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412165855408.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
## 二、Hadoop运行环境搭建

### 2.1、前期环境准备

- VMware 虚拟机安装
- CentOS7 安装（都可以百度有对应的教程）

#### 2.1.1、配置虚拟机网络

1. ​	打开 编辑--》虚拟网络编辑器

2. ​    选择 删除VMnet 8 ，添加新网络VMnet2， 选择NAT模式，NET设置修改网关（不建议），DHCP设置修改分配的IP地址（虚拟机设置的IP必须要在这个范围内）一般是 xxx.xxx.xxx.3~254

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412165951882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


3. 在Windows下，打开网络和Internet设置---》更改网络配置----》更改适配器选项--》找到VMnet2(刚才添加的信网络)---》点击属性---》选择Internet协议版本4（TCP\IPv4）

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170010197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


4. 设置IP地址，子网掩码，默认网关，首选DNS服务器和网关一样，备选DNS服务器国际默认8.8.8.8

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170024318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


5. 在VMware ，点击网络适配器，选择自定义，选择新建的网络VMnet2

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170038461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


6. 登入虚拟机，编辑网络配置文件 **/etc/sysconfig/network-scripts/ifcfg-ens33** ，内容如下，编辑后重启或 执行**source /etc/sysconfig/network-scripts/ifcfg-ens33**命令

  ```bash
  TYPE="Ethernet"
  PROXY_METHOD="none"
  BROWSER_ONLY="no"
  #设置为静态ip,默认是dchp,动态分配ip地址
  BOOTPROTO="static"
  DEFROUTE="yes"
  IPV4_FAILURE_FATAL="no"
  IPV6INIT="yes"
  IPV6_AUTOCONF="yes"
  IPV6_DEFROUTE="yes"
  IPV6_FAILURE_FATAL="no"
  IPV6_ADDR_GEN_MODE="stable-privacy"
  NAME="ens33"
  UUID="b400b844-2c25-4434-8f17-1da5faf08353"
  DEVICE="ens33"
  ONBOOT="yes"
  #设置网络IP地址，网关，DNS服务器地址
  IPADDR="192.168.75.100"
  GATEWAY="192.168.75.2"
  DNS1="192.168.75.2"
  ```

7. 验证是否完成配置，**ping  www.baidu.com**  ，如果能接收数据就表示成功完成网络配置

8. ps:不要通过克隆来创建其他虚拟机，会出现网络bug,直接从最初镜像或干净的虚拟机进行同步

#### 2.1.2、配置主机名

1. 登入linux，编辑主机名称，**vi /etc/hostname**  

2. 修改主机名称映射（在本机中自动将相应的字符转为ip地址，如localhost 对应127.0.0.0）

     **vi /etc/hosts**  

     ```bash
     127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
     ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
     #添加下列信息
     192.168.75.100 Hadoop100
     192.168.75.101 Hadoop101
     192.168.75.102 Hadoop102
     192.168.75.103 Hadoop103
     ```

3. 重启系统 （或者用source 命令）

#### 2.1.3、安装epel-release

Extra Packages for Enterprise Linux 是红帽系的操作系统提供额外的软件包，适用于RHEL 、CentOS 和 Scientific Linux。相当于是一个软件仓库，大多数rpm包在官方repository是找不到的

```bash
yum install -y epel-release
```

**如果Linux系统安装的是最小系统版，还需要配置如下工具：**

net-tools:工具包集合，包含ifconfig等命令

```bash
yum install -y net-tools
```

vim 编辑器

```bash
yum install -y vim
```

#### 2.1.4 关闭防火墙、关闭防火墙开机自启

一般公司对外网的入口建防火墙，内部服务器不建防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld.service
```

#### 2.1.5、 卸载虚拟机自带的JDK

==如果是虚拟机最小化安装不需要执行这一步==

```bash
 #查询所安装的所有rpm软件包
rpm -qa
#忽略大小写过滤
grep -i
#表示每次只传递一个参数
xargs -n1
#强制卸载软件
rpm -e --nodeps
#完整命令
rpm -qa |grep -i java |xargs -n1 rpm -e --nodeps
```

###    2.2、远程访问工具

- xshell+xftp
- （或）putty + WinSCP
- 安装教程自行百度 

### 2.3、克隆虚拟机(不推荐，可以用下面的同步更新脚本)

1、打开VMware，选择要克隆的模板机，右键管理，克隆，选择**创建完整性克隆**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170135839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)



2、设置克隆机的ip地址和主机名，参考上面的方法（如果克隆的是已经配置好的linux系统，则只需修改 /etc/sysconfig/network-scripts/ifcfg-ens33  和 /etc/hostname ）

### 2.4、安装JDK

- 1、XFTP软件将JDK的压缩包传输到 /opt/software/目录下

- 2、解压压缩包：

```bash
cd /opt/software/
tar -zxvf jdk-8u161-linux-x64.tar.gz -C /opt/module/
```

- 3、配置环境变量

```bash
cd /etc/profile.d
#创建my_env.sh 文件
vi my_env.sh
```

在 my_env.sh 文件添加如下内容

```bash
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_161
export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
```

重新启动配置文件

```bash
source /etc/profile
#验证是否配置完成
java
```

### 2.5、安装Hadoop

- 1、XFTP软件将Hadoop的压缩包传输到 /opt/software/目录下

- 2、解压压缩包：

```bash
cd /opt/software/
tar -zxvf hadoop-2.7.3.tar.gz -C /opt/module/
```

- 3、配置环境变量

```bash
cd /etc/profile.d
```

在 my_env.sh 文件添加如下内容

```bash
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

重新启动配置文件

```bash
source /etc/profile
#验证是否配置完成
hadoop
```

### 2.6、Hadoop 文件目录说明
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170228553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


```js
bin: 存储hdfs、yarn、mapred 相关命令
etc: 存储重要的配置文件
sbin: 集群相关命令
share: 学习资料，有一些官方的资料
```

## 三、Hadoop集群配置

### 3.1、Hadoop运行模式

- 本地模式：单台服务器，数据存储在linux本地
- 伪分布式：单台服务器，数据存储在HDFS  
- 分布式： 多台服务器,数据存储在HDFS，多台服务器工作

### 3.2、本机模式案例--统计单词出现的个数

```bash
cd /opt/module/hadoop-2.7.3
mkdir wcinput
vi word.txt
#编辑输入若干个单词如下
# qtds  ppx  tthy
# ppx   qtds  dmxy
# qtds 
#调用hadoop 命令执行统计单词数量
#输出目录wcoutput 会自动生成，不需要创建  输入数据文件的目录 wcinput/
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount wcinput/  wcoutput
#查看输出结果
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170246338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


### 3.3、编写集群分发脚本 xsync
#### 3.3.1、scp (secure copy) 安全拷贝
（1）scp定义

​		scp 可以实现服务器与服务器之间的数据拷贝

 （2）基本语法

​     **scp  -r   要拷贝的文件路径/名称  目的地用户@主机：目的地路径/名称**

​    **-r 文件递归**

1. 在hadoop100 机器上将文件拷贝给hadoop102

   `    scp -r /opt/software/hadoop-2.7.3.tar.gz  root@hadoop102:/opt/module/`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170259832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


2. 在hadoop102机器上从hadoop100拿取文件

   `scp -r root@hadoop100:/opt/software/hadoop-2.7.3.tar.gz  /opt/software/`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170340534.png#pic_center)

3. 在hadoop102机器上从hadoop100机器传输文件到hadoop103机器

   `scp -r root@hadoop100:/opt/software/*  root@hadoop103:/opt/software/`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170359840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


#### 3.3.2、rsync  远程同步工具

   rsync主要用于**备份和镜像**。具有**速度快、避免复制相同内容**和支持符号链接的优点。

   rsync和scp区别：用rsync做文件的复制要比scp速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。

   （1）rsync  基本语法

    **rsync  -av   要拷贝的文件路径/名称  目的地用户@主机：目的地路径/名称**
    
     **-a 归档拷贝**
    
    **-v 显示复制过程**

   如果没有rsync 命令，则需下载 yum -y install rsync

   1. 在hadoop102删除部分文件

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170412814.png#pic_center)


   2. 从hadoop100机器同步Hadoop-2.7.3文件到hadoop102机器

   `rsync -a root@hadoop100:/opt/module/hadoop-2.7.3/*  /opt/module/hadoop-2.7.3/`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170428106.png#pic_center)
#### 3.3.3、 xsync 集群分发脚本

   需求：循环复制文件到所有节点的目录下

   需求分析：

   1. rsync 原始拷贝文件

   2. 脚本要同步文件名

   3. 脚本要在任何地方都能使用（脚本放在了声明全局变量的路径）

      在 /etc/profile.d/my_env.sh  编辑添加脚本的全局路径，编辑好后，执行 **source /etc/profile**  命令

      ```bash
      #XSYNC_HOME
      export XSYNC_HOME=/root
      export PATH=$PATH:$XSYNC_HOME/bin
      ```

   实现脚本xsync：

   ```bash
   #!/bin/bash
   #1. 判断参数个数
   if [ $# -lt 1 ]
   then
    echo Not Enough Arguement!
    exit;
   fi
   #2. 遍历集群所有机器
   for host in hadoop102 hadoop103 hadoop104
   do
    	echo ==================== $host ====================
    	#3. 遍历所有目录，挨个发送
    	for file in $@
    	do
    	    #4. 判断文件是否存在
    		if [ -e $file ]
    		then
    			#5. 获取父目录
    			pdir=$(cd -P $(dirname $file); pwd)
    			#6. 获取当前文件的名称
    			fname=$(basename $file)
    			ssh $host "mkdir -p $pdir"
    			rsync -av $pdir/$fname $host:$pdir
    		else
    			echo $file does not exists!
    		fi
    	done
   done
   ```

   示例，将hadoop100机器 /etc/profile.d/  下的文件进行同步到hadoop102上

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170455907.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


### 3.4、SSH免密登录

#### 3.4.1、原理

   1. 有两台服务器A和B ，在A服务器上创建一对密钥（**公钥和私钥**），注意：私钥始终掌握在本机手中，不能传给其他服务器，否则会将服务器完全暴露给其他机器
   2. 把**公钥**拷贝给服务器B，服务器B将公钥放在**授权文件**里
   3. 当服务器A **ssh** 访问服务器B时(**数据用私钥加密**)
   4. 服务器B 接收到数据后，去**授权文件**中查找服务器A的**公钥**，并**解密数据**
   5. 服务器B用服务器A的**公钥加密数据**返回给服务器A
   6. 用服务A**公钥加密的数据**，只能通过服务器A的**私钥解密**
#### 3.4.2 、举例

   1. 在hadoop102机器上生成**密钥对**

      ```bash
      #切换到用户目录下的.ssh目录
      #如果没有.ssh目录，则需要用ssh 命令登入一下其他机器
      cd /root/.ssh/
      #生成密钥对
      ssh-keygen -t rsa
      #然后敲（三个回车），就会生成两个文件 id_rsa（私钥）、id_rsa.pub（公钥）
      ```

   2. 将hadoop102机器的**公钥** 拷贝给hadoop100

      **ssh-copy-id hadoop100**

   3. 在hadoop102就可以免密登入hadoop100了
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041217051936.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


 ### 3.5、集群配置

#### 3.5.1、配置方案

|      | hadoop100               | Hadoop102                        | hadoop103          |
| ---- | ----------------------- | -------------------------------- | ------------------ |
| HDFS | ==NameNode==   DataNode | DataNode                         | ==2NN==   DataNode |
| YARN | NodeManager             | ==ResourceManger==   NodeManager | NodeManager        |

​      备注：1、NameNode 和 2NN ，不要安装在同一台服务器

​                  2、ResourceManager**也**很消耗内存，不要和NameNode 和 2NN 配置在同一服务器

#### 3.5.2、配置文件说明

​			Hadoop 配置文件分两类，默认配置文件和自定义配置文件，==只有用户想修改某一默认配置时才需要修改自定义配置文件==

1. 默认配置文件

    **/opt/module/hadoop-2.7.3/share/hadoop**  目录下	 

   | 要获取的默认文件   | 文件存放在 Hadoop 的 jar 包中的位置                       |
   | ------------------ | --------------------------------------------------------- |
   | core-default.xml   | hadoop-common-2.7.3.jar/core-default.xml                  |
   | hdfs-default.xml   | hadoop-hdfs-2.7.3.jar/hdfs-default.xml                    |
   | yarn-default.xml   | hadoop-yarn-common-2.7.3.jar/yarn-default.xml             |
   | mapred-default.xml | hadoop-mapreduce-client-core-2.7.3.jar/mapred-default.xml |

2. 自定义配置文件

   **/opt/module/hadoop-2.7.3/etc/hadoop** 目录下

   **core-site.xml**、**hdfs-site.xml**、**yarn-site.xml**、**mapred-site.xml** 四个配置文件

#### 3.5.3、配置集群

1. 配置 core-site.xml，添加如下内容

   `vi /opt/module/hadoop-2.7.3/etc/hadoop/core-site.xml`

   ```xml
   <configuration>
    	<!-- 指定 NameNode 的地址 -->
    	<property>
    		<name>fs.defaultFS</name>
    		<value>hdfs://hadoop100:8020</value>
    	</property>
   	 <!-- 指定 hadoop 数据的存储目录 -->
    	<property>
   		 <name>hadoop.tmp.dir</name>
   		 <value>/opt/module/hadoop-2.7.3/data</value>
    	</property>
   </configuration>
   ```

2.  配置hdfs-site.xml，添加如下内容

   `vi /opt/module/hadoop-2.7.3/etc/hadoop/hdfs-site.xml`

   ```xml
   <configuration>
   	<!-- nn web 端访问地址-->
   	<property>
    		<name>dfs.namenode.http-address</name>
    		<value>hadoop100:9870</value>
    	</property>
   	<!-- 2nn web 端访问地址-->
    	<property>
    		<name>dfs.namenode.secondary.http-address</name>
    		<value>hadoop103:9868</value>
    	</property>
   </configuration>
   ```

3. 配置 yarn-site.xml，添加如下内容

   `vi /opt/module/hadoop-2.7.3/etc/hadoop/yarn-site.xml`

   ```xml
   <configuration>
    	<!-- 指定 MR 走 shuffle -->
    	<property>
    		<name>yarn.nodemanager.aux-services</name>
    		<value>mapreduce_shuffle</value>
    	</property>
    	<!-- 指定 ResourceManager 的地址-->
    	<property>
    		<name>yarn.resourcemanager.hostname</name>
    		<value>hadoop102</value>
    	</property>
    	<!-- 环境变量的继承 -->
    	<property>
    	<name>yarn.nodemanager.env-whitelist</name>
   		<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CO
   		NF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAP
   		RED_HOME</value>
   	 </property>
   </configuration>
   ```

4. 配置 mapred-site.xml，添加如下内容

   ```xml
   <configuration>
   	<!-- 指定 MapReduce 程序运行在 Yarn 上 -->
    	<property>
    		<name>mapreduce.framework.name</name>
    		<value>yarn</value>
    	</property>
       <property>
       	<name>yarn.app.mapreduce.am.env</name>
       	<value>HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.3</value>
   	</property>
   	<property>
       	<name>mapreduce.map.env</name>
       	<value>HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.3</value>
   	</property>
   	<property>
       	<name>mapreduce.reduce.env</name>
       	<value>HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.3</value>
   	</property>
   </configuration>
   ```

#### 3.5.4、同步配置文件

执行脚本 **xsync  /opt/module/hadoop-2.7.3/etc/hadoop/**

#### 3.5.5、启动集群 

1. 配置**workers**

     `vim  /opt/module/hadoop-2.7.3/etc/hadoop/workers`

   在该文件中增加如下内容：

   ==注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行。==

   ```bash
   hadoop100
   hadoop102
   hadoop103
   ```

   同步所有节点配置文件

   `xsync /opt/module/hadoop-2.7.3/etc/`

2. **启动集群**

   **如果集群是第一次启动**，需要在 hadoop102 节点格式化 NameNode（注意：格式化 NameNode，会产生新的集群 id，导致 NameNode 和 DataNode 的集群 id 不一致，集群找不到已往数据。如果集群在运行过程中报错，需要重新格式化 NameNode 的话，一定要先停 止 namenode 和 datanode 进程，并且要删除所有机器的 data 和 logs 目录，然后再进行格式化。）

   在hadoop100机器上格式化NameNode

   执行命令：` hdfs namenode -format` 

3. **启动 HDFS**

   在各个机器上启动HDFS执行(==在hadoop目录下执行下面命令==)

   `sbin/start-dfs.sh`

4. 在各个机器上启动 YARN (==在hadoop目录下执行下面命令==)

   `sbin/start-yarn.sh`

   执行  **jps**  查看启动的服务是否与配置方案一样

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170931979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


5. Web 端查看 HDFS 的 NameNode

   浏览器中输入：http://192.168.75.100:9870/

   **如果是核心最小版的需要下载 httpd，并且关闭防火墙**

   `yum install -y  httpd`

   在启动http服务

   `/bin/systemctl start httpd.service`

   设置http服务开机自启：

   `systemctl enable httpd.service`

   查询服务是否自启

   `systemctl is-enabled httpd.service`

   查询服务是否运行

   `systemctl status httpd.service`

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170551477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


6. Web 端查看 YARN 的 ResourceManager

  http://192.168.75.102:8088
  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412170605760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

  
#### 3.5.6、HDFS测试

1. 上传文件到集群

   ```bash
   #创建目录input
   hadoop fs -mkdir /wcinput  
   #上传文件
   fs -put /opt/module/hadoop-2.7.3/wcinput/word.txt   /input
   ```

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171000588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


2. 查看 HDFS 文件存储路径

   根据 core-site.xml 的配置 ,查看 HDFS 在磁盘存储文件内容

   ```xml
    <!-- 指定 hadoop 数据的存储目录 -->
    	<property>
   		 <name>hadoop.tmp.dir</name>
   		 <value>/opt/module/hadoop-2.7.3/data</value>
    	</property>
   ```

   在 /opt/module/hadoop-2.7.3/data 目录下 能找到对应上传的word.txt 数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171017171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


   

3. 复制的三份分别存在hadoop100、hadoop102、hadoop103,在hadoop102、hadoop103 机器相同的路径下，能找到一模一样的文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171043871.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


#### 3.5.7、YARN测试

   执行 wordcount 程序

```bash
 # 注意文件路径都是HDFS系统下的路径
 hadoop jar /opt/module/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /input /output
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171119195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171130238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


#### 3.5.8、集群崩溃解决方法

1. 杀死进程 

   ==hadoop目录下执行== sbin/stop-dfs.sh

2. 删除每个节点的 data（数据存放目录） 和 logs（日志存放目录） 文件

   ==hadoop目录下执行== rm -rf   data/   logs/

3. 格式化Namenode 节点

   ==hadoop目录下执行== hdfs namenode -format

4. 启动集群 

   ==hadoop目录下执行== sbin/start-dfs.sh

### 3.6、配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：

#### 3.6.1、**配置** mapred-site.xml

在hadoop100机器上编辑 mapred-site.xml

`vim  /opt/module/hadoop-2.7.3/etc/hadoop/mapred-site.xml`

添加如下内容

```bash
<!-- 历史服务器端地址 -->
<property>
 	<name>mapreduce.jobhistory.address</name>
 	<value>hadoop100:10020</value>
</property>
<!-- 历史服务器 web 端地址 -->
<property>
 	<name>mapreduce.jobhistory.webapp.address</name>
 	<value>hadoop100:19888</value>
</property>
```

#### 3.6.2、同步mapred-site.xml文件

`xsync /opt/module/hadoop-2.7.3/etc/hadoop/mapred-site.xml`

#### 3.6.3、启动历史服务器

本文配置在hadoop100机器上，所有在hadoop100执行

`/opt/module/hadoop-2.7.3/sbin/mr-jobhistory-daemon.sh start historyserver`

#### 3.6.4、查看历史服务器是否启动

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171146660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


##### 3.6.5、查看jobhistory

http://192.168.75.100:19888/jobhistory

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171156781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


### 3.7、配置日志的聚集

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到 HDFS 系统上。

日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。

==注意：开启日志聚集功能，需要重新启动 NodeManager 、ResourceManager 和HistoryServer。==

#### 3.7.1、**配置** yarn-site.xml

在hadoop100机器上编辑 yarn-site.xml

`vim  /opt/module/hadoop-2.7.3/etc/hadoop/yarn-site.xml`

添加如下内容

```bash
<!-- 开启日志聚集功能 -->
<property>
 	<name>yarn.log-aggregation-enable</name>
 	<value>true</value>
</property>
<!-- 设置日志聚集服务器地址 -->
<property> 
 	<name>yarn.log.server.url</name> 
 	<value>http://hadoop100:19888/jobhistory/logs</value>
</property>
<!-- 设置日志保留时间为 7 天 -->
<property>
 	<name>yarn.log-aggregation.retain-seconds</name>
 	<value>604800</value>
</property>
```

#### 3.7.2、同步mapred-site.xml文件

`xsync /opt/module/hadoop-2.7.3/etc/hadoop/yarn-site.xml`

####  3.7.3、关闭YARN和HistoryServer

```bash
jps #查看启动的服务
/opt/module/hadoop-2.7.3/sbin/stop-yarn.sh  #hadoop102
/opt/module/hadoop-2.7.3/sbin/mr-jobhistory-daemon.sh stop historyserver #hadoop100
```
#### 3.7.4、启动YARN和HistoryServer

```bash
#hadoop102(ResourceManager 配置在hadoop102)
/opt/module/hadoop-2.7.3/sbin/start-yarn.sh 
#hadoop100(historyserver配置在hadoop100)
/opt/module/hadoop-2.7.3/sbin/mr-jobhistory-daemon.sh start historyserver 
jps
```

#### 3.7.5 查看jobhistory

网页：http://192.168.75.100:19888/jobhistory

删除前面单词统计生成的目录(**输出目录不能存在，执行wordcount 程序会自行创建**)

`hadoop fs -rm -r /output`

执行WordCount 程序

`hadoop jar /opt/module/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /input /output`

通过网页查看任务执行日志

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171212200.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412171224384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


### 3.8、编写脚本启动/关闭集群

#### 3.8.1 、启动/关闭集群脚本

1. 进入 /root/bin/

2.  编辑 myhadoop.sh 脚本 

   ```
   #!/bin/bash
   if [ $# -lt 1 ]
   then
    	echo "No Args Input..."
    	exit ;
   fi
   case $1 in
   "start")
    	echo " =================== 启动 hadoop 集群 ==================="
    	echo " --------------- 启动 hdfs ---------------"
    	ssh hadoop100 "/opt/module/hadoop-2.7.3/sbin/start-dfs.sh"
    	ssh hadoop102 "/opt/module/hadoop-2.7.3/sbin/start-dfs.sh"
    	ssh hadoop103 "/opt/module/hadoop-2.7.3/sbin/start-dfs.sh"
    	echo " --------------- 启动 yarn ---------------"
    	ssh hadoop100 "/opt/module/hadoop-2.7.3/sbin/start-yarn.sh"
    	ssh hadoop102 "/opt/module/hadoop-2.7.3/sbin/start-yarn.sh"
    	ssh hadoop103 "/opt/module/hadoop-2.7.3/sbin/start-yarn.sh"
    	echo " --------------- 启动 historyserver ---------------"
    	ssh hadoop100 "/opt/module/hadoop-2.7.3/sbin/mr-jobhistory-daemon.sh start  historyserver"
   ;;
   "stop")
    	echo " =================== 关闭 hadoop 集群 ==================="
    	echo " --------------- 关闭 historyserver ---------------"
    	ssh hadoop100 "/opt/module/hadoop-2.7.3/sbin/mr-jobhistory-daemon.sh stop historyserver"
    	echo " --------------- 关闭 yarn ---------------"
    	ssh hadoop100 "/opt/module/hadoop-2.7.3/sbin/stop-yarn.sh"
    	ssh hadoop102 "/opt/module/hadoop-2.7.3/sbin/stop-yarn.sh"
    	ssh hadoop103 "/opt/module/hadoop-2.7.3/sbin/stop-yarn.sh"
    	echo " --------------- 关闭 hdfs ---------------"
    	ssh hadoop100 "/opt/module/hadoop-2.7.3/sbin/stop-dfs.sh"
    	ssh hadoop102 "/opt/module/hadoop-2.7.3/sbin/stop-dfs.sh"
    	ssh hadoop103 "/opt/module/hadoop-2.7.3/sbin/stop-dfs.sh"
   ;;
   *)
    echo "Input Args Error..."
   ;;
   esac
   ```

3.  赋予可执行权限  chmod +x  myhadoop.sh

#### 3.8.2 查看hadoop 服务状态脚本

   /root/bin/ 目录下，创建脚本  jpsall ，赋予可执行权限

   ```bash
   #!/bin/bash
   for host in hadoop100 hadoop102 hadoop103
    do
    	echo =============== $host ===============
    	ssh $host jps 
    done
   ```

将以上脚本同步到其他集群机器上 

xsync  /root/bin/

### 3.9、 两个面试题

常用端口号

| 端口名称                   | hadoop.2.x  | hadoop.3.x       |
| -------------------------- | ----------- | ---------------- |
| NameNode内部通信端口       | 8020 / 9000 | 8020 / 9000/9820 |
| NameNode HTTP UI           | 50070       | 9870             |
| MapReduce 查看执行任务端口 | 8088        | 8088             |
| 历史服务器                 | 19888       | 19888            |

常用的配置文件

| hadoop3.x     | core-site.xml   hdfs-site.xml   yarn-site.xml   mapred-site.xml   works |
| ------------- | ------------------------------------------------------------ |
| **hadoop2.x** | **core-site.xml   hdfs-site.xml   yarn-site.xml   mapred-site.xml    slaves** |

### 3.10、集群时间同步

==如果服务器在公网环境（能连接外网）==，可以不采用集群时间同步，因为服务器会定期和公网时间进行校准；如果服务器在内网环境，必须要配置集群时间同步，否则时间久了，会产生时间偏差，导致集群执行任务时间不同步。



doop103
    do
    	echo =============== $host ===============
    	ssh $host jps 
    done
   ```

将以上脚本同步到其他集群机器上 

xsync  /root/bin/

### 3.9、 两个面试题

常用端口号

| 端口名称                   | hadoop.2.x  | hadoop.3.x       |
| -------------------------- | ----------- | ---------------- |
| NameNode内部通信端口       | 8020 / 9000 | 8020 / 9000/9820 |
| NameNode HTTP UI           | 50070       | 9870             |
| MapReduce 查看执行任务端口 | 8088        | 8088             |
| 历史服务器                 | 19888       | 19888            |

常用的配置文件

| hadoop3.x     | core-site.xml   hdfs-site.xml   yarn-site.xml   mapred-site.xml   works |
| ------------- | ------------------------------------------------------------ |
| **hadoop2.x** | **core-site.xml   hdfs-site.xml   yarn-site.xml   mapred-site.xml    slaves** |

### 3.10、集群时间同步

==如果服务器在公网环境（能连接外网）==，可以不采用集群时间同步，因为服务器会定期和公网时间进行校准；如果服务器在内网环境，必须要配置集群时间同步，否则时间久了，会产生时间偏差，导致集群执行任务时间不同步。




