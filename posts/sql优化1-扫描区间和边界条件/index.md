# sql优化(1)-扫描区间和边界条件


<!--more-->



## sql优化(1)-扫描区间和边界条件

对于某个查询来说，最简单粗暴的执行方案就是扫描表中的所有记录，判断每一条记录是否符合搜索条件。如果符合，就将其发送到客户，否则就跳过该记录。这种执行方案也称为全表扫描。对于使用 InnoD 存储引擎的表来说，全表扫描意味着从聚簇索引第一个叶子节点的第一条记录开始，沿着记录所在的单向链表向后扫描， 直到最后一个叶子节点的最后一条记录。虽然全表扫描是种很笨的执行方案，但却是一种万能的执行方案， 所有的查询都可以使用这种方案来执行。

还有更快的方法，利用 B+ 树查找索引列值等于某个值的记录，这样可以明显减少需要扫描的记录数量（**索引本质是已经过排序的数据结构**），只需要扫描某个区间或者某些区间中的记录也可以明显减少需要扫描的记录数量。

为方便理解，先建一个演示表

```mysql
CREATE TABLE table_demo ( 
	id INT NOT NULL AUTO_INCREMENT, 
	keyl VARCHAR(lOO) , 
	key2 INT, 
	key3 VARCHAR(100) , 
	key_partl VARCHAR(100) , 
	key_part2 VARCHAR(10Q) , 
	key_part3 VARCHAR(100) , 
	common_field VλRCHAR(lOO) , 
	PRIMARY  KEY (id) , 
    KEY idx_keyl (keyl),
    UNIQUE KEY uk_key2 (key2) , 
    key idx_key3 (key3),
    KEY idx_part(key_partl. key_part2, keY_part3) 
)
```

为这个 table_demo 表建立1个聚簇索引4个二级索引，分别是：

- 为 id 列建立的聚簇索引; 
- 为 key1 列建立的 idx_key1 二级索引
- 为 key2 列建立的 uk_key2 唯一索引
- 为 key3 列建立的 idx_key3 二级索引 
- 为key_part1、key_part2、 key_part3列建立联合索引

假设往这个表插入10万条随机数据。然后进行下面这些查询操作：

查询语句1：

```mysql
select * from table_demo where id>=2 and id<=100
```

这个语句其实是想查找 id 值在 [2,100] 区间中的所有聚簇索引记录。可以通过聚簇索引对应树快速地定位 id 值为 2的那条聚簇索引记录，然后沿着记录所在的单向链表向后扫描直到某条聚簇索引记录的 id 值不在 [2,100] 区间中为止 (即 id 值不再符合 id<= 100 条件)。把这个例子中待扫描记录的 id 值所在的区间称为**扫描区间**，把形成这个扫描区间的搜索条件(也就是 id >= 2 AND  id <=100)  称为形成这个扫描区间的**边界条件**。

>其实对于全表扫描来说 相当于扫描 值在 (-∞，+∞ ) 区间中 记录，也就是说全农扫描对应的扫描区间是 (-∞，+∞ ) 

查询语句2：

```mysql
select * from table_demo where key2 in (1438,6328) or (key2 > 38 and key2 < 79)
```

该查询的搜索条件涉及 key2 列， 又正好 key2 列建立了 uk_key2 索引 .如果使 uk_key2 索引执行这个查询，则相当于从下面的3个扫描区间获取二级索引记录。

- [1438,6328]：对应的边界条件是 key in (1438)
- [6328,6328]：对应的边界条件是 key in (6328)
- (38,79)：对应的边界条件是 key>38 and key<79 

方便起见，可以把像 [1438,1438]、[6328,6328] 这样只包含一个值的扫描区间称为**单点扫描区间**， [38,79] 这样包含多个值的拍描区间称为**范围扫描区间**。

查询语句3：

```mysql
select * from table_demo where keyl < ' a ' ANO key3 > 'z'
```

- 如果使用 idx_key1 索引执行查询，那么相应的扫描区间就是 (-∞，‘a’ )， 形成该扫描区间的边界条件就是 key1 < 'a'， key3 > 'z' 就是普通的搜索条件，这些普通的搜索条件需要在获取到 idx_key1二级索引记录后，再执行回表操作，在获取到完整的用户记录后才能去判断它们是否成立。 
- 如果使用 idx_key3 索引执行查询，那么相应的扫描区间就是 (‘z’ , +∞ )， 形成该扫描区间的边界条件就是 key3 > 'z'， key1 < 'a' 就是普通的搜索条件，这些普通的搜索条件需要在获取到 idx_key2二级索引记录后，再执行回表操作，在获取到完整的用户记录后才能去判断它们是否成立。 

