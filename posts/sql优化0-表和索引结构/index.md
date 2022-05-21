# sql优化(0)-表和索引结构


<!--more-->


### 前言
{{< admonition info >}}
------

说明：这是学习《MySQL是怎样运行的 从根儿上理解MySQL》的笔记，非商业用途，仅供学习分享

------

表中的数据到底存到了哪里？以什么格式存放的？MySQL 以什么方式来访问这些数据？

MySQL 务器中负责对表中的数据进行 读取和写入工作的部分是存储引擎，而服务器又支持不同类型的存储引擎 比如 lnnoDB 、MyISAM 、MEMORY 等。不同的存储引擎一般是由不同的人为实现不同的特性而开发的，真实数据在不同存储引擎 中的存放格式一般是不同的，甚至有的存储引擎（比如 EMORY） 都不用磁盘来存储数据。也就是对于使用 MEMORY 存储引擎的表来说，关闭服务器后表中的数据就消失了。由于 lnnoDB MySQL 默认的存储引擎，也是最常用到的存储引擎。本文介绍InnoDB 存储引擎的记录存储结构，其他存储引擎可以作为参照进行学习。
{{< /admonition >}}


### 行格式

平时都是以记录为单位向表中插入数据的，这些记录在磁盘上的存放形式也被称为行格式或者记录格式。InnoDB 存储引擎有不同类型的行格式，分别是  Compact、Redundant、Dynamic和Compressed ，这些行格式在原理上大体都是相同的。 

{{< admonition >}} 
为了更好地理解，本文会从一些例子出发，请注意这些必不可少的步骤：
  
  1. 先建一个表
```mysql
//设置行格式 Compact，字符集ASCII
CREATE TABLE record_format_demo(
 	cl VARCHAR(1O) , 
 	c2 VARCHAR(1O) NOT NULL, 
 	c3 CHAR(1O) , 
 	c4 VARCHAR(1O) 
 ) CHARSET=ascii ROW_FORMAT=COMPACT
```


  2. 插入两条数据

```mysql
NSERT INTO record_format_demo
     (cl c2 , c3 , c4) 
     VALUES 
     ('aaaa' , 'bbb' , " CC' , 'd') , ('eeee' , 'fff', NULL ，NULL) ;
```
{{< /admonition >}}

最后生成的表数据如下图所示：

<img src="https://img-blog.csdnimg.cn/d6bfcbdd16f84135bb63b8bab63ad3e5.png#pic_center" width = "100%" height = "100%" />


{{< admonition info >}}

**Compact** 格式示意图如下：


![在这里插入图片描述](https://img-blog.csdnimg.cn/f351975dda3b4b758d29438023564bd4.png)

{{< /admonition >}}

#### 记录的额外信息

这部分信息是服务器为了更好地管理记录而不得不额外添加的一些信息.这些额外信息分为3个部分，分别是变长字段长度列表、 NULL 值列 和记录头信息。

##### （1）变长字段长度列表 

我们知道， MySQL 支持一些变长的数据类型，比如 VARCHAR间\VARBINARY(M)、 各种 TEXT 类型、各种 BLOB 类型。我们也可以把属于这些数据类型的列称为变长字段。变 长字段中存储多少字节的数据是不固定的，所以我们在存储真实数据的时候需要顺便把这些数据占用的字节数也存起来，这样才不至于把 MySQL 服务器搞懵。也就是说这些变长字段占用的存储空间分为两部分： 

- 真正的数据内容. 

- 该数据占用的字节数

COMPACT 行格式中，所有变长字段的真实数据占用字节数都存放在记录的开头位置，从而形成一个变长字段长度列表，各变长字段的真实数据占用的字节数**按照列的顺序逆序存放**。

**字节数设定**：如果该变长字段允许存储的最大字节数超过 255 字节，并且真实数据占用的字节数超过 127 字节，则使用2字节来表示真实数据占用的字节数，否则使用1字节。

比如：用UTF-8作为字符集，一个字符占用1~3个字节，那么 varchar(50) 允许存储的最大字节数就是 50X3 =150字节

另外需要注意的 一点是，**变长字段长度列表中只存储值为非 NULL 的列的内容长度，不存储值为 NULL 列的内容长度。**



那么表record_format_demo 列 c1、c2、c4都是varchar 类型，属于变长字段。字符集ASCII是每个字符1字节，可以得出下表：
<img src="https://img-blog.csdnimg.cn/6e0ede74716f4a46aadc712c554d3a5d.png" width = "100%" height = "100%" />

{{< admonition info >}}
对应的 **Compact** 格式值就是：

<img src="https://img-blog.csdnimg.cn/efb1dcb0a29d4bbfae28cd1e52cc7613.png" width = "100%" height = "100%" />

{{< /admonition >}}



##### （2）NULL 列表

我们知道 一条记录中的某些列可能存储 NULL 值，如果把这些 NULL 值都放到记录的真实数据中存储会很占地方，所以COMPACT 行格式把一条记录中值为 NULL 的列统一管理起来 ，存储到 NULL 值列表中.它的处理过程如下所示：

{{< admonition abstract >}}
1. 首先统计表中允许存储 NULL 的列有哪些
2. 如果表中没有允许存储 NULL 的列，则 NULL 值列表也就不存在了，否则将每个允许存储 NULL 的列对应一个 进制位，**二进制位按照列的顺序逆序排列** 。二进制值为1 时，代表该列的值为NULL;反之，代表该列的值不为NULL;
3.  MySQL 规定 NULL 值列表必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补0;
{{< /admonition >}}



根据 record_format_demo定义，c2列设置为非空，所以允许空值的列依次为 c1、c3、c4，总共3个字段，用3位二进制表示NULL列表，因为一个字节是8位，所以不足一个字节的话高位补0。
<img src="https://img-blog.csdnimg.cn/6f57875ccebe4f38805afbe4e6748892.png" width = "100%" height = "100%" />

转换成16进制，分别是 `0x00`、`0x06`,假设NULL列表后，两条记录的底层格式为如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ad3a4d7034364d1fa1cf0dc1b4408a2d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


##### （3）记录头信息

除了变长字段长度列表、 NUL 值列表之外，还有一个称之为记录头信息的部分。记录头信息由固定的5字节组成，用于描述记录的一些属性。5字节也就是40个二进制位 ，不同的位代表不同的意思，如下图所示：
<img src="https://img-blog.csdnimg.cn/ff555a12763b40a1bc697ffe2b8a3a11.png" width = "100%" height = "100%" />

>**这些记录头信息与数据页、索引有关，这里暂且不细讲，先了解，后续的篇幅将会展开**

#### 记录真实数据

对于record_format_demo表来说，记录的真实数据除了 c1 、c2、 c3 、 c4 ，这几个我们自己定义的列的数据外. **MySQL 会为每个记录默认地添加一些列**(也称为隐藏列)。具体的列如表
<img src="https://img-blog.csdnimg.cn/547d10efb4cd4b38a5659bb42bd2069d.png" width = "100%" height = "100%" />

{{< admonition >}}
拓展：InooDB 表的主键生成策略 

1. 优先使用用户自定义的主键作为主键
2. 如果用户没有定义主键，则选取一个不允许存储 NULL 值的UNIQUE键作为主键
3. 如果表中连非空唯一约束的键都没有定义，则InooDB会为表默认添加一个名为 row_id 隐藏列作为主键
{{< /admonition >}}

因为表record_format_demo 没有设置主键，索引mysql会自动生成一个主键，并且将字符串按ASCII字符集存储对应的十六进制值。像char、int、float等固定存储长度的数据类型，其如果是null值，一般都会用全是0的二进制保存。

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe309e35941c41abb1b6ef25ee921f78.png)

{{< admonition >}}
拓展：对char(M)列存储格式

