# sql执行流程概述


<!--more-->



## sql执行流程

### 一、sql执行流程概述

作为编程的基础，少不了和数据库打交道。一般都知道sql的基本语法，包括表查询、删除、插入、创建等语句的使用，那么从sql脚本到最终返回结果，这中间有哪些流程呢？本着好奇心，了解一下sql执行流程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f5e93d9217484c438a7d7295b2e15d7d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


根据上图，简要描述一下sql执行流程:

1、在打开客户端后，最初需要和sql服务器建立连接，账号认证和校验权限。

2、认证后，客户端发生查询sql脚本给服务器

3、服务器先检查查询缓存，如果命中了缓存，则立刻返回存储在缓存中的结果。否则进入下一阶段。

4、服务器端进行SQL解析、预处理，再由优化器生成对应的执行计划。

5、MySQL根据优化器生成的执行计划，再调用存储引擎的API来执行查询。

6、将结果返回给客户端。

### 二、mysql 组件

#### 2.1、连接器

> 连接器：Client与Server建立连接、进行鉴权、保持连接、管理连接。

1、**mysql -h ip -P port -u user -p password**，建立连接后，连接器会到权限表中查出user拥有的权限，之后的权限判断逻辑，都依赖此时读取到的权限。

TIPS：如果该连接权限变更，但不重新登录，依然使用此次登录时的Session（测试环境已验证）可以继续进行操作。

2、**mysql> SHOW PROCESSLIST;**，查看当前连接，如果连接长时间处于Sleep，到达wait_timeout，连接器会自动将连接主动断开。（Info 是正在执行的sql语句）