所以，对于查询语句3，无论选择哪个索引，都有一部分筛选条件不能成为边界条件。

一个查询语句中的 WHERE 子句可能有很多个小的搜索条件，这些搜索条件使用 AND 或者 OR 操作符连接起来。

-  条件1 AND 条件2  只有当 条件1 和 条件2 都为 TRUE 时，整个表达式才为 TRUE 
- 条件1 OR  条件2  只要 条件1 或者 条件2 有一个为 TRUE，整个表达式就为 TRUE 

在执行一个查询语句时，首先需要找出所有可用的索引以及使用它们时对应的扫描区间。

查询语句4：

```mysql
select * from table_demo where key2>100 and key2>200;
select * from table_demo where key2>100 or key2>200;
```

对于  key2>100 、key2>200 可以用到索引uk_key2，对应的扫描区间是 (100, +∞ )、（200，+∞）。and 就是把两个扫描区间取交集，即(200,+∞)；or就是把两个扫描区间取并集，即(100,+∞ )，这个就比较好理解。

查询语句5：

```mysql
select * from table_demo where key2>100 and key1>'c';
select * from table_demo where key2>100 or key1>'c';
```

对于这个两个查询，如果使用uk_key2 索引的话，key2>100 可用形成扫描区间(100,+∞) ，key1>'c' 对于uk_key2 索引来说是没有办法按照扫描区间来查找的（因为uk_key2 索引是按照key2列的值按大小顺序进行排序的，对于key1的值来说，是无序的），就只能全部扫描，即 (-∞,+∞ )。(100,+∞ )和 (-∞,+∞ ) 的并集也还是 (-∞,+∞ )，所以，即便使用uk_key2 索引，还是需要全表扫描，并且先扫描索引，还需要通过回表操作获取key1列的值，这无疑是比直接全表扫描更耗时。

> 对于扫描区间的 (-∞,+∞ )的筛选条件可以当成逻辑判断中的状态 TRUE 
>
> 例如：
>
> select * from table_demo where key2>100 or key1>'c';
>
> 替换：
>
> select * from table_demo where key2>100 or TRUE;
>
> 即：
>
> select * from table_demo where TRUE ;
>
> 使用key2列索引形成的索引区间是 TRUE，即 (-∞,+∞ ) 全表扫描

查询语句6：

```mysql
select * from table_demo where
	(key1 > 'xyz' and key2 = 748 )  or
	(key1 < 'abc' and key1 > 'lmn') or
	(key1 like '%suf' and key1 > 'zzz' and (key2 <8000 or common_field='abc'));
```

**假设使用 idx_keyl 执行查询**

那么对于key2、common_field 的筛选条件就都是需要全表扫描的，对于key1 like '%suf' 是匹配字符末尾，也是需要全表扫描的， 即扫描范围是 (-∞,+∞ ) ,即查询语句可以替换成以下：

>select * from table_demo where
>	(key1 > 'xyz' and TRUE )  or
>	(key1 < 'abc' and key1 > 'lmn') or
>	(TRUE and key1 > 'zzz' and (TRUE orTRUE));

因为 key1 < 'abc' 和 key1 > 'lmn' 这两个条件永远无法满足，即FALSE，上述条件可用简化成：

>select * from table_demo where key1 > 'xyz' or key1 > 'zzz'

即扫描区间是（'xyz',+∞）和 （'zzz',,+∞）取并集，即扫描区间是（'xyz',+∞），需要把满足 key1 > 'xyz' 条件的所有二级索引记录都取出来，针对获取到的每一条二级索引记录，都要用它的主键值再进行回表操作， 在得到完整的用户记录之后再使用其他的搜索条件进行过滤 。

**假设使用 idx_key2 执行查询**

那么对于key1、common_field 的筛选条件就都是需要全表扫描的，即扫描范围是 (-∞,+∞ ) ,即查询语句可以替换成以下：

>select * from table_demo where
>
>​    (TRUE and key2 = 748 )  or
>​	(FALSE) or
>​	(TRUE and TRUE and (key2 <8000 or TURE));

key2 <8000 or TURE  结果 肯定是TURE，继续简化：

>select * from table_demo where key2 = 748 or TRUE
>
>即：select * from table_demo where TRUE

得出来的扫描空间是： (-∞,+∞ ) ，全表扫描。需要把所有二级索引记录都取出来，针对获取到的每一条二级索引记录，都要用它的主键值再进行回表操作， 在得到完整的用户记录之后再使用其他的搜索条件进行过滤 。

显然，对于查询语句6，使用idx_keyl索引执行查询需要扫描的记录数比idx_keyl2索引少。

