
```sql
//eg single_table表建立了1个聚簇索引和4个二级索引
CREATE TABLE single_table (
    id INT NOT NULL AUTO_INCREMENT,
    key1 VARCHAR(100),
    key2 INT,
    key3 VARCHAR(100),
    key_part1 VARCHAR(100),
    key_part2 VARCHAR(100),
    key_part3 VARCHAR(100),
    common_field VARCHAR(100),
    PRIMARY KEY (id),
    KEY idx_key1 (key1),
    UNIQUE KEY idx_key2 (key2),
    KEY idx_key3 (key3),
    KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;
```
## 访问方法（access method）的概念

```sql
SELECT * FROM single_table WHERE id = 1438;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623120421560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

```sql
SELECT * FROM single_table WHERE key2 = 3841;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623120442233.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
><font color="red">把这种通过主键或者唯一二级索引列来定位一条记录的访问方法定义为：const，意思是常数级别的，代价是可以忽略不计的。</font>不过这种const访问方法只能在主键列或者唯一二级索引列和一个常数进行等值比较时才有效，如果主键或者唯一二级索引是由多个列构成的话，索引中的每一个列都需要与常数进行等值比较，这个const访问方法才有效（这是因为只有该索引中全部列都采用等值比较才可以定位唯一的一条记录）。

```sql
//对于唯一二级索引来说，查询该列为NULL值的情况比较特殊，比如这样：
SELECT * FROM single_table WHERE key2 IS NULL;
//因为唯一二级索引列并不限制 NULL 值的数量，所以上述语句可能访问到多条记录，也就是说 上边这个语句不可以使用const访问方法来执行
```
## ref
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623121139380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

```sql
SELECT * FROM single_table WHERE key1 = 'abc';
```
><font color="red">把这种搜索条件为二级索引列与常数等值比较，采用二级索引来执行查询的访问方法称为：ref。</font>

不过需要注意下边两种情况
- 二级索引列值为NULL的情况
不论是普通的二级索引，还是唯一二级索引，它们的索引列对包含NULL值的数量并不限制，所以我们采用key IS NULL这种形式的搜索条件最多只能使用ref的访问方法，而不是const的访问方法。
- 对于某个包含多个索引列的二级索引来说，只要是最左边的连续索引列是与常数的等值比较就可能采用ref的访问方法.
```sql
SELECT * FROM single_table WHERE key_part1 = 'god like';

SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 = 'legendary';

SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 = 'legendary' AND key_part3 = 'penta kill';
```
## ref_or_null

```sql
SELECT * FROM single_table WHERE key1 = 'abc' OR key1 IS NULL;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623121850587.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
## range

```sql
SELECT * FROM single_table WHERE key2 IN (1438, 6328) OR (key2 >= 38 AND key2 <= 79);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623122159320.png)
也就是从数学的角度看，每一个所谓的范围都是数轴上的一个区间，3个范围也就对应着3个区间：

- 范围1：key2 = 1438

- 范围2：key2 = 6328

- 范围3：key2 ∈ [38, 79]，注意这里是闭区间。

我们可以把那种索引列等值匹配的情况称之为单点区间，上边所说的范围1和范围2都可以被称为单点区间，像范围3这种的我们可以称为连续范围区间。
## index

```sql
SELECT key_part1, key_part2, key_part3 FROM single_table WHERE key_part2 = 'abc';
```
查询符合下边这两个条件：

- 它的查询列表只有3个列：key_part1, key_part2, key_part3，而索引idx_key_part又包含这三个列。

- 搜索条件中只有key_part2列。这个列也包含在索引idx_key_part中。
>由于二级索引记录比聚簇索记录小的多（聚簇索引记录要存储所有用户定义的列以及所谓的隐藏列，而二级索引记录只需要存放索引列和主键），而且这个过程也不用进行回表操作，所以直接遍历二级索引比直接遍历聚簇索引的成本要小很多。这种采用遍历二级索引记录的执行方式称之为：index。
## all
>最直接的查询执行方式就是我们已经提了无数遍的全表扫描，对于InnoDB表来说也就是直接扫描聚簇索引，把这种使用全表扫描执行查询的方式称之为：all。

