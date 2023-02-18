# mysql

端口号3306

## 数据库通用基础

- 数据库的四大范式

1. 1NF属性不可拆分、无重复的列。 如：员工拥有手机和固定电话，这时需要拆成两个列
2. 2NF有一个唯一主键，并且非主属性对候选键是完全依赖的。且第二范式必须先满足第一范式 **目的减少不确定性的依赖**
3. 3NF：消除传递依赖：数据库中的非主属性只能依赖于候选键，不能存在其他依赖
   即为，该表中不存在其他表中已经出现了的非主属性信息。**减少冗余**
   ：两个表关联，一个表只能有第二个表的一个依赖
4. 4NF：**一个表的主键只对应一个值**：比如：
   表中包含：职工、职工孩子、职工选修课，这其中职工孩子和选修课可能有多个，因此要拆分。
5. BCNF：

## 数据类型

- **char和varchar的区别**
  char是定长的，定义为100以后，一直都会占用那么多空间
  varchar是可变的，定义了255以后会根据实际存放的字段大小来确定占用空间
  char的空间利用率低，varchar利用率高
  char的效率高，varchar效率低

比如身份证号，手机号等，可以使用char类型。而不固定大小的数据用varchar更好

## 锁

- 全局锁

- 表锁

- 行锁

- 乐观锁、悲观锁

- 悲观锁是怎么实现的

- 间隙锁
  在innodb可重复读级别下解决幻读问题。

间隙锁会封锁该条记录相邻两个键之间的空白区域，防止其它事务在这个区域内插入、修改、删除数据；所谓间隙是将数据分为不同区间，对该区间范围进行加锁，区间的规则为左开右闭，比如当数据为1,3,5时，对应的区间为(-∞，1],(1,3],(3,5],(5,+∞]；

- 共享锁
  select ... lock in share mode
- 排他锁
  select for update
- 锁优化

- 意向锁 intention exclusive lock 
  S行锁：IS表锁
  X行🔒：IX表锁

## 事务

- ACID
  - Atomicity [原子性]
  - Consistency [一致性]
  - Isolation [隔离性]
  - Durability [持久性]

- 特征

- 脏读、幻读、不可重复读

- **隔离级别**

1. **读未提交(read uncommitted)**：一个事务还没提交时，它做的变更就能被别的事务看到。
2. **读已提交(read committed)**：一个事务提交之后，它做的变更才会被其他事务看到。
3. **可重复读(repeatable read)**：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务是不可见的。
4. **串行化(serializable)**：对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

- **读已提交**和**可重复读**：假设数据表T中只有一列，value=1，下面按照时间顺序执行两个事务
- 并发事务
  MVCC - 多版本并发控制
- 实现原理
  MVCC保存某个时间点上的数据快照。在一个事物内看到的是同一个版本快照


-  **mvcc能否解决幻读**
-  在**快照读**读情况下，mysql通过mvcc来避免幻读。
-  在**当前读**读情况下，mysql通过next-key来避免幻读。


## 索引

- **建立索引的sql语句：**
  create index index_name on table_name(col_name)

- **索引的类型**
  主键索引：唯一且非空
  普通索引：单列
  组合索引：多列索引，满足最左匹配原则
  唯一索引：唯一值的索引
  全文索引：myisam引擎，innodb不支持全文索引。

聚簇索引，非聚簇索引

- **索引有哪些**
  b+树索引，
  hash索引
  位图索引

- **聚簇索引**
  innodb的主键使用的就是聚簇索引，聚簇索引是指索引的键和值都存在一起的。
  也就是B+树的叶子结点中，不仅存放着索引的键，也存储着值。
  一张表中只有一个聚集索引
- **非聚簇索引**
  叶子结点中只存储了该索引的键
  比如在innodb中建立了一个索引，然后利用该索引查询，首先查询到的是该结果所在行的主键值，然后再使用查到 的主键去查询实际需要查询的值。
  会回表查询
- **所有的非聚簇索引都一定会回表吗？**
  不一定，如果查询的结果在叶子结点上存在了的话，就不需要回表。

- **最左匹配原则**

- **最左匹配原则底层原理**
  索引底层是B+树，本质是一个B+树文件。
  联合索引(a, b, c)简历的索引中，a是有序的，而bc无序，a相等则按b排，b相等则按c排。
  ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/sBkaxo.jpg)
  所以最左匹配原则遇上范围查询就会停止,
  而a>1and b=2，a字段可以匹配上索引，但b值不可以


- 前缀索引
- 如何看索引是否生效
  explain sql

### 索引失效的条件

1. 最左前缀原则
2. like ‘%'
3. 在索引列上做计算操作
4. 查询两边类型不想等



