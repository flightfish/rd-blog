# 连接简介
## 连接的本质
```sql
//这两个表都有两个列，一个是INT类型的，一个是CHAR(1)类型的，填充好数据的两个表长这样：
mysql> SELECT * FROM t1;
+------+------+
| m1   | n1   |
+------+------+
|    1 | a    |
|    2 | b    |
|    3 | c    |
+------+------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM t2;
+------+------+
| m2   | n2   |
+------+------+
|    2 | b    |
|    3 | c    |
|    4 | d    |
+------+------+
3 rows in set (0.00 sec)
```
连接的本质就是把各个连接表中的记录都取出来依次匹配的组合加入结果集并返回给用户。<font color="red">连接查询的结果集中包含一个表中的每一条记录与另一个表中的每一条记录相互匹配的组合，像这样的结果集就可以称之为笛卡尔积。</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623152852787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
## 连接过程简介

```sql
//eg
SELECT * FROM t1, t2 WHERE t1.m1 > 1 AND t1.m1 = t2.m2 AND t2.n2 < 'd';
```
- 此处假设使用t1作为驱动表，那么就需要到t1表中找满足t1.m1 > 1的记录，因为表中的数据太少，我们也没在表上建立二级索引，所以此处查询t1表的访问方法就设定为all吧，也就是采用全表扫描的方式执行单表查询。关于如何提升连接查询的性能我们之后再说，现在先把基本概念捋清楚哈。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623155642530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
- 针对上一步骤中从驱动表产生的结果集中的每一条记录，分别需要到t2表中查找匹配的记录，所谓匹配的记录，指的是符合过滤条件的记录。因为是根据t1表中的记录去找t2表中的记录，所以t2表也可以被称之为被驱动表。上一步骤从驱动表中得到了2条记录，所以需要查询2次t2表。此时涉及两个表的列的过滤条件t1.m1 = t2.m2就派上用场了：
	- 当t1.m1 = 2时，过滤条件t1.m1 = t2.m2就相当于t2.m2 = 2，所以此时t2表相当于有了t2.m2 = 2、t2.n2 < 'd'这两个过滤条件，然后到t2表中执行单表查询。
	- 当t1.m1 = 3时，过滤条件t1.m1 = t2.m2就相当于t2.m2 = 3，所以此时t2表相当于有了t2.m2 = 3、t2.n2 < 'd'这两个过滤条件，然后到t2表中执行单表查询。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623155715338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>从上边两个步骤可以看出来，我们上边唠叨的这个两表连接查询共需要查询1次t1表，2次t2表。当然这是在特定的过滤条件下的结果，如果我们把t1.m1 > 1这个条件去掉，那么从t1表中查出的记录就有3条，就需要查询3次t2表了。也就是说在<font color="red">两表连接查询中，驱动表只需要访问一次，被驱动表可能被访问多次。</font>

## 内连接和外连接

```sql
CREATE TABLE student (
    number INT NOT NULL AUTO_INCREMENT COMMENT '学号',
    name VARCHAR(5) COMMENT '姓名',
    major VARCHAR(30) COMMENT '专业',
    PRIMARY KEY (number)
) Engine=InnoDB CHARSET=utf8 COMMENT '学生信息表';

CREATE TABLE score (
    number INT COMMENT '学号',
    subject VARCHAR(30) COMMENT '科目',
    score TINYINT COMMENT '成绩',
    PRIMARY KEY (number, subject)
) Engine=InnoDB CHARSET=utf8 COMMENT '学生成绩表';
mysql> SELECT * FROM student;
+----------+-----------+--------------------------+
| number   | name      | major                    |
+----------+-----------+--------------------------+
| 20180101 | 杜子腾    | 软件学院                 |
| 20180102 | 范统      | 计算机科学与工程         |
| 20180103 | 史珍香    | 计算机科学与工程         |
+----------+-----------+--------------------------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM score;
+----------+-----------------------------+-------+
| number   | subject                     | score |
+----------+-----------------------------+-------+
| 20180101 | 母猪的产后护理              |    78 |
| 20180101 | 论萨达姆的战争准备          |    88 |
| 20180102 | 论萨达姆的战争准备          |    98 |
| 20180102 | 母猪的产后护理              |   100 |
+----------+-----------------------------+-------+
4 rows in set (0.00 sec)
```
<font color="red">内连接和外连接的概念区别：
- <font color="red">外连接——驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集</font>
- <font color="red">内连接——驱动表中的记录在被驱动表中找不到匹配的记录，该记录不会加入到最后的结果集，</font>