# 注意事项
## 重温 二级索引 + 回表

```sql
//eg
SELECT * FROM single_table WHERE key1 = 'abc' AND key2 > 1000;
```
假设优化器决定使用idx_key1索引进行查询，那么整个查询过程可以分为两个步骤：
- 步骤1：使用二级索引定位记录的阶段，也就是根据条件key1 = 'abc'从idx_key1索引代表的B+树中找到对应的二级索引记录。
- 步骤2：回表阶段，也就是根据上一步骤中找到的记录的主键值进行回表操作，也就是到聚簇索引中找到对应的完整的用户记录，再根据条件key2 > 1000到完整的用户记录继续过滤。将最终符合过滤条件的记录返回给用户。

><font color="red">这里需要特别提醒大家的一点是，因为二级索引的节点中的记录只包含索引列和主键，所以在步骤1中使用idx_key1索引进行查询时只会用到与key1列有关的搜索条件，其余条件，比如key2 > 1000这个条件在步骤1中是用不到的，只有在步骤2完成回表操作后才能继续针对完整的用户记录中继续过滤。
>- 一般情况下执行一个查询只会用到单个二级索引，不过还是有特殊情况的，
## 明确range访问方法使用的范围区间
其实对于B+树索引来说，只要索引列和常数使用=、<=>、IN、NOT IN、IS NULL、IS NOT NULL、>、<、>=、<=、BETWEEN、!=（不等于也可以写成<>）或者LIKE操作符连接起来，就可以产生一个所谓的区间。
### 所有搜索条件都可以使用某个索引的情况

```sql
//eg
SELECT * FROM single_table WHERE key2 > 100 AND key2 > 200;
SELECT * FROM single_table WHERE key2 > 100 OR key2 > 200;
```
### 有的搜索条件无法使用索引的情况

```sql
SELECT * FROM single_table WHERE key2 > 100 AND common_field = 'abc';
```
请注意，这个查询语句中能利用的索引只有idx_key2一个，而idx_key2这个二级索引的记录中又不包含common_field这个字段，所以在使用二级索引idx_key2定位记录的阶段用不到common_field = 'abc'这个条件，这个条件是在回表获取了完整的用户记录后才使用的，而范围区间是为了到索引中取记录中提出的概念，所以在确定范围区间的时候不需要考虑common_field = 'abc'这个条件，我们在为某个索引确定范围区间的时候只需要把用不到相关索引的搜索条件替换为TRUE就好了。

```sql
//相当于
SELECT * FROM single_table WHERE key2 > 100 AND TRUE;
//化简之后就是这样：
SELECT * FROM single_table WHERE key2 > 100;
```
### 复杂搜索条件下找出范围匹配的区间

```sql
SELECT * FROM single_table WHERE 
        (key1 > 'xyz' AND key2 = 748 ) OR
        (key1 < 'abc' AND key1 > 'lmn') OR
        (key1 LIKE '%suf' AND key1 > 'zzz' AND (key2 < 8000 OR common_field = 'abc')) ;
```
- 假设我们使用idx_key1执行查询

```sql
(key1 > 'xyz' AND TRUE ) OR
(key1 < 'abc' AND key1 > 'lmn') OR
(TRUE AND key1 > 'zzz' AND (TRUE OR TRUE))
//化简一下上边的搜索条件就是下边这样：
(key1 > 'xyz') OR
(key1 < 'abc' AND key1 > 'lmn') OR
(key1 > 'zzz')
//最终
key1 > 'xyz'
```
<font color="red">上边那个有一坨搜索条件的查询语句如果使用 idx_key1 索引执行查询的话，需要把满足key1 > xyz的二级索引记录都取出来，然后拿着这些记录的id再进行回表，得到完整的用户记录之后再使用其他的搜索条件进行过滤。

- 假设我们使用idx_key2执行查询