在COMPACT 行格式里，char(M) 是否属于变长字段取决于表使用的字符集。如果字符集中所有字符都是固定长度（比如ASCII，字符都是占一个字节），那么char(M)就属于固定长度字段；如果字符集中字符所占用的字节数据是变动的(比如gbk,一个字符占用1~2个字节，utf8，一个字符占用1~3个字节)，那么char(M)就属于变长字段。一般如果定义char(M),则MySQL至少会分配M个字节来存储char(M)数据类型的值。
{{< /admonition >}}

### 数据页

#### 数据页概览

页是 InnoDB 管理存储空间的基本单位， 一个页的大小一般是16KB。InnoDB 为了不同的目的而设计了多种不同类型的页，比如存放表空间头部信息的页、存放 Change Buffer 信息的页、存放的INODE 信息的页、存放 undo 日志信息的页; 现在要说的是那些存放表中记录的那种类型的页，官方称这种存放记录的页为索引 (INDEX）页。 鉴于还没有介绍过索引是什么，而这些表中的记录就是我们日常所称的数据，所以目前还是将这种存放记录的页称为数据页。

![在这里插入图片描述](https://img-blog.csdnimg.cn/075118ad68e84fbb9d56597c25a3454a.png)


一个 InnoDB 数据 的存储空间大致被划分成 7个部分，有的部分占用的字节数是确定的，有的部分占用的字节数是不确定的。 大致描述如下图：

<img src="https://img-blog.csdnimg.cn/5613061ff44749ce972eab1e6f96a589.png" width = "100%" height = "100%" />





####  Infimum和Supremum 

我们向表中插入的记录从本质上来说都是放到数据页的 User Records 部分，这些记录一条条亲密无间地排列着，把记录一条一条亲密无间排列的结构称之为堆 (heap)。

<img src="https://img-blog.csdnimg.cn/488c95317faa43b88a2fbc179289db22.png" width = "100%" height = "100%" />




InnoDB会自动给每个页里面加了两条记录，由于这两条记录并不是用户自己插入的，所以有时候也称为伪记录或者虚拟记录。在这两条伪记录中，一条代表页面中的最小记录(也可以写作 Infimum 记录) ，另外一条代表页面中的最大记录（也可以写作 Supremum 记录)。这两条伪记录也算作堆的一部分。

<img src="https://img-blog.csdnimg.cn/7b83a7d2f6e04e6ea144335daeef9547.png" width = "100%" height = "100%" />


Infimum和Supremum 这两条记录的构造十分简单，都是由5字节大小的记录头信息和 8字节大小的一个固定单词组成。

<img src="https://img-blog.csdnimg.cn/53d701aaef8846d099bb61daf022a43e.png" width = "100%" height = "100%" />


{{< admonition info >}}
记录也可以比大小。对于一条完整的记录来说，比较记录的大小就是比较主键的大小。**InnoDB 规定任何用户记录都比Infimum 记录大 ，任何用户记录都比 Supremum记录小 。**
{{< /admonition >}}


#### User Records（记录在页中的存储）

在页的7个组成部分中，存储的记录会按照指定的行格式存储到User Records 部分。但是在一开始生成页的时候，其实并没有 User Records 部分，每当插入一条记录时 都会从 Free Space部分(也就是尚未使用的存储空间〉申请一个记录大小的空间，并将这个空间划分到 User Records 部分。当 Free Space 部分的空间全部被 User Records 部分替代掉之后，也就意味着这个页使用完了，此时如果还有新的记录插入，就需要去申请新的页了 。

{{< admonition example >}}

那么多条记录在页中具体是怎么存储的呢？现在就以一个例子展开：

创建一个表 page_demo，主键为c1 ,字符集是ASCII

```mysql
CREATE TABLE page_demo( 
	 cl INT, 
	 c2 INT, 
	 c3 VARCHAR(lOOOO) , 
	 PRIMARY KEY (cl) 
-> ) CHARSET=ascii ROW_FORMAT=COMPACT; 
```

然后插入4条数据

```mysql
 INSERT INTO page_demo 
  VALUES
    (l , 100 , 'aaaa') , 
    (2 , 200 , 'bbbb') , 
    (3 , 300 , 'cccc') , 
    (4 , 400 , 'dddd') ;
```
那么4条数据在页中存储如下图（之战时部分记录头信息和列值，剩余的信息都归为其他信息）：

<img src="https://img-blog.csdnimg.cn/2cb4b7a9f4ab47e3a94c71dacbc3f96a.png" width = "100%" height = "100%" />
{{< /admonition >}}

我们先看一下头信息的详细信息：

<img src="https://img-blog.csdnimg.cn/fc6dbfeb71324534bc81cbe53a036e52.png" width = "100%" height = "100%" />


{{< admonition info >}}

 **deleted_flag** ：这个属性用来标记当前记录是否被删除，占用1比特。值为0时表示记录没有被删除，值为1 表示记录被删除了。被删除的记录还在页中么？是的。这些被删除的记录之所以不从磁盘上移除，是因为在移除它们之后 还需要在磁盘上重新排列其他的记录，这会带来性能消耗，所以只打一个删除标记就可以避免这个问题。所有被删除掉的记录会组成一个垃圾链表，记录在这个链表中占用的空间称为可重用空间(关于链表是怎么形成的，在介绍过 next record 属性后大家就知道了)。之后若有新记录插入到表中，它们就可能覆盖掉被删除的这些记录占用的存储空间。



**min_rec_flag**：B+树每层非叶子节点中的最小的目录项记录都会添加该标记。MySQL一般把真实数据记录放在B+树的叶子节点，所以数据记录这个位值一般为0。



**n_owned**：这个放到下面的 Page Directory  篇幅来讲。



**heap_no**： 我们向表中插入的记录从本质上来说都是放到数据页的 User Records 部分，这些记录一条条亲密无间地排列着，把记录一条一条亲密无间排列的结构称之为堆 (heap)。为了方便管理这个堆，他们把一条记录(这条记录的 deleted_flag可以为 1)在堆中的相对位置称之为heap_no。 在页面前边的记录 heap_no相对较小，在页面后边的记录 heap_no相对较大，每次新申请一条记录的存储空间时，该条记录比物理位置在它前边的那条记录的 heap_no 值大1。Infimum记录和Supremum 记录的 heap_no值分别是0和1，也就是说它们在堆中的相对位置最靠前。注意的一点是，**堆中记录的heap_no值在分配之后就不会发生改动了，即使之后删除了堆中的某条记录，这条被删除记录的 heap_no 值也仍然保持不变。**

<img src="https://img-blog.csdnimg.cn/b095492cc6494534bc1f4cd9c3017104.png" width = "30%" height = "30%" />


**record_type**： 这个属性表示当前记录的类型。一共有4类型的记录。0表示普通记录 ，1表示B+ 树非叶子节点的目录项记录，2 表示 lnfimum 记录，3 表示 Supremum记录。



**next_record**：这个属性非常重要 它表示从当前记录的**真实数据**到下一条记录的**真实数据**的距离。

- 如果该属性值为正数，说明当前记录的下一条记录在当前记录的后面。

- 如果该属性值为负数，说明当前记录的下一条记录在当前记录的前面。

{{< /admonition >}}

前面插入的4条数据记录，每一条记录都是32个字节大小的空间。按照前面信息，分析如下：
<img src="https://img-blog.csdnimg.cn/15ba3d0ab5d34d5b902afb691c565893.png" width = "100%" height = "100%" />




- Infimum和Supremum 都是由5字节大小的记录头信息和 8字节大小的一个固定单词组成。

- 第 1条记录的 next_record 值为 32， 意味着从第1条记录的真实数据的地址处向后找 28字节便是下一条记录的真实数据（从infimum记录的真实值占 8字节，Sepremum记录占13字节，第一条记录的变长字段列表占1字节，NULL值列表占1字节，记录头信息占5字节，所以是8+13+1+1+5=28字节）。

- 第4条记录的 next_record 值为 -111，意味着从4条记录的真实数据的地址处向前找 109字节便是下一条记录的真实数据。（从第4条记录的额外信息占用7字节，第1条记录、第2条记录、第3条记录，每条记录占用32字节，Sepremum记录的真实数据占用8字节 ，所以是 7+32X3+8=111字节）。

这其实就是个链表，可以通过一条记录找到它的下一条记录。但是需要注意的一点是 下一条记录指的并不是插入顺序中的下一条记录，**而是按照主键值由小到大的顺序排列的下一条记录**。而且规定 Infimum 记录的下一条记录就是本页中主键值最小的用户记录，本页中主键值最大的用户记录的下一条记录就是 Supremum 记录。
<img src="https://img-blog.csdnimg.cn/022898e9cb7b43d5acff2d8bb812f8d6.png" width = "100%" height = "100%" />


{{< admonition question >}}
**next_record 为什么要指向记录头信息和真实数据之间的位置？**
- 为什么不指向整条记录的开头位置 ，也就是记录的额外信息开头的位置呢？原因是这个位置刚刚好，向左读取就是记录头信息，向右读取就是真实数据。前面还说过变长字段列表、 NULL值列表中的信息都是逆序存放的， 这样可以使记录中位置靠前的字段段和它对应的字段长度信息在内存中的距离更近，这可能会提高高速缓存的命中率。

**如果把page_demo中的第2条记录删除，那么对应的链表又是怎么样的呢？**

```mysql
DELETE FROM page_demo WHERE cl = 2;
```
删除第2条记录后，发送如下变化：

- 第2条记录并没有从存 空间中移除，而是将该条记录的deleted_flag 设置为1 ;  
- 第2条记录的 next_record 值变为0,  意味着该记录没有下一条记录了
- 第1条记录的next_record 指向了第 3条记录
- Supremum 记录的n_owned值从5变成4

<img src="https://img-blog.csdnimg.cn/9bc04662d14e4f398db73c54091fd6a6.png" width = "100%" height = "100%" />

主键值2的记录被删掉了，但是没有回收存储空间(该记录的heap_no 也未发生改变) 。再次条把这条记录插入到表中 ，则这条新插入的记录会复用被删除记录的存储空间。

```mysql
INSERT INTO page_demo 
  VALUES
    (2 , 200 , 'bbbb') ;
```
<img src="https://img-blog.csdnimg.cn/a9fa51096d2f4605bba5d15e4011c2db.png" width = "100%" height = "100%" />

{{< /admonition >}}



####  Page Directory (页目录) 

我们平时在一本书中查找某个内容的时候，一般会先看目录，找到该内容对应的图书页码，然后再到对应的页码去查看内容。InnoDB也为记录制作了 一个类似的目录，制作过程如下所示： 

1. **将所有正常的记录**（包括 Infimum Supremum 记录，但不包括已经标记为删除的记录）**划分为几个组**。
2. **每个组的最后一条记录**（也就是组内主键值最大的那条记录）**相当于"带头大哥"，组内其余的记录相当于"小弟 ”。“带头大哥”记录的头信息中的n_owned 属性表示该组内共有几条记录 。**
3.  **将每个组中最后一条记录在页面中的地址偏移量**（就是该记录的真实数据与页面中第0个字节之间的距离）**单独提取出来，按顺序存储到靠近页尾部的地方。这个地方就是 Page Directory 。页目录中的这些地址偏移量称为槽 (Slot) ，每个槽占用 字节。页目录就是由多个槽组成的。**

{{< admonition info >}}
一个正常的页面也就是 16KB 大小，即 16384 字节，而 字节可以表示的地址偏移量范围是 0-65535， 所以 用2字节表示一个槽足够了。
{{< /admonition >}}



InnoDB 对每个分组中的记录条数是有规定的:

- **对于Infimum 记录所在的分组只能有1条记录；** 
- **Supremum 记录所在的分组拥有的记录条数只能在1~8条之间；**
- **剩下的分组中记录的条数范围只能是在4~8条之间；**

所以给记录进行分组是按照下面的步骤进行的： 

1. **在初始情况下，一个数据页中只有 Infimum 记录和 Supremum 记录这两条，它们分属于两个分组。页目录中也只有两个槽，分别代表 lnfimum 记录和 Supremum 记录在页面中的地址偏移量。**
2. **之后每插入一条记录 ，都会从页目录中找到对应记录的主键值比待插入记录的主键值大并且差值最小的槽（从本质上来说，槽是一个组内最大的那条记录在页面中的地址偏移量，通过槽可以快速找到对应的记录的主键值）。然后把该槽对应的记录的n_owned值加1，表示本组内又添加了 一条记录，直到该组中的记录数等于8个。**
3. **当一个组中的记录数等于8后 ，再插入一条记录，会将组中的记录拆分成两个组，其中一个组中4条记录，另一个5条记录。这个拆分过程会在页目录中新增一个槽，记录这个新增分组中最大的那条记录的偏移量。**


{{< admonition example >}}

在 page_demo 表中正常的记录共有6条。InnoDB会把它们分成2个组，第一组只有一个 Infimum 记录，第二组是剩余的5条记录，5条记录中，规定Supremum记录主键值最大。2个组就对应着 2个槽，每个槽中存放每个组中最大的那条记录在页面中的地址偏移量。（关于Infimum记录的地址偏移量：File Header 占用38字节，page Header占用56字节，Infimum记录的记录头信息占5字节，即 38+56+5=99；Supremum记录就是在Infimum记录的地址偏移量基础上，Infimum记录的真实数据占用8字节，Supremum记录的记录头信息占5字节，即 99+8+5=112;）

<img src="https://img-blog.csdnimg.cn/c5f50d9834e045feb6dcca6f8ccbd489.png" width = "100%" height = "100%" />

在 page_demo 表中的记录太少，无法演示在添加页目录之后是如何加快查找速度的，所 以我们再 page_demo 添加一些记录

```mysql
INSERT INTO page_demo VALUES 
(5,500,'eeee'), (6,600,'ffff'), (7,700,'gggg') ,
(8,800, 'hhhh'),(9,900, 'iiii'), (10,1000,'jjjj') ,
(11,1100,'kkkk'),(12,1200,'llll'),(13,1300, 'mmmm')，
(14,1400,'nnnn'), (15, 1500,'0000') , (l6, 1600, 'pppp');
```
生成的记录表数据如下：

<img src="https://img-blog.csdnimg.cn/e48e3765e6144a65b2079b701fdecc66.png" width = "50%" height = "20%" />

当插入c1值为7的记录时，Supremum记录所在组已有8个记录，如下图所示：

<img src="https://img-blog.csdnimg.cn/079bfbf2095e412ba0478386bfc14195.png" width = "100%" height = "100%" />

当插入c1值为8的记录时，Supremum记录所在组就需要拆分成两个组，其中一个组中4条记录，另一个5条记录。

<img src="https://img-blog.csdnimg.cn/de6b388e0a8041d29b0fa53483191360.png" width = "100%" height = "100%" />

以此类推，可以得到下图的结果：

<img src="https://img-blog.csdnimg.cn/1619aba401434e6e9b76c9bef3b205d1.png" width = "100%" height = "100%" />

{{< /admonition >}}

{{< admonition question >}}
**如何找主键值为6的记录，过程是这样的：**

1. 计算中间槽的位置：(0+4)/2=2， 查看槽2对应记录的主键值为8 ；又因为 8> 6，所以设置 high=2，low 保持不变。
2. 重新计算中间槽的位置：(0+2)/2=1，查看槽1对应记录的主键值为4；又因为 4< 6，所以设置low=1，hight保持不变。
3. 因为 high-low的值为 1， 所以确定主键值为6的记录在槽2对应的组中。此时需要找到槽2 所在分组中主键值最小的那条记录，然后沿着单向链表遍历槽2中的记录。但是每个槽对应的记录都是该组中主键值最大的记录，这里槽2对应的记 录是主键值为8的记录，怎么定位一个组中最小的记录呢？别忘了各个槽都是挨着的， 我们可以很轻易地找到槽 1对应的记录(主键值为4 )，这条记录的下一条记录就是槽 2所在分组中主键值最小的记录，其主键值为 5。所以，可以从这条主键值为 记录出发，遍历槽2中的各条记录，直到找到主键值为6的那条记录即可.由于一个组中包含的记录条数最多是 8条，所以遍历一个组中的记录的代价是很小的 。

综上所述 在一个数据页中查找指定主键值的记录时，过程分为两步：

1. 通过二分法确定该记录所在分组对应的槽，然后找到该槽所在分组中主键值最小的那条记录
2. 通过记录的 next_record 属性遍历该槽所在的组中的各个记录
{{< /admonition >}}


#### Page Header (页面头部)

l为了能得到存储在数据页中的记录的状态信息 ，比如数据页中已经存储了多少条记录、 Free Space 在页面中地址偏移量、页目录中存储了多少个槽等，特意在数据页中定义了一个名为 Page Header 的部分占用固定的56 字节，专门存储各种状态信息。

<img src="https://img-blog.csdnimg.cn/9e83f930f5d5402aba4b209ed355372e.png" width = "100%" height = "100%" />


####  File Header（文件头部）

 File Header 通用于各种类型的页，也就是说各种类型的页都会以 File Header 作为第一个组成部分，它描述了一些通用于各种页的信息，比如这个页的编号是多少，它的上一个页和下一个页是谁；File Header 部分占用固定的 38 字节

<img src="https://img-blog.csdnimg.cn/4f2ba7bf5aea420abcdaa8822ecd486c.png" width = "100%" height = "100%" />


#### File Trailer (文件尾部)

InnoDB 存储引擎会把数据存储到磁盘上，但是磁盘速度太慢，需要以页为单位把数据加载到内存中处理。如果该页中的数据在内存中被修改了，那么在修改后的某个时间还需要把数据刷新到磁盘中。但是，如果在刷新还没有结束的时候断电了该咋办?为了检测一个页是否完整(也就是在刷新时有没有发生只刷新了一部分的尴尬情况)，InnoDB 在每个页的尾部都加了一个 File Trailer ，这个部分由8字节组成，可以分成 2个小部分. 

- 前4字节代表页的校验和。这个部分与 Fi1e Header 中的校验和相对应。每当一个页在内存中发生修改时 ，在刷新之前就要把页面的校验和算出来。因为 File Header 在页面的前边，所以 File Header 中的校验和会被首先刷新到磁盘，当完全写完后，校验和也会被写到页的尾部。如果页面刷新成功，则页首和页尾的校验和应该是一致的。如果刷新了一部分后断电了 ，那 File Header 中的校验和就代表着己经修改过的页，而 File Trailer 中的校验和代表着原先的页，二者不同则意味着刷新期间发生了错误。 
- 后4字节代表页面被最后修改时对应的 LSN的后4字节，正常情况下应该与  File Header 的 FIL_PAGE_LSN的后4字节相同。这个部分也是用于校验页的完整性

### 索引

#### B+索引

在很多时候，表中存放的记录都是非常多的，需要用到好多的数据页来存储这些记录。在很多页中找记录可以分为两个步骤: 

- 定位到记录所在的页

- 从所在的页内查找相应的记录

{{< admonition example >}}

还是结合例子，展开说明

1. 先建立一个表 index_demo

```mysql
create table index_demo(
	c1 int,
    c2 int,
    c3 char(1),
    primary key(c1)
)row_format=compact;
```

先看一下index_demo表的行格式

<img src="https://img-blog.csdnimg.cn/3e8849694bad4472b0c155d270b3058f.png" width = "100%" height = "100%" />


- record_type：表示当前记录的类型， 0表示普通记录 ，1表示B+ 树非叶子节点的目录项记录，2 表示 lnfimum 记录，3 表示 Supremum记录

- next_record：记录头信息的一项属性 表示从当前记录的真实数据到下一条记录的真实数据的距离。每一条记录都有next_record，通过这个属性，可以把这些记录串联在一块，形成一个链表。

c1、c2、c3列中，c1列作为主键，其他信息暂时不管，都归入其他信息。

2. 这里做一个假设：每个数据页最多能存放3条记录（实际上个一数据页非常大，可以存放好多记录）。有了这个假设之后，向 index_demo 表插入3 条记录：

```mysql
 INSERT INTO index_demo VALUES(l, 4, 'u') , (3 , 9, 'd') , (5 , 3, 'a' );
```

每个数据页在File header（头文件）中，会在FIL_PAGE_OFFSET 属性记录每个页的专属页号。数据记录在页中的存储大致是如下（忽略很多其他信息）：

<img src="https://img-blog.csdnimg.cn/1cf6dd2bdead45e3be31bd197a39b18b.png" width = "100%" height = "100%" />



3. 继续插入很多记录，显然是一个页是放不下的，需要分散存储在多个数据页。每个页的File Header头文件中 FIL_PAGE_PREV 和 FIL_PAGE_NEXT 分别表是前一页页号和后一页页号，构成一个双向链表。就是这个双向链表将物理地址上分散的数据串联起来。规定：**下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值**，而数据页内的数据记录也是按主键值排序过的。整个表的记录就按照主键值进行从小到大排序。

<img src="https://img-blog.csdnimg.cn/72d6db23e3f9449e83049f1c01c95465.png" width = "100%" height = "100%" />


由于这些大小为 16KB的页在磁盘上可能并不挨着，如果想从这么多页中根据主键值快速定位某些记录所在的页，就需要给它们编制一个目录，每个页对应一个目录项，每个目录项包括下面两个部分 ：

- 页的用户记录中最小的主键值，用 key 来表示 

- 页号，用 page_no 表示

<img src="https://img-blog.csdnimg.cn/d56d930352c44f57b72bf014e9d25b85.png" width = "100%" height = "100%" />

InnoDB 需要一种可以灵活管理所有目录项的方式。InnoDB设计人员发现这些目录项其实与用户记录长得很像，只不过目录项中的两个列是主键和页号而己 。复用了之前存储用户记录的数据页来存储目录项。为了与用户记录进行区分 ，把这些用来表示目录项的记录称为目录项记录。通过记录头信息中的 record_ type 属性来区分一条记录是普通用户记录是目录项记录（ 0表示普通记录 ，1表示B+ 树非叶子节点的目录项记录，2 表示 lnfimum 记录，3 表示 Supremum记录）

<img src="https://img-blog.csdnimg.cn/b1809e233e5b4c498b3da40c856afda3.png" width = "100%" height = "100%" />

{{< /admonition >}}



比较一下目录项记录和普通的用户记录的不同点： 

- 目录项记录的 record_ type 值是1，普通用户记录的 record_type 值是0

- 目录项记录只有主键值和页的编号两个列，而普通用户记录的列是用户自己定义的 可能包含很多列，另外还有InnoDB 添加的隐藏列

- 只有目录项记录中最小的目录项记录的记录头信息min_rec_flag 属性值是1 ，普通用户记录的 min_rec_flag属性值都是0

除了上述几点外，这两者就没啥差别了：它们用的是一样的数据页（页面类型都是0x45BF ，这个属性在 File Heade 中）；页的组成结构也是一样的〈就是前面介绍过的7部分)；都会为主键值生成 Page Directory（页目录），从而在按照主键值进行查找时可以使用二分法来加快查询速度。
![在这里插入图片描述](https://img-blog.csdnimg.cn/8fe311cd56c04148b99ec2e2d798b1d6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


一个B+树的节点其实可以分成好多层。InnoDB 规定：最下面的那层〈也就是存放用户记录的那层〉为第0层，之后层级依次往上加（在 Page header 中的 page_level属性会记录层数）。

在真实环境中 一个页存放的记录数量是非常大的。假设所有存放用户记录的叶子节点所代表的数据页可以存放 100 条用户记录(16kb，100条记录，每条记录大约160字节)，所有存放目录项记录的内节点所代表的数据页可以存放 1000 条目录项记录（16kb，1000条记录，每条记录大约16字节，除去记录头信息5字节，页号4字节，大约有7字节存储主键值），那么： 

- 如果B+树只有1层 ，也就是只有1 个用于存放用户记录的节点，则最多能存放 100 条用户记录 
- 如果B+树有2层，最多能存放 1,000 x 100 = 100,000 条用户记录
- 如果 B+树有3层，最多能存放 1,000 x 1,000 x 100 = 100,000,000 条用户记录
-  如果B+树有4层，最多能存放 1,000 x 1,000 x 1,000 x 100 = 100,000,000,000 条用户记录

在一般情况下，我们用到的 B+ 树都不会超过4层。这样一来，在通过主键值去查找某条记录时 最多只需要进行4个页面内的查找（查找3个存储目录项记录的页和1个存储用户记录的页)。又因为在每个页面内存在 Page Directory (页目录) ，所以在页面内也可以通过二分法快速定位记录。

####  聚簇索引和二级索引

**聚簇索引的特点：**

1、使用记录主键值的大小进行记录和页的排序，这包括3方面的含义：

- 页〈包括叶子节点和内节点〉内的记录按照主键的大小顺序排成一个单向链表，页内的记录被划分成若干个组，每个组中主键值最大的记录在页内的偏移量会被当作槽依次存放在页目录中 ，可以在页目录中通过二分法快速定位到主键列等于某个值的记录
- 各个存放用户记录的页也是根据页中用户记录的主键大小顺序排成一个双向链表 
- 存放目录项记录的页分为不同的层级，在同一层级中的页也是根据页中目录项记录的主键大小顺序排成个双向链表

2、 B+ 树的叶子节点存储的是完整的用户记录。所谓完整的用户记录，就是这个记录中存储了所有列的值(包括隐藏列)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5d13681ead534463a852e1ea85ffc5c8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

所有完整的用户记录都存放在这个聚簇索引的叶子节点处。这种聚簇索引并不需要我们在 SQL 语句中显式地去创建， InnoDB 存储引擎会自动创建聚簇索引。在 InnoDB 存储引擎中, 聚簇索引就是数据存储方式（所有的用户记录都存储在了叶子节点) ，也就是所谓的"索引即数据，数据即索引”。