![img](https://img-blog.csdnimg.cn/img_convert/7adfba74ef1326ac44195920c906270b.png)

通过**mysql> SHOW GLOBAL VARIABLES LIKE 'wait_timeout';**，可以查询当前server的wait_timeout时间，默认为8小时。

![img](https://img-blog.csdnimg.cn/img_convert/8d7b243cc6caaaae5df9f8baa0deb58c.png)

如果Client在断开连接后，继续发送请求，则会收到Lost connection to MySQL serve during query返回。

3、数据库中，长连接是指连接成功后，客户端持续有请求，则一直使用同一个连接；短连接则是指每次执行完很少的几次查询就断开连接，下次查询再重新建立。

使用长连接，会遇到MySQL内存飞涨（临时内存管理是在连接对象中创建）；如果长连接过多，会导致内存占用过大，发生OOM（Out Of Memory），导致MySQL重启。

解决因为长连接导致的OOM：

- 定期断开长连接；
- 程序判断执行占用内存大的查询后，断开连接；
- \>=5.7版本，通过mysql_reset_connection重新初始化连接资源。

#### 2.2、查询缓存

 MySQL查询缓存保存查询返回的完整结构。当查询命中该缓存时，MySQL会立刻返回结果，跳过了解析、优化和执行阶段。 查询缓存系统会跟踪查询中涉及的每个表，如果这些表发生了变化，那么和这个表相关的所有缓存数据都将失效。 

MySQL将缓存存放在一个引用表中，通过一个哈希值引用，这个哈希值包括了以下因素，即查询本身、当前要查询的数据库、客户端协议的版本等一些其他可能影响返回结果的信息。 

当判断缓存是命中时，MySQL不会进行解析查询语句，而是直接使用SQL语句和客户端发送过来的其他原始信息。所以，任何字符上的不同，例如空格、注解等都会导致缓存的不命中。 当查询语句中有一些不确定的数据时，则不会被缓存。例如包含函数NOW()或者CURRENT_DATE()的查询不会缓存。包含任何用户自定义函数，存储函数，用户变量，临时表，mysql数据库中的系统表或者包含任何列级别权限的表，都不会被缓存。

 有一点需要注意，MySQL并不是会因为查询中包含一个不确定的函数而不检查查询缓存，因为检查查询缓存之前，MySQL不会解析查询语句，所以也无法知道语句中是否有不确定的函数。 事实则是，如果查询语句中包含任何的不确定的函数，那么其查询结果不会被缓存，因为查询缓存中也无法找到对应的缓存结果。

**MySQL 查询不建议使用缓存**，因为查询缓存失效在实际业务场景中可能会非常频繁，假如你对一个表更新的话，这个表上的所有的查询缓存都会被清空。对于不经常更新的数据来说，使用缓存还是可以的。

**所以，一般在大多数情况下我们都是不推荐去使用查询缓存的。**

MySQL 8.0 版本后删除了缓存的功能，官方也是认为该功能在实际的应用场景比较少，所以干脆直接删掉了。

#### 2.3、解析器

1、先通过词法分析：

从左到右一个字符、一个字符地输入，然后根据构词规则识别单词。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2265f92168a44de6a54701a44a2f2145.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


2、接下来，进行语法解析，判断输入的这个 SQL 语句是否满足 MySQL 语法.

根据MySQL 定义的语法规则，根据SQL 语句生成一个数据结构，这个数据结构我们把它叫做解析树（select_lex）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d29975953e8447b9b690d08263118b63.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


语法解析主要是对SQL语句的语法进行检查，看看其是否合乎语法规则。如果服务器进程认为SQL语句不符合语法规则的时候，就会把这个错误信息反馈给客户端。在这个语法检查的过程中，不会对SQL语句中所包含的表名、列名等等进行检查，只是检查语法。

3、接下来进行语义解析：

若SQL 语句符合语法上的定义的话，则服务器进程接下去会对语句中涉及的表、索引、视图等对象进行解析，并对照数据字典检查这些对象的名称以及相关结构，看看这些字段、表、视图等是否在数据库中。如果表名与列名不准确的话，则数据库会就会反馈错误信息给客户端。

所以，有时候我们写select语句的时候，若语法与表名或者列名同时写错的话，则系统是先提示说语法错误，等到语法完全正确后再提示说列名或表名错误。

#### 2.4、预处理器

1、即时 SQL

​       一条 SQL 在 DB 接收到最终执行完毕返回，大致的过程如下：
  　　1. 词法和语义解析；
        　　2. 优化 SQL 语句，制定执行计划；
            　　3. 执行并返回结果；


如上，一条 SQL 直接是走流程处理，一次编译，单次运行，此类普通语句被称作 Immediate Statements （即时 SQL）。

2、预处理 SQL

　　但是，绝大多数情况下，某需求某一条 SQL 语句可能会被反复调用执行，或者每次执行的时候只有个别的值不同（比如 select 的 where 子句值不同，update 的 set 子句值不同，insert 的 values 值不同）。如果每次都需要经过上面的词法语义解析、语句优化、制定执行计划等，则效率就明显不行了。

　　所谓预编译语句就是将此类 SQL 语句中的值用占位符替代，可以视为将 SQL 语句模板化或者说参数化，一般称这类语句叫Prepared Statements。

　　预编译语句的优势在于归纳为：一次编译、多次运行，省去了解析优化等过程；此外预编译语句能防止 SQL 注入。
注意：

　　虽然可能是通过预处理 SQL 的方式一定程度的提高了效率，但是对于优化而言，最优的执行计划不是光靠 SQL 语句的模板化来实现的，往往还是需要通过具体值来预估出成本代价。

#### 2.5、查询优化器

优化器的目的是按照一定原则来得到她认为的目标SQL在当前情形下最有效的执行路径,优化器的目的是为了得到目标SQL的执行计划。

传统关系型数据库里面的优化器分为CBO和RBO两种。

**RBO--- Rule_Based Potimizer 基于规则的优化器:**

RBO所用的判断规则是一组内置的规则，这些规则是硬编码在数据库的编码中的，RBO会根据这些规则去从SQL诸多的路径中来选择一条作为执行计划（比如在RBO里面，有这么一条规则：有索引使用索引。那么所有带有索引的表在任何情况下都会走索引）所以，RBO现在被很多数据库抛弃（oracle默认是CBO，但是仍然保留RBO代码，MySQL只有CBO）

**RBO最大问题在于硬编码在数据库里面的一系列固定规则，来决定执行计划。并没有考虑目标SQL中所涉及的对象的实际数量，实际数据的分布情况，这样一旦规则不适用于该SQL，那么很可能选出来的执行计划就不是最优执行计划了。**

**CBO---Cost_Based Potimizer 基于成本的优化器:**

CBO在会从目标诸多的执行路径中选择一个成本最小的执行路径来作为执行计划。这里的成本他实际代表了MySQL根据相关统计信息计算出来目标SQL对应的步骤的IO，CPU等消耗。也就是意味着数据库里的成本实际上就是对于执行目标SQL所需要IO,CPU等资源的一个估计值。而成本值是根据索引，表，行的统计信息计算出来的。(计算过程比较复杂)

#### 2.6、执行计划

查询语句后，经过sql的优化器，会产生一个执行计划。**根据MySQL执行计划的输出，分析索引使用情况、扫描的行数可以预估查询效率；进而可以重构SQL语句、调整索引，提升查询效率。**

详细请参考链接：[MySQL——执行计划](https://www.cnblogs.com/sunjingwu/p/10755823.html)

#### 2.7、查询执行引擎

开始执行的时候，首先要确认我们是否有操作这个表的权限，如果没有权限则会返回没有权限的错误

```mysql
mysql> select * from T where ID=10;

ERROR 1142 (42000): SELECT command denied to user 'b'@'localhost' for table 'T'
```

如果有权限，就打开表权限执行，打开表的时候执行器会根据表的引擎定义，去使用这个引擎提供的接口。
比如在这个例子中的表T中的ID字段时没有索引的，那么执行器的流程是这样的：

用InnoDB引擎接口去扫描这个表的第一行，判断ID是否为10，如果不是则跳过，如果是则将这行存在结果集中 调用引擎接口取下一行，重复相同的逻辑判断，直到取到这个表的最后一行

 执行器将上述便利过程的所有满足条件的行组成的记录集作为结果集返回给客户端

### 参考

[MySQL探秘(二)：SQL语句执行过程详解](https://cloud.tencent.com/developer/article/1200822)

[一条SQL语句在MySQL中执行过程全解析](https://blog.csdn.net/weter_drop/article/details/93386581)

[sql的语句执行过程 ](https://www.cnblogs.com/zzl-156783663/p/8506488.html)

[「MySQL」 - SQL查询语句执行流程](https://zhuanlan.zhihu.com/p/58291123)

[SQL执行过程详解](https://zhuanlan.zhihu.com/p/358646458)

[sql语句的执行流程](https://blog.csdn.net/weixin_30409927/article/details/109143380)

[MySQL查询语句完整语法解析](https://www.cnblogs.com/huuangrui/p/6107051.html)

[MySQL的SQL预处理(Prepared)](https://www.cnblogs.com/DataArt/p/10181361.html)

[MySQL查询语句完整语法解析](https://www.cnblogs.com/huuangrui/p/6107051.html)

[MySQL的SQL预处理(Prepared)](https://www.cnblogs.com/DataArt/p/10181361.html)

[mysql之优化器、执行计划、简单优化](https://www.cnblogs.com/lbg-database/p/10108513.html)