```sql
(TRUE AND key2 = 748 ) OR
(TRUE AND TRUE) OR
(TRUE AND TRUE AND (key2 < 8000 OR TRUE))
//=>
key2 = 748 OR TRUE
//这个结果也就意味着如果我们要使用idx_key2索引执行查询语句的话，需要扫描idx_key2二级索引的所有记录，
//然后再回表，这不是得不偿失么，所以这种情况下不会使用idx_key2索引的。
```
## 索引合并
把这种使用到多个索引来完成一次查询的执行方法称之为：index merge，具体的索引合并算法有下边三种。
### Intersection合并(交集)

```sql
SELECT * FROM single_table WHERE key1 = 'a' AND key3 = 'b';
```
- 从idx_key1二级索引对应的B+树中取出key1 = 'a'的相关记录。
- 从idx_key3二级索引对应的B+树中取出key3 = 'b'的相关记录。
- 二级索引的记录都是由索引列 + 主键构成的，所以我们可以计算出这两个结果集中id值的交集。
- 按照上一步生成的id值列表进行回表操作，也就是从聚簇索引中把指定id值的完整用户记录取出来，返回给用户。
>虽然读取多个二级索引比读取一个二级索引消耗性能，但是<font color="red">读取二级索引的操作是顺序I/O，而回表操作是随机I/O，</font>所以如果只读取一个二级索引时需要回表的记录数特别多，而读取多个二级索引之后取交集的记录数非常少，当节省的因为回表而造成的性能损耗比访问多个二级索引带来的性能损耗更高时，读取多个二级索引后取交集比只读取一个二级索引的成本更低。

<font color="red">MySQL在某些特定的情况下才可能会使用到Intersection索引合并：</font>
- 情况一：二级索引列是等值匹配的情况，对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现只匹配部分列的情况。

```sql
//eg
SELECT * FROM single_table WHERE key1 = 'a' AND key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c';
```

```sql
//下边这两个查询就不能进行Intersection索引合并
SELECT * FROM single_table WHERE key1 > 'a' AND key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c';
SELECT * FROM single_table WHERE key1 = 'a' AND key_part1 = 'a';
```

- 情况二：主键列可以是范围匹配

```sql
//eg
SELECT * FROM single_table WHERE id > 100 AND key1 = 'a';
```
><font color="red">之所以在二级索引列都是等值匹配的情况下才可能使用Intersection索引合并，是因为只有在这种情况下根据二级索引查询出的结果集是按照主键值排序的。

>按照有序的主键值去回表取记录有个专有名词儿，叫：Rowid Ordered Retrieval，简称ROR，以后大家在某些地方见到这个名词儿就眼熟了。

### Union合并(并集)
<font color="red">MySQL在某些特定的情况下才可能会使用到Union索引合并：</font>
- 情况一：二级索引列是等值匹配的情况，对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现只出现匹配部分列的情况。

```sql
//eg
SELECT * FROM single_table WHERE key1 = 'a' OR ( key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c');
```

```sql
//下边这两个查询就不能进行Union索引合并
SELECT * FROM single_table WHERE key1 > 'a' OR (key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c');

SELECT * FROM single_table WHERE key1 = 'a' OR key_part1 = 'a';
```
- 情况二：主键列可以是范围匹配
- 情况三：使用Intersection索引合并的搜索条件

```sql
//eg
SELECT * FROM single_table WHERE key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c' OR (key1 = 'a' AND key3 = 'b');
```
### Sort-Union合并

```sql
SELECT * FROM single_table WHERE key1 < 'a' OR key3 > 'z'
```
- 先根据key1 < 'a'条件从idx_key1二级索引中获取记录，并按照记录的主键值进行排序
- 再根据key3 > 'z'条件从idx_key3二级索引中获取记录，并按照记录的主键值进行排序
- 因为上述的两个二级索引主键值都是排好序的，剩下的操作和Union索引合并方式就一样了。
## 索引合并注意事项
联合索引替代Intersection索引合并