{{< admonition question >}}

聚簇索引只能在搜索条件是主键值时才能发挥作用，原因是B+中的数据都是按照主键进行排序的。如果以别的列作为搜索条件该咋办呢?

比如为 c2 建立索引， 用 c2 列的大小作为数据页、页中记录的排序规则，然后再建一棵B+树 如图 ：

![在这里插入图片描述](https://img-blog.csdnimg.cn/d01dbd3648d94870a84e79ba0b2e80c2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
{{< /admonition >}}

**二级索引的特点：**

1、使用记录 索引列的大小迸行记录和页的排序，这包括3方面的含义：

- 页（包括叶子节点和内节点）内的记录是按照索引列的大小顺序排成一个单向链表，页内的记录被划分成若干个组，每个索引列（c2）值最大的记录在页内的偏移量会被当作槽依次存放在页目录中，可以在页目录中通过二分法快速定位到索引列（c2）等于某个值的记录
- 各个存放用户记录的页也是根据页中记录的索引列（c2）大小顺序排成一个双向链表
- 存放目录项记录的页分为不同的层级，在同一层级中的页也是根据页中目录项记录索引列（c2）大小顺序排成一个双向链表

2、树的叶子节点存储的并不是完整的用户记录，而只是 **索引列（c2）+主键 ** 这两个列的值

3、目录项记录中不再是主键+页号的搭配而变成了**索引列（c2）+主键值+页号** 的搭配（索引列可能有大量相同的值，为了更精确的定位，就加上主键值作为索引列值相同时第二级的比较值）



如果想查找满足搜索条件 c2=4的记录，就可以使用刚刚建好的这棵 B+。需要注意一下，因为 c2 列并没有唯一位约束，也就是说满足搜索条件 c2 的记录可能有很多条，只需要在该B+树的叶子节点处定位到第一条满足搜索条件 c2=4的那条记录，然后沿着由记录组成的单向链表一直向后扫描即可。另外，各个叶子节点组成了双向链表，搜索完了本页面的记录后可以很顺利地跳到下一个页面中的第一条记录，然后继续沿着记录组成的单向链表向后扫描。

其中比较重要的细节是：在 这个 B+ 树的叶子节点处定位到第 一条符合条件的那条用户记录之后，需要根据该记录中的主键信息到聚簇索引中查找到完整的用户记录。

>**这个通过携带主键信息到聚簇索引中重新定位完整的用户记录的过程也称为回表。**

然后再返回到这棵B+ 树的叶子节点处，找到刚才定位到的符合条件的那条用户记录， 并沿着记录组成的单向链表向后继续搜索其他也满足 c2=4的记录，每找到一条的话就继续进行回表操作。重复这个过程，直到下 条记录不满足 c2斗的这个条件为止。

{{< admonition question >}}
为什么还需要一次回表操作呢？直接把完整的用户记录放到时子节点不就好了么？

如果把完整的用户记录放到叶子节点是可以不用回表，但是太占地方了。 相当于每建立一颗 B+ 树都需要把所有的用户记录复制一遍，这就太浪费存储空间了。

{{< /admonition >}}

联合索引也是二级索引，联合索引是由多个列作为索引，原理都是一样的，按照索引列的先后顺序进行排序。

#### 拓展：MylSAM 中的索引方案简介

在 lnnoDB 中索引即数据，也就是聚簇索引的那棵 B+ 树的叶子节点中已经包含了所有完整的用户记录。MyISAM 索引方案虽然也使用树形结构，但是却将索引和数据分开存储。

- **将表中的记录按照记录的插入顺序单独存储在一个文件中(称之为数据文件)。这个文件并不划分为若干个数据页 ，有多少记录就往这个文件中塞多少记录。这样一来，可以通过行号快速访问到一条记录。**

{{< admonition example >}}

MyISAM 记录也需要记录头信息来存储一些额外数据。还是以 index_dem表为例，看一下这个表在使用 MyISAM 作为存储引擎时， 它的记录如何在存储空间中表示

<img src="https://img-blog.csdnimg.cn/922a245e09184ad18d0f634ef063ec3b.png" width = "50%" height = "50%" />

{{< /admonition >}}

由于在插入数据时并没有刻意按照主键大小排序，所以不能在这些数据上使用二分法进行查找

- **使用 MyISAM 存储引擎的表会把索引信息单独存储到另外一个文件中（称为索引文件）。MyISAM 会为表的主键单独创建一个索引，只不过在索引的 子节点中存储的不是完整的用户记录，而是主键值与行号的组合。也就是先通过索引找到对应的行号，再通过行号去找对应的记录** 

这一点与 lnnoDB 是完全不相同的。在 InnoDB 存储引擎中，只需要根据主键值对聚簇索引进行一次查找就能找到对应的记录。 而在MyISAM 中却需要进行一次回表操作，这也意味着MyISAM 中建立的索引相当于全部都是二级索引 

- **也可以为其他列分别建立索引或者建立联合索引，其原理与 lnnoDB中的索引差不多，只不过在叶子节点处存储的是相应的列+行号。**

>MyISAM 的行格式有定长记录格式 Static 、交长记录格式（ Dynamic）、压缩记录格式（Compressed ）等。前面用到的 index_demo 表采用定长记录格式，也就是一条记录占用的存储空间是固定的，这样就可以使用行号轻松算出某条记录在数据文件中的地址偏移量了。但是变长记录格式就不行了， MyISAM 会直接在索引叶子节点处存储该条记录在数据文件中的地址偏移量。 此可以看出，MyISAM 的回表操作是十分快速 的，因为它是拿着地址偏移量直接到文件取数据，而 InnoDB 是通过获取主键之后再去聚簇索引中找记录，虽然说不慢， 但还是比不上直接用地址去访问

### 表空间

#### 表空间概述

表空间中的每一个页都对应着一个页号，也就是每个页的File Header（文件头部）中的 FIL_PAGE_OFFSET 属性，可以通过这个页号在表空间中快速定位到指定的页面。这个页号由4字节组成，也就是 32 位，所以一个2的32次方个页（大约40亿的数量级），如果按照页的默认大小为 16KB来算，一个表空间最多支持 64TB 的数据。

表空间中的页实在是太多了，为了更好地管理这些页面，InnoDB 的设计者提出了区 （extent） 的概念。对于 16KB 的页来说，连续的 64 页就是一个区 ，也就是说一个区默认占 1MB 空间大小。无论是系统表空间还是独立表空间 ，都可以看成是由若干个连续的区组成的，每 256 个区被划分成一组。

{{< admonition info >}}

>每向表中插入 一条记录，本质上就是向该表的聚簇索引以及所有二级索引代表的 B+ 树的节点中插入数据。而 B+ 树每一层中的页都会形成一个双向链表，如果以页为单位来分配存储空间，双向链表相邻的两个页之间的物理位置可能离得非常远。 使用 B+ 树来减少记录的扫描行数的过程是通过一些搜索条件到 B+ 树的叶子节点中定位到第一条符合该条件的记录（对于全表扫描来说就是定位到第一个叶子节点的第一条记录) 。然后沿着由记录组成的单向链表以及由数据页组成的双向链表一直向后扫描就可以了。如果双向链表中相邻的两个页的物理位置不连续，对于传统的机械硬盘来说，需要重新定位磁头位置，也就是会产生随机 I/O 。 这样会影响磁盘的性能。所以应该尽量让页面链表中相邻的页的物理位置也相邻，这样在扫描叶子节点中大量的记录时才可以使用顺序 I/O
>
>所以才引入了区 (extent)的概念。一个区就是在物理位置上连续的 64 个页(区里页面的页号都是连续的)。在表中的数据量很大时，为某个索引分配空间的时候就不再按照页为单位分配了，而是按照区为单位进行分配。甚至在表中的数据非常非常多的时候，可以一次性分配多个连续的区，虽然这可能造成一点点空间的浪费(数据不足以填充满整个区) 。但是从性能角度看，可以消除很多的随机 I/O


