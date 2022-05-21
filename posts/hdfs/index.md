# hdfs学习文档


<!--more-->


# HDFS

## 一、HDFS概述

### 1.1、HDFS产生背景

随着数据量越来越大，在一个操作系统存不下所有的数据，那么就分配到更多的操作系统管理的磁盘中，但是不方便管理和维护，迫切需要==一种系统来管理多台机器上的文件==，这就是==分布式文件管理系统==。==HDFS 只是分布式文件管理系统中的一种。==

### 1.2、HDFS概念

HDFS（Hadoop Distributed File System），它是一个**文件系统**，用于存储文件，通过目录树来定位文件；其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

HDFS 的**使用场景**：**适合一次写入，多次读出的场景**。一个文件经过创建、写入和关闭之后就不需要改变。

### 1.3、HDFS优缺点

#### 1.3.1、HDFS优点

1. 高容错性

   数据会备份多个副本（默认备份3份），当其中一份数据副本丢失，HDFS会自动恢复备份。

2. 适合处理大数据

   ​	数据规模：能够处理数据规模达到GB、TB、甚至PB级别的数据； 

   ​    文件规模：能够处理百万规模以上的文件数量，数量相当之大。

3. ）==可构建在廉价机器上==，通过多副本机制，提高可靠性。

#### 1.3.2、HDFS缺点

1. **不适合低延时数据访问**， 像mysql快速的增删改查

2. **无法高效的对大量小文件进行存储。** 

   存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息。这样是不可取的，因为NameNode的内存总是有限的；

   小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标。 

3. **不支持并发写入、文件随机修改。** 

   一个文件只能有一个写，不允许多个线程同时写；

   **仅支持数据append（追加）**，不支持文件的随机修改。

### 1.4、HDFS组成架构

1. **NameNode（nn）**：就是Master，它是一个主管、管理者。 

   （1）管理HDFS的名称空间；

   （2）配置副本策略； 

   （3）管理数据块（Block）映射信息； 

   （4）处理客户端读写请求。 

2.   **DataNode**：就是Slave。NameNode下达命令，DataNode执行实际的操作。

   （1）存储实际的数据块；

   （2）执行数据块的读/写操作。

3. **Client**：就是客户端。 

   （1）文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传； 

   （2）与NameNode交互，获取文件的位置信息；

   （3）与DataNode交互，读取或者写入数据；

   （4）Client提供一些命令来管理HDFS，比如NameNode格式化； 

   （5）Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作； 

4. **Secondary NameNode（2nn）**：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。 

   （1）辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode ； 

   （2）在紧急情况下，可辅助恢复NameNode

### 1.5、文件块大小