>- <font color="red">内连接中的WHERE子句和ON子句是等价的
>- <font color="red">对于左（外）连接和右（外）连接来说，必须使用ON子句来指出连接条件。</font>
>- <font color="red">内连接和外连接的根本区别就是在驱动表中的记录不符合ON子句中的连接条件时不会把该记录加入到最后的结果集</font>

外连接过滤条件：
- WHERE子句中的过滤条件
WHERE子句中的过滤条件就是我们平时见的那种，不论是内连接还是外连接，凡是不符合WHERE子句中的过滤条件的记录都不会被加入最后的结果集。
- ON子句中的过滤条件
对于外连接的驱动表的记录来说，如果无法在被驱动表中找到匹配ON子句中的过滤条件的记录，那么该记录仍然会被加入到结果集中，对应的被驱动表记录的各个字段使用NULL值填充。
### 左（外）连接的语法

```sql
SELECT * FROM t1 LEFT [OUTER] JOIN t2 ON 连接条件 [WHERE 普通过滤条件];
```
### 右（外）连接的语法
```sql
SELECT * FROM t1 RIGHT [OUTER] JOIN t2 ON 连接条件 [WHERE 普通过滤条件];
```
### 内连接的语法
```sql
SELECT * FROM t1 [INNER | CROSS] JOIN t2 [ON 连接条件] [WHERE 普通过滤条件];
```
>- <font color="red">由于在内连接中ON子句和WHERE子句是等价的，所以内连接中不要求强制写明ON子句</font>
>- <font color="red">对于内连接来说，驱动表和被驱动表是可以互换的，并不会影响最后的查询结果</font>
>- <font color="red">左外连接和右外连接的驱动表和被驱动表不能轻易互换</font>
# 连接的原理
## 嵌套循环连接（Nested-Loop Join）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623164104594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

```base
for each row in t1 {   #此处表示遍历满足对t1单表查询结果集中的每一条记录
    
    for each row in t2 {   #此处表示对于某条t1表的记录来说，遍历满足对t2单表查询结果集中的每一条记录
    
        for each row in t3 {   #此处表示对于某条t1和t2表的记录组合来说，对t3表进行单表查询
            if row satisfies join conditions, send to client
        }
    }
}
```
<font color="red">驱动表只访问一次，但被驱动表却可能被多次访问，访问次数取决于对驱动表执行单表查询后的结果集中的记录条数</font>的连接执行方式称之为<font color="red">嵌套循环连接（Nested-Loop Join）</font>，这是最简单，也是最笨拙的一种连接查询算法。
## 使用索引加快连接速度
## 基于块的嵌套循环连接（Block Nested-Loop Join）
><font color="red">尽量减少访问被驱动表的次数</font>
<font color="red">join buffer</font>就是执行连接查询前申请的一块固定大小的内存，先把若干条驱动表结果集中的记录装在这个join buffer中，然后开始扫描被驱动表，每一条被驱动表的记录一次性和join buffer中的多条驱动表记录做匹配，因为匹配的过程都是在内存中完成的，所以这样可以显著减少被驱动表的<font color="red">I/O</font>代价

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623165815894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
 基于块的嵌套循环连接。提前划出一块内存（join buffer）存储驱动表结果集中的记录，然后开始扫描被驱动表，每一条被驱动表的记录一次性和这块内存中的多条驱动表记录匹配，可以显著减少被驱动表的I/O操作。