## 日志

- 慢查询日志
  mysql.general_log

- 错误日志

- redo log

- binlog

- undo log


## 引擎

- innodb

- myisam

- B+树高度问题
  一个节点为1页，一页大小是16k，
  如果key使用的是bigint，则为8字节，指针在mysql中为6字节，一共是14字节，则16k能存放 16 * 1024 / 14 = 1170 个索引指针。于是可以算出，对于一颗高度为2的B+树，根节点存储索引指针节点，那么它有1170个叶子节点存储数据，每个叶子节点可以存储16条数据，一共 1170 x 16 = 18720 条数据。而对于高度为3的B+树，就可以存放 1170 x 1170 x 16 = 21902400 条数据（两千多万条数据），也就是对于两千多万条的数据，我们只需要高度为3的B+树就可以完成，通过主键查询只需要3次IO操作就能查到对应数据。所以在 InnoDB 中B+树高度一般为3层时，就能满足千万级的数据存储，所以一个节点为1页，也就是16k是比较合理的。

高度为3可以存储千万条数据。主键查询3次io



## 内连接和外连接的区别

内连接： inner join
仅返回符合条件的行 inner join xxx  on a=b
外连接：left join，right join，全连接 full join （mysql不支持） 可以用union 实现
左外连接是指在连接的时候将加上主表未匹配的数据
右外连接一样
全链接是将左右两表的数据都加起来

交叉连接：笛卡尔积：select * from student, score;

### sql语句

删除语句：delete、truncate、drop区别：
delete：可以加控制条件控制删除哪些行的数据
truncate不能加条件控制，是删除表的所有行数据,在innodb中会重置索引
drop是删除整张表。
数据恢复：delete可以恢复、truncate和drop不可恢复
执行速度：drop truncate delete

- `count(1) 、 count(*)和count(col)的区别`
  `count(1)和count(*)都是对表中列的数量做统计，count(1)是直接取索引，count(*)会扫描全部数据，都不会忽略列NULL。`
  `count(col)是对某一列统计，不包含该列为null的元素`

```sql
cource(id, name)
student(id, name)
student_score(cource_id, student_id, score)
输出每一个课程最高分学生的名字
课程 | 最高分分数 | 学生名

SELECT
sc.score AS '最高分',
s.`name` AS '姓名',
c.`name` AS '课程'
FROM
student_score AS sc
INNER JOIN student s ON s.id = sc.student_id
INNER JOIN cource c ON c.id = sc.cource_id
WHERE
sc.score = 
(
	SELECT
	MAX(ss.score) AS maxscore
	FROM
	student_score as ss
	WHERE
	ss.cource_id = sc.cource_id
	GROUP BY
	ss.cource_id
)

```


#### 查询表A重复3次以上的记录

```sql
select * from A where id in 
(
select id from A group by id having count(id) > 3
)
```

#### 实现upsert

on duplicatate key update

```sql
insert into A (id, name, age)
values 
	(1, 'a', 18)
on duplicatate key update id = 1;
```

### 数据库字符集

utf-8默认是utf-8mb3，是使用1-3个字节表示字符
utf8mb4才是正宗的utf8字符集。

字符集名称
Maxlen
`ascii`
`1`
`latin1`
`1`
`gb2312`
`2`
`gbk`
`2`
`utf8`
`3`
`utf8mb4`
`4`

- 字符集：
  character_set_server // 服务器级别

character_set_client // 服务器解码请求时使用的字符串
character_set_connection // 服务器处理请求时会把请求字符串从character_set_client转为character_set_connection
character_set_results // 服务端返回给客户端字符集

配置：

```
default-character-set
[client]
default-character-set=utf8s
```

SET NAMES utf8

### innodb引擎行格式

- compact格式
  ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/mR1keh.png)

- 变长字段长度列表
- NULL值列表
- 记录头信息
  ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/ULieoW.png)
  ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/Ud3vuX.png)


### 校验和：

在页头部分有校验和属性，校验和是通过算法将页面中的数据计算成 一个短的字符串。这样如果想要比较两个页面是否相同，可以直接先比较校验和，如果校验和不相同，则这两个页一定不相同。

作用：由于页的信息很多都是缓存在内存中的，在写入磁盘持久化的过程中，如果断电了，则可能会导致数据不一致。
因此在写入磁盘的时候会先将FileTrailer写入磁盘，这一块内容中包含页数据的校验和，然后开始将数据写入磁盘。
如果中间发生了断电，则重新启动的时候会对比校验和和数据的是否一致，如果不一致，则需要重新写入。保证了数据的完整性

FileTrailer