HDFS中的文件在物理上是分块存储（Block），块的大小可以通过配置参数( dfs.blocksize）来规定，默认大小在**Hadoop2.x/3.x版本中是==128M==，1.x版本中是==64M==**。

**为什么块的大小不能设置太小，也不能设置太大？** 

（1）**HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置；**

​		例如：一个块1kb，一个100kb的文件需要切成100块，读取这个文件时需要找到这100个块，寻找数据块的时间大大增加

（2）**如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。导致程序在处理这块数据时，会非常慢**

​	例如：一个块1GB，一个100kb的文件存储，读取文件时需要在1GB 磁盘里读取数据（相当于读取1GB的数据量）。

## 二、HDFS的Shell操作（开发重点）

### 2.1 基本语法

- ​	hadoop  fs  具体命令   
-  （或者）hdfs   dfs  具体命令

### 2.2、常用命令

#### 2.2.1、上传

1.  -moveFromLocal：从本地剪切粘贴到 HDFS

   ```bash
   hadoop fs -moveFromLocal 本地文件路径   HDFS文件路径
   #举例
   hadoop fs -moveFromLocal ./test1.txt /output
   ```

2. -copyFromLocal:从本地文件系统中拷贝文件到 HDFS 路径去

   ```bash
   hadoop fs -copyFromLocal 本地文件路径   HDFS文件路径
   #举例
   hadoop fs -copyFromLocal ./test2.txt /output
   ```

3. -put：等同于 copyFromLocal，==生产环境更习惯用 put==

   ```bash
   hadoop fs -put 本地文件路径   HDFS文件路径
   #举例
   hadoop fs -put ./test3.txt /output
   ```

4.  -appendToFile：追加一个文件到已经存在的文件末尾

   ```bash
   hadoop fs  -appendToFile 本地文件路径   HDFS文件路径
   #举例
   hadoop fs  -appendToFile ./test4.txt /output/test3.txt 
   ```

#### 2.2.2、下载

-copyToLocal：从 HDFS 拷贝到本地

```bash
hadoop fs  -copyToLocal    HDFS文件路径     本地文件路径
#举例
hadoop fs  -copyToLocal  /output/test3.txt    ./
```

-get：等同于 copyToLocal，==生产环境更习惯用 get==

```bash
hadoop fs  -put    HDFS文件路径     本地文件路径
#举例
hadoop fs  -put  /output/test3.txt    ./
```

#### 2.2.3  HDFS直接操作

-ls: 显示目录信息

```bash
hadoop fs  -ls    HDFS文件路径 
#举例
hadoop fs  -ls    /
```

-cat：显示文件内容

```bash
hadoop fs  -cat    HDFS文件路径 
#举例
hadoop fs  -cat    /output/test3.txt
```

-chgrp、-chmod、-chown：Linux 文件系统中的用法一样，修改文件所属权限

```bash
 hadoop fs -chmod 666   /output/test3.txt
```

-mkdir：创建文件夹

```bash
hadoop fs -mkdir  /output/aaa
```

-cp：从 HDFS 的一个路径拷贝到 HDFS 的另一个路径

```bash
 hadoop fs -cp   /output/test3.txt   /tmp
```

-mv：在 HDFS 目录中移动文件

```
hadoop fs -cp   /tmp/test3.txt    /output/aaa   
```

-tail：显示一个文件的末尾 1kb 的数据

```bash
 hadoop fs -tail   /output/test3.txt
```

-rm：删除文件或文件夹

```bash
hadoop fs -rm   /tmp/test3.txt
```

-rm -r：递归删除目录及目录里面内容

```bash
hadoop fs -rm  -r /tmp
```

-du 统计文件夹的大小信息

```bash
hadoop fs -du -h /output
hadoop fs -du -s /output
```

-setrep：设置 HDFS 中文件的副本数量

```bash
 hadoop fs -setrep 10  /output/test3.txt
```

这里设置的副本数只是记录在 NameNode 的元数据中，是否真的会有这么多副本，还得看 DataNode 的数量。因为目前只有 3 台设备，最多也就 3 个副本，只有节点数的增加到 10台时，副本数才能达到 10

## 三、HDFS读写流程

### 3.1、HDFS写数据流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415101719756.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)




1. 客户端向NameNode 请求上传文件  /usr/data/data1  

2. NameNode 检查客户端有没有权限写入数据，检查目录  文件是否已经存在 ，如果有权限且文件不存在，则响应客户端可以上传文件

3. 客户端请求上传第一个block(0~128M 数据块) ，请返回DataNode

4. NameNode 通过节点距离计算和负载均衡得出存储的节点，返回存放数据的3个DataNode 节点  dn1、dn2、dn3

5. 客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。

6. dn1、dn2、dn3 逐级应答客户端,通知客户端可以传数据

7. 客户端开始往 dn1 上传第一个 Block（先从磁盘读取数据放到一个本地内存缓存），以 Packet 为单位（大小64k），dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 每传一个 packet会放入一个应答队列等待应答

8. 当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。（重复执行 3-7 步）。

### 3.2、网络拓扑-节点距离计算

在 HDFS 写数据的过程中，NameNode 会选择距离待上传数据最近距离的 DataNode 接

  收数据。那么这个最近距离怎么计算呢？

  节点距离：**两个节点到达最近的共同祖先的距离总和**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415101706562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


  理解： 节点1------机架1------集群1---数据总部 ，走过3个节点

  ​			节点2------机架2------集群2------数据总部 ， 走过3个节点，那么节点1和节点2距离为6

### 3.3、机架感知（副本存储节点选择）

-   第一个副本在Client所处的节点上。如果客户端在集群外，随机选一个。

-   第二个副本在另一个机架的随机一个节点。

-   第三个副本在第二个副本所在机架的随机节点。
### 3.4、HDFS读数据流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415101756185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


1. 客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的 DataNode 地址。
2.  挑选一台 DataNode（就近原则，然后随机）服务器，请求读取数据。
3. DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。 
4. 客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

## 三、NameNode **和** SecondaryNameNode

###  3.1、NN和2NN工作机制

思考：NameNode 中的元数据是存储在哪里的？

首先，我们做个假设，如果存储在 NameNode 节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生**在磁盘中备份元数据的FsImage**。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新 FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦 NameNode 节点断电，就会产生数据丢失。**因此，引入 Edits 文件（只进行追加操作，效率很高）。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到 Edits 中。**这样，一旦 NameNode 节点断电，可以通过 FsImage 和 Edits 的合并，合成元数据。

但是，如果长时间添加数据到 Edits 中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行 FsImage 和 Edits 的合并，如果这个操作由NameNode节点完成，又会效率过低。因此，**引入一个新的节点SecondaryNamenode，专门用于 FsImage 和 Edits 的合并。**



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415101738369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


**第一阶段： NameNode 启动**

1. 第一次启动 namenode 格式化后，创建 fsimage 和 edits （在 namenode 所在结点的 hadooop/data 目录下）文件。如果不是第一次启动，直接加载编辑日志edits_inprogress_001和镜像文件fsimage 到内存。
2.  客户端对元数据进行增删改的请求
3. namenode 记录操作日志edits_inprogress_001
4.  namenode 在内存中对数据进行删改



**第二阶段： Secondary NameNode 工作**

1.  Secondary NameNode 询问 namenode 是否需要 checkpoint 。直接带回 namenode 是否检查结果。
2.  Secondary NameNode 请求执行 checkpoint 。
3.  namenode 滚动正在写的 edits 日志（将正在使用编辑的编辑日志edits_inprogress_001改名edits_001，并使用edits_inprogress_002作为记录操作新的编辑日志）
4. 将滚动前的编辑日志edits_001和镜像文件fsimage 拷贝到 Secondary NameNode
5. Secondary NameNode 加载编辑日志和镜像文件fsimage 到内存，并合并。
6. 生成新的镜像文件 fsimage.chkpoint 
7. 拷贝 fsimage.chkpoint 到 namenode 
8.  namenode 将 fsimage.chkpoint 重新命名成 fsimage，原先的fsimage作为历史文件保留

###   3.2、Fsimage 和Edits

NameNode被格式化之后，将在/opt/module/hadoop-3.1.3/data/tmp/dfs/name/current目录中产生如下文件

```
fsimage_0000000000000000000

fsimage_0000000000000000000.md5

seen_txid

VERSION
```

（1）Fsimage文件：HDFS文件系统元数据的一个**永久性的检查点**，其中包含HDFS文件系统的所有目

​									录和文件inode的序列化信息。 

（2）Edits文件：存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先

​								会被记录到Edits文件中。 

（3）seen_txid文件保存的是一个数字，就是最后一个edits_的数字

（4）每 次NameNode启动的时候都会将Fsimage文件读入内存，加 载Edits里面的更新操作，保证内存

中的元数据信息是最新的、同步的，可以看成NameNode启动的时候就将Fsimage和Edits文件进行了合				并。 

## 四、DataNode工作机制

### 4.1、DataNode 工作机制

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415101828667.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


1. 一个数据块在 DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是**数据本身**，一个是**元数据**包括数据块的长度，块数据的校验和，以及时间戳。 

2. DataNode 启动后向 NameNode 注册，通过后，周期性（6 小时）的向 NameNode 上报所有的块信息。（向NameNode汇报文件状态有没有损毁）

   ​	DN 向 NN 汇报当前解读信息的时间间隔，默认 6 小时；

   ​	DN 扫描自己节点块信息列表的时间，默认 6 小时

3. 心跳是每 3 秒一次，心跳返回结果带有 NameNode 给该 DataNode 的命令如复制块数据到另一台机器，或删除某个数据块。如果超过 10 分钟没有收到某个 DataNode 的心跳，则认为该节点不可用。（向NameNode汇报DataNode的状态，是否挂掉，如果10分钟内没有反馈，则认为DataNode 不存在，不会再和这个DataNode通信）

### 4.2、数据完整性

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415101818848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


奇偶校验：

数据： 0100 0001  1的个数是2  是偶数  奇偶校验位为0

收到的传输数据进行奇偶校验，如果1的个数奇偶和奇偶校验位一致，则认为数据是完整的

hadoop 采用 crc（32），常见的校验算法还有 md5（128），sha1（160） 

都是对数据按一定算法得出的结果和校验位进行比较，判断数据是否完整

### 4.4、**掉线时限参数设置**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415101840208.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)


如果定义超时时间为TimeOut，则超时时长的计算公式为：

TimeOut = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval。

而默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。

hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒，dfs.heartbeat.interval 的单位为秒。

```xml
<property>
 	<name>dfs.namenode.heartbeat.recheck-interval</name>
 	<value>300000</value>
</property>
<property>
 	<name>dfs.heartbeat.interval</name>
 	<value>3</value>
</property>
```

 dfs.heartbeat.interval。

而默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。

hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒，dfs.heartbeat.interval 的单位为秒。

```xml
<property>
 	<name>dfs.namenode.heartbeat.recheck-interval</name>
 	<value>300000</value>
</property>
<property>
 	<name>dfs.heartbeat.interval</name>
 	<value>3</value>
</property>
```