![在这里插入图片描述](https://img-blog.csdnimg.cn/73ac97eef3b846d4af0125ddf805fb75.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
{{< /admonition >}}

使用 B+ 树执行查询时只是在扫描叶子节点的记录 ，而如果不区分叶子节点和非叶子节点，统统把节点代表的页面放到申请到的区中，扫描效果就大打折扣了。所以，lnnoOB存储引擎对 B+ 树的叶子节点和非叶子节点进行了区别对待，也就是说叶子节点有自己独有的区，非叶子节点也有自己独有的区。存放叶子节点的区的集合就算是一个段 (segment) ，存放非叶子节点的区的集合也算是一个段。**也就是说一个索引会生成两个段：一个叶子节点段和一个非叶子节点段。**

默认情况下，一个使用 InnoOB 存储引擎的表只有一个聚簇索引个索引会生成两个段。而段是以区为单位申请存储空间的，一个区默认占用 1MB 存储空间。所以，默认情况下一个只存放了几条记录的小表也需要 2MB 的存储空间么？以后每次添加一个索引都要多申请 2MB 的存储空间么？这对于存储记录比较少的表来说简直是天大的浪费。

这个问题的症结在于一个区被整个分配给某一个段，或者说区中的所有页面都是为了存储同一个段的数据而存在的。即使段的数据填不满区中所有的页面，剩下的页面也不能挪作他用。现在为了考虑"以完整的区为单位分配给某个段时，对于数据量较小 表来说太浪费存储空间" 这种情况，提出了**碎片 ( fragment ) **的概念。也就是在一个碎片区中，并不是所有的页都是为了存储同一个段的数据而存在的，碎片区中的页可以用于不同的目的，比如有些页属于段 A，有些页属于段 B，有些页甚至不属于任何段。碎片区直属于表空间，并不属于任何一个段。所以此后为某个段分配存储空间的策略是这样的：

- 在刚开始向表中插入数据时，段是从某个碎片区以单个页面为单位来分配存储空间的
- 当某个段已经占用了 32 个碎片区页面之后，就会以完整的区为单位来分配存储空间 (原先占用的碎片区页面并不会被复制到新申请完整的区中)

**段现在不能仅定义为某些区的集合，更精确的来说应该是一些零散的页面以及一些完整的区的集合**。除了索引的叶子节点段和非叶子节点段之外，InnoOB 中还有为存储一些特殊的数据而定义的段，比如回滚段。

![在这里插入图片描述](https://img-blog.csdnimg.cn/e8475f0df13a485ea1d2539569bd5ded.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


#### 区

前面介绍表空间是由若干个区组成的，这些区大致可以分为4 种类型，这4种类型的区也可以称为区的4种**状态(Status)**：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea91f3faa7e04d2da7d479681b885a90.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_18,color_FFFFFF,t_70,g_se,x_16#pic_center)


**处于 FREE、FREE_FRAG 以及 FULL_FRAG 种状态的区都是独立的，是直属于表空间；而处于 FSEG 状态的区是附属于某个段的。**

为了方便管理这些区，InnoDB 有一个称为 XDES Entry (Extent Descriptor Entry) 的结构。**每一个区都对应着一个 XDES Entry 结构**，这个结构记录了对应的区的一些属性
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a6442db796144ada0752d6b660a809e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


- Segment ID ： 每一个段都有一个唯一的编号，用ID表示。 Segment ID 字段表示的就是该区所在的段，前提是该区已经被分配给某个段了，不然该字段的值没有意义
- List Node： 这个部分可以将若干个 XDES Entry 结构串连成一个链表。如果想定位表空间内的某一个位置，只需指定页号以及该位置在指定页号中的页内偏移量。Prev Node Page Number 和 Prev Node Offset 的组合就是指向前一个 XDES Entry 的指针，Next Node Page Number 和 Next Node Offset 的组合就是指向后一个 XDES Entry 的指针
- State ：这个字段表明区的状态.可边的值分别是 FREE、FREE_FRAG、 FULL_FRAG、FSEG (前面提过)
- Page State Bitmap，这个部分共占用16字节，也就是 128 位。一个区默认有 64 个页，这 128 位被划分为 64 个部分，每个部分有 2 位，对应区中的一个页（比如 Page State Bitmap 部分的第 1位和第 2位对应着区中的第 1个页面，第 3位和第 4位对应着区中的第 2个页面….. 127 位和 128 位对应着区中的第 64 个页面）。这 2个位中的第 1位表示对应的页是否是空闲的，第 2位还没有用到。

**区、段、碎片区、附属于段的区、 XDES Entry 结构，把事情搞得这么复杂，初心仅仅是想减少随机 I/O， 又不至于让数据量少的表浪费空间。**向表中插入数据本质上就是向表中各个索引的叶子节点段、非叶子节点段插入数据。 

> 段中插入数据时，申请新页面的过程：
>
> 当段中数据较少时，首先会查看表空间中是否有状态为 FREE_FRAG 的区(也就是查找还有空闲页面的碎片区)。如果找到了，那么从该区中取一个零散页把数据插进去 ;否则到表空间中申请一个状态为 FREE 的区(也就是空闲的区) ，把该区的状态变为 FREE_FRAG，然后从该新申请的区中取一个零散页把数据插进去。之后 ，在不同的段使用零散页的时候都从该区中取，直至该区中没有空闲页面；然后该区的状态就变成了 FULL_FRAG。

现在的问题是知道表空间中哪些区的状态是 FREE，哪些区的状态是 FREE_FRAG，哪些区的状态是 FULL_FRAG 呢？这用到了 XDES Entry 中的 List Node部分：

![在这里插入图片描述](https://img-blog.csdnimg.cn/cb0622e0f2884063a2126bbe6e607259.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


- 通过 List Node 把状态为 FREE 的区对应的 XDES Entry 结构连接成一个链表 ，这个链表称为 FREE 链表。

- 通过 List Node 把状态为 FREE_FRAG 的区对应的 XDES Entry 结构连接成一个链表，这个链表称为 

  FREE _FRAG 链表。

- 通过 List Node 把状态为 FULL_FRAG 的区对应的 XDES Entry 结构连续成一个链表，这个链表称为

  FULL _FRAG 链表。



每当想查找一个 FREE FRAG 状态的区时，就直接把 FREE_FRAG 链表的头节点拿出来，从这个节点对应的区中取一些零散页来插入数据。就修改它的 State 字段的值，然后将其从 FREE_FRAG 链表移到 FULL_FRAG链表中.同理 如果 FREE_FRAG 链表 一个节点都没有，那么就直接从 FREE 链表中取一个节点移动到 FREE_FRAG 链表，并修改该节点 STATE 字段值为 FREE_FRAG。 然后再从这个节点对应区获取零散页。

当段中 数据已经占满了 32 个零散的页后，就直接申请完整的区来插入数据。怎么知道哪些区属于哪个段呢？可以根据段号 (Segment ID ) 来建立链表 (不同的索引分别有对应的叶子节点段和非叶子节点段 )。

>FREE链表：同一个段中，所有页面都是空闲页面的区对应的 XDES Entry 结构会被加入到这个链表中。注意这与直属于表空间的 FREE 链袭区别开了，此处的 FREE 链表是附属于某个段的链表
>
>NOT_FULL链表：同一个段中，仍有空闲页面的区对应的 XDES Entry 结构会被加入到这个链表中
>
>FULL链表：同一个段中，已经没有空闲页面的区对应的 XDES Entry 结构会被加入到这个链表中

怎么找到这些链表呢 或者说怎么找到某个链表的头节点或者尾节点在表空间中的位置呢？InnoDB有一个名 List Base Node 链表基节点，这个结构中包含了链表的头节点和尾节点的指针以及这个链表中包含了多少个节点的信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/f28b7a9db5514609bc31148131ce22dd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


前面介绍的每个链表都对应这么 List Base Node 结构，其中. 

- List Length 表明该链表一共有多少个节点 

- First Node Page Nurnbe 和 First Node Offset 表明该链表的头节点在表空间中的位置

- Last Node Page Number 和Last Nod Offset 表明该链袤的尾节点在表空间中的位置 

**总结：表空间是由若干个区组成的，每个区都对应 XDES Entry ，直属表空间的区对应的 XDES Entry 结构可以分成 FREE、FREE_FRAG 和 FULL_FRAG 这3个链表。每个段可以拥有若干个区，每个 中的 区对应的 XDES Entry 结构可以构成 FREE、NOT_FULL 和  FULL 这3个链表。每个链表都对应一个 List Base Node 结构，这个结构中记录了链表的头尾节点的位置以及该链表中包含的节点数。**

#### 段

段其实不对应表空间中某一个连续的物理区域，而是一个逻辑上的概念，由若干个零散的页面以及一些完整的区组成。InnoDB 存储引擎定义了一个的 INODE Entry 结构来记录段的属性。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d061e2e06d93448f8d11169a8be4b74e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


**Segment lD** ：INODE Entry 结构对应的段的编号（lD） 

**NOT_FULL_N_USED**：在NOT_FULL链表中已经使用了多少个页面 

**3个List Base Node** ：分别为段的 FREE 链表、 NOT_FULL 链表、 FULL 链表定义了 List Base Node ，当查找某个段的某个链表的头节点和尾节点时，可以直接到这个部分找到对应链表的  List Base Node 

**Magic Number** ：用来标记这个 INODE Entry 是否已经被初始化（即把各个字段的值都填迸去了）。如果这个数字的值是 97,937,874， 表明该 INODE Entry 已经初始化，否则没有被初始化 (不用纠结值 97,937,874 有啥特殊含义 ，这是规定的) 

**Fragment Array Entry** ：段是一些零散页面和一些完整的区的集合，每个 Fragment Array Entry 结构都对应着一个零散的页面，这个结构一共4字节，表示 一个零散页面的页号。

#### 独立表空间

表空间结构如图所示，每 256 个区被划分成一组，每个组的第一个区都会存放一些特殊的页面来管理这些空间。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b0d15dc191124879bddc1ab8da1ec5c6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


##### FSP_HDR 类型

首先来看第一个组的第一个页面，也是表空间的第一个页面，页号为 0 。 这个页面的类型是 FSP_HDR 它存储了表空间的一些整体属性以及第一个组内 256 个区对应的 XDES Entry结构。一个完整 FSP_HDR 类型的页面大致由 5 部分组成
![在这里插入图片描述](https://img-blog.csdnimg.cn/0380526d740a46b3beb01c947e863ad6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


File Header 和 File Trailer 在数据页中已说明，就不重复提了，Enmpty Space 未使用，不需要了解。

**File Space Header （表空间头部）**

详细的说明如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3cc3f4af194347eebe752d285d4abf35.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


对于SEG_INODES_FULL 和 SEG_INODES_FREE 的说明：

>每个段对应的 INODE Entry 结构会集中存放到一个类型为 INODE 的页中 。如果表空间中的段特别多，则会有多个  INODE Entry 结构，此时可能一个页放不下，就需要多个的 INODE 类型的页面。这些 INODE 类型的页会构成下面两种链表
>
>SEG_INODES_FULL 链表：在该链表中，INODE类型的页面都已经被的 INODE Entry 结构填充满，没有空闲空间存放额外的 INODE Entry  
>
>SEG_INODES_FREE 链表：在该链表中 ，INODE 类型页面仍有空闲空间来存放 INODE Entry 结构

**XDES Entry**

紧挨着 File Space Header 就是 XDES Entry 部分。XDES Entry 存储在表空间的第一个页面中。一个 XDES Entry 结构的大小是40字节，由于一个页面的大小有限， 只能存放数量有限的 XDES Entry 结构，所以才把 256 区划分成一组。在每组的第一个页面中存放 256 XDES Entry 结构 



##### XDES 类型

每一个 XDES Entry 对应表空间的一个区。虽然 XDES Entry 结构只占用40 字节，但是表空间中区的数量可以不断增多。在区的数量非常多时 一个单独的页可能无法存放足够多的 XDES Entry 结构。所以把表空间的区分为若干个组，每组开头的的第一个页面记录着本组内所有的区对应 XDES Entry 结构。由于第一个组的第一个页面有些特殊(它也是整个表空间的第一个页面) ，除了记录本组中所有区对应的 XDES Entry 结构外，还记录着表空间的 一些整体属性，这个页面的类型就是 FSP_HDR 类型，整个表空间里只有一个这种类型的页面。除第一个分组以外，之后每个分组的第一个页面只需要记录本组内所有的区对应的 XDES Entry 结构 。为了与 FSP_HDR 类型进行区别， 把之后每个分组中第一个页面的类型定义为XDES ， 它的结构与  FSP_HDR  类型非常相似

![在这里插入图片描述](https://img-blog.csdnimg.cn/d83a08cecfaf4b699cad3a2f259b757c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


#####  IBUF _BITMAP 类型

向表中插入一条记录，其实本质上是向每个索引对应的 B+ 树中插入记录。该记录首先插入聚簇索引页面，然后再插入每个二级索引页面。这些页面在表空间中随机分布，将会产生大量的随机 I/O ， 严重影响性能。对于 UPDATE 和 DELETE 操作来说，也会带来许多的随机 I/O 。 InnoDB 引擎引入了一种称为 Change Buffer 的结构(本质上也是表空间中的 一颗 B+ 树，它的根节点存储在系统表空间中)。 在修改非唯一二级索引页面时 (修改唯一二级索引页面时是否利用 Change Buffer 取决于很多情况，这里就不展开讨论) ，如果该页面尚未被加载到内存中(仍在磁盘上) 。那么该修改将先被暂时缓存到 Change Buffer 中，之后服务器空闲或者其他什么原因导致对应的页面从磁盘上加载到内存中时 ，再将修改合并到对应页面。另外 在很久之前的版本中只会缓存 INSERT 操作对二级索引页面所做的修改 ，所以 Change Buffer 以前被称作 Insert Buffer. 所以在各种命名上延续了之前的叫法 ，比方说 IBUF 其实是 Insert Buffer 的缩写。

##### INODE 类型

 InnoD引擎为每个索引定义了两个段（叶子节点和非叶子节点），还有为某些特殊功能定义了特殊的段。为了方便管理，又为每个段设计了一个INODE Entry 结构 ，这个结构记录了这个段的相关属性。INODE 类型的页就是为了存储 INODE Entry 结构而存在的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f430d1d587bb442d8f5bd6b40b4a2ac9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


**List Node for INODE Page List**：如果一个表空间中存在的段超过 85 个，那么一个 INODE 类型的页面不足以存储所有的段对应的 INODE Entry 结构，所以就需要额外的 INODE 类型的页面来存储这些结构。为了方便管理这些的 INODE 类型的页面，将这些的 INODE 类型的页面串连成两个不同的链表。 

的大叔将这些的 类型的页面串连成两个不同的链表. 

• SEG_INODES_FULL 链表：在该链表中， INODE 类型的页面中己经没有空闲空间来存储额外的 INODE Entry 结构. 

• SEG_INODES_FREE 链表：在该链表中，INODE 类型的页面中还有空闲空间来存储额外的 INODE Entry 结构。

前面提到过，这两个链表的基节点就存储在 FSP_HDR 类型页面的 File Space Header 中。也就是说这两个链表的基节点的位置是固定的，从而可以轻松访问这两个链表。以后每当新创建一个段（创建索引时就会创建段）时，都会创建一个与之对应的 INODE Entry 结构。存储 INODE Entry 的过程大致如下所示： 

1. 先看看 SEG_INODES_FREE 链表是否为空。如果不为空，直接从该链表中获取一个节点，也就相当于获取到一个仍有空闲空间的 INODE  类型的页面，然后把INODE Entry 结构放到该页面中。当该页面中无剩余空间时，就把该页放到 SEG_INODES_FULL 链表
2. 如果SEG_INODES_FREE 链表为空，则需要从表空间的 FREE_FRAG 表中申请一个页面，并将该页丽的类型修改为 INODE ，把该页面放到 SEG_INODES_FREE 链表中，与此同时把 INODE Entry 结构放入该页面

##### Segment Header

前面说过，一个索引会产生两个段，分别是叶子节点段和非叶子节点段，而每个段都会对应一个INODE Entry 结构。怎么知道某个段对应哪个INODE Entry 结构呢？这个对应关系记在数据页的 Page Header 部分的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/50a808cae9e1477bb61e307d7ecf1d7d.png#pic_center)


PAGE BTR_SEG_LEAF 和 PAGE_BTR_SEG_TOP都占用10个字节，对应一个 Segment Header 的结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/acd893f94a16485db8a4460218c46b6a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riF6aOO5ZKM5pyI5piO,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


PAGE BTR_SEG_LEAF 记录着叶子节点段对应的 INODE Entry 结构的地址是哪个表空间中哪个页面的哪个偏移量； PAGE_BTR_SEG_TOP 记录着非叶子节点段对应 INODE Entry结构的地址是哪个表空间中哪个页面的哪个偏移盘。这样，索引和对应的段的关系就建立起来了。不过需要注意的一点是，因为一个索引只对应两个段，所以只需要在索引的根页面中记录这两个结构。

#### 总结

InnoDB根据不同的目的而创建不同类型的页面，这些不同类型的页面基本都有 File Header 和 File Trailer 的通用结构。表空间被划分为许多连续的区，对于大小为 16KB 的页面来说，每个区默认由 64 个页（也就是 1MB） 组成，每 256 区（也就是 25 6MB）划分为一组，每个组最开始的几个页面的类型是固定的。

段是一个逻辑上的概念，是某些零散的页面以及一些完整的区的集合。每个区都对应一个 XDES Entry 结构，这个结构中存储了一些与这个区有关的属性。这些区可以被分为下面几种类型。

- 空闲的区：这些区会被加入到 FREE 链表
- 有剩余空闲页面的碎片区：这些区会被加入到 FREE_FRAG 链表
- 没有剩余空闲页面的碎片区：这些区会被加入到 FULL_FRAG 链表
- 附属于某个段的区：每个段所属的区又会被组织成下面几种链表
  -  FREE 链表：在同一个段中，所有页面都是空闲页面的区对应的 XDES Entry 结构会被加入到这个链表
  -  NOT_FULL 链表：在同一个段中，仍有空闲页面的区对应 XDES Entry 结构会被加入到这个链表
  -  FULL 链表：在同一个段中，已经没有空闲页面的区对应的 XDES Entry 结构会被加入到这个链表



每个段都会对应一个 INODE Entry 结构，该结构中存储了一些与这个段有关的属性。表空间中第一个页面的类型为 FSP_HDR，它存储了表空间的一些整体属性以及第一个组内 256 个区对应的 XDES Entry 结构。除了表空间的第一个组以外，其余组的第一个页面的类型为 XDES ，这种页面的结构和 FSP_HDR 类型的页面对比，除了少了 File Space Header 部分之外（也就是除了少了记录表空间整体属性的部分之外），其余部分是一样的。 

每个组的第二个页面的类型为 IBUF_BITMAP，存储了一些关于 Change Buff 的信息。 

表空间中第一个分组的第三个页面的类型是 INODE ，它是为了存储INODE Entry 结构而设计的，这种类型的页面会组织成下面两个链表.

-  SEG_INODES_FULL 链表：在该链表中， INODE 类型的页面中己经没有空闲空间来存储额外的 INODE Entry 结构

- SEG_INODES_FREE 链表：在该链表中，INODE 类型的页面中还有空闲空间来存储额外的 INODE Entry 结构

Segment Header 结构占用 10 韦节，是为了定位到具体的时ODEEntry 结构而设计的。



### 参考
[MySQL原理 - InnoDB引擎 - 行记录存储 - Compact 行格式](https://zhuanlan.zhihu.com/p/152216816)



