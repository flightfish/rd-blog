# innodb

# InnoDB记录存储结构
InnoDB是MySQL默认的存储引擎，InnoDB是一个将表中的数据存储到磁盘上的存储引擎
<font color="red">InnoDB采取的方式是：将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB中页的大小一般为 <font  color=grep>16 KB</font></font>

## InnoDB行格式

>设计InnoDB存储引擎有4种不同类型的行格式，分别是<font color="red">Compact、Redundant、Dynamic和Compressed</font>行格式
### COMPACT行格式![!](https://img-blog.csdnimg.cn/2020061319244453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

```sql
mysql> CREATE TABLE record_format_demo (
    ->     c1 VARCHAR(10),
    ->     c2 VARCHAR(10) NOT NULL,
    ->     c3 CHAR(10),
    ->     c4 VARCHAR(10)
    -> ) CHARSET=ascii ROW_FORMAT=COMPACT;
Query OK, 0 rows affected (0.03 sec)
mysql> SELECT * FROM record_format_demo;
+------+-----+------+------+
| c1   | c2  | c3   | c4   |
+------+-----+------+------+
| aaaa | bbb | cc   | d    |
| eeee | fff | NULL | NULL |
+------+-----+------+------+
2 rows in set (0.00 sec)

mysql>
```
#### 记录的额外信息
这部分信息是<font color="red">服务器为了描述这条记录而不得不额外添加的一些信息</font>，这些额外信息分为3类，分别是<font color="#FF5733">变长字段长度列表、NULL值列表和记录头信息</font>，我们分别看一下。
		在Compact行格式中，<font color="red">把所有变长字段的真实数据占用的字节长度都存放在记录的开头部位，从而形成一个变长字段长度列表，各变长字段数据占用的字节数按照列的顺序逆序存放</font>，我们再次强调一遍，是<font color="red">***逆序***</font>存放！

#### 变长字段长度列表

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616171642837.png) 1. 假设某个字符集中表示一个字符最多需要使用的字节数为W，也就是使用SHOW CHARSET语句的结果中的Maxlen列，比方说utf8字符集中的W就是3，gbk字符集中的W就是2，ascii字符集中的W就是1。
 1. 对于变长类型VARCHAR(M)来说，这种类型表示能存储最多M个字符（注意是字符不是字节），所以这个类型能表示的字符串最多占用的字节数就是M×W。
 2. 假设它实际存储的字符串占用的字节数是L。
<font color="red">总结一下就是说：如果该可变字段允许存储的最大字节数（M×W）超过255字节并且真实存储的字节数（L）超过127字节，则使用2个字节，否则使用1个字节。</font>
变长字段长度列表中只存储值为 <font color="red">***非NULL*** </font>的列内容占用的长度，值为 NULL 的列的长度是不储存的 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616172150353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
#### NULL值列表
首先统计表中允许存储NULL的列有哪些。
 1. 我们前边说过，主键列、被NOT NULL修饰的列都是不可以存储NULL值的，所以在统计的时候不会把这些列算进去
 2.  <font color="red">如果表中没有允许存储 NULL 的列，则 NULL值列表 也不存在了</font>，否则将每个允许存储NULL的列对应一个二进制位，二进制位按照列的<font color="red">***顺序***</font>逆序排列，二进制位表示的意义如下：
	 - 二进制位的值为1时，代表该列的值为NULL
	 - 二进制位的值为0时，代表该列的值不为NULL
 3. MySQL规定NULL值列表必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补0。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200613184800804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
![!\[](https://img-blog.csdnimg.cn/20200613192556637.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

#### 记录头信息

除了变长字段长度列表、NULL值列表之外，还有一个用于描述记录的记录头信息，它是由<font color="red">固定的5个字节组成。5个字节也就是40个二进制位</font>，不同的位代表不同的意思

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616173211137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
这些二进制位代表的详细信息如下表：



| 名称 | 大小（单位：bit）      | 描述      |
|:------------:| :-------------:| :-------------:|
| <font color="red" size=1>预留位1</font>  |  <font color="red"  size=1>1</font> | 没有使用|
| <font color="red" size=1>预留位2</font>  |  <font color="red"  size=1>1</font> | 没有使用|
| <font color="red" size=1>delete_mask</font>  |  <font color="red"  size=1>1</font> | 标记该记录是否被删除(0的时候代表记录并没有被删除，1代表记录被删除掉了)|
| <font color="red" size=1>min_rec_mask</font>  |  <font color="red"  size=1>1</font> | B+树的每层非叶子节点中的最小记录都会添加该标记|
| <font color="red" size=1>n_owned</font>  |  <font color="red"  size=1>4</font> | 表示当前记录拥有的记录数|
| <font color="red" size=1>heap_no</font>  |  <font color="red"  size=1>13</font> |表示当前记录在记录堆的位置信息|
| <font color="red" size=1>record_type</font>  |  <font color="red"  size=1>3</font> | 表示当前记录的类型， <font color="red" size=1>0</font> 表示普通记录,<font color="red" size=1>1</font>  表示B+树非叶子节点记录，<font color="red" size=1>2</font> 表示最小记录，<font color="red" size=1>3</font> 表示最大记录|
| <font color="red" size=1>next_record</font>  |  <font color="red"  size=1>16</font> | 表示下一条记录的相对位置|
![!\[](https://img-blog.csdnimg.cn/20200613192207296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
MySQL会为每个记录默认的添加一些列（也称为隐藏列），
| 列名 | 是否必须      | 占用空间      |描述
|:------------:| :-------------:| :-------------:|  :-------------:|
| <font color="red" size=1>row_id</font>  | 否| <font color="red"  size=1>6</font>字节 | 行ID，唯一标识一条记录|
| <font color="red" size=1>transaction_id</font>  | 是|  <font color="red"  size=1>6</font>字节 | 事务ID|
| <font color="red" size=1>roll_pointer</font>  | 是|  <font color="red"  size=1>7</font>字节 | 回滚指针|
>这里需要提一下InnoDB表对主键的生成策略：优先使用用户自定义主键作为主键，如果用户没有定义主键，则选取一个Unique键作为主键，如果表中连Unique键都没有定义的话，则InnoDB会为表默认添加一个名为row_id的隐藏列作为主键。所以我们从上表中可以看出：<font color="red">InnoDB存储引擎会为每条记录都添加 ***transaction_id 和 roll_pointer*** 这两个列，但是 row_id 是可选的（在没有自定义主键以及Unique键的情况下才会添加该列）</font>。这些隐藏列的值不用我们操心，InnoDB存储引擎会自己帮我们生成的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616175349564.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>看这个图的时候我们需要注意几点：
 >- 表record_format_demo使用的是ascii字符集，所以0x61616161就表示字符串'aaaa'，0x626262就表示字符串'bbb'，以此类推。
> - 注意第1条记录中c3列的值，它是CHAR(10)类型的，它实际存储的字符串是：'cc'，而ascii字符集中的字节表示是'0x6363'，虽然表示这个字符串只占用了2个字节，但整个c3列仍然占用了10个字节的空间，除真实数据以外的8个字节的统统都用<font color="red">空格字符</font>填充，空格字符在ascii字符集的表示就是0x20。
 >- 注意第2条记录中c3和c4列的值都为NULL，它们被存储在了前边的NULL值列表处，在记录的真实数据处就不再冗余存储，从而节省存储空间。
对于 CHAR(M) 类型的列来说，当列采用的是定长字符集时，该列占用的字节数不会被加到变长字段长度列表，而如果采用变长字符集时，该列占用的字节数也会被加到变长字段长度列表。

### Redundant行格式
其实知道了Compact行格式之后，其他的行格式就是依葫芦画瓢了。我们现在要介绍的Redundant行格式是MySQL5.0之前用的一种行格式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616193435730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

### CHAR(M)列的存储格式
<font color="red">ascii字符集一个定长字符集</font>，也就是说表示一个字符采用固定的一个字节，如果采用变长的字符集（也就是表示一个字符需要的字节数不确定，比如<font color="red">gbk表示一个字符要1～2个字节、utf8表示一个字符要1~3个字节等）</font>的话，c3列的长度也会被存储到变长字段长度列表中：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616180042588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
### 行溢出数据
#### VARCHAR(M)最多能存储的数据
>如果VARCHAR(M)类型的列使用的不是ascii字符集，那M的最大取值取决于该字符集表示一个字符最多需要的字节数。在列的值允许为NULL的情况下，gbk字符集表示一个字符最多需要2个字节，那在该字符集下，M的最大取值就是32766（也就是：65532/2），也就是说最多能存储32766个字符；utf8字符集表示一个字符最多需要3个字节，那在该字符集下，M的最大取值就是21844，就是说最多能存储21844（也就是：65532/3）个字符
#### 记录中的数据太多产生的溢出
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200613192053131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
从图中可以看出来，对于Compact和Redundant行格式来说，如果某一列中的数据非常多的话，在本记录的真实数据处只会存储该列的前<font color="red">**768**</font>个字节的数据和一个指向其他页的地址，然后把剩下的数据存放到其他页中，这个过程也叫做<font color="red">**行溢出**</font>，存储超出768字节的那些页面也被称为<font color="red">**行溢页**</font>。画一个简图就是这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200613192039434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
<font color="red">***不只是 VARCHAR(M) 类型的列，其他的 TEXT、BLOB 类型的列在存储数据非常多的时候也会发生行溢出***</font>

#### 行溢出的临界点
>每个页除了存放我们的记录以外，也需要存储一些额外的信息，乱七八糟的额外信息加起来需要132个字节的空间（现在只要知道这个数字就好了），其他的空间都可以被用来存储记录。

>每个记录需要的额外信息是27字节。
这27个字节包括下边这些部分：
2个字节用于存储真实数据的长度
1个字节用于存储列是否是NULL值
5个字节大小的头信息
6个字节的row_id列
6个字节的transaction_id列
7个字节的roll_pointer列

<font color="red">***我们一条记录的某个列中存储的数据占用的字节数非常多时，该列就可能成为溢出列***</font>
><font color="red">我现在使用的MySQL版本是5.7，它的默认行格式就是Dynamic</font>。Dynamic和Compressed行格式这俩行格式和Compact行格式挺像，只不过在处理行溢出数据时有点儿分歧，它们不会在记录的真实数据处存储字段真实数据的前768个字节，而是把所有的字节都存储到其他页面中，只在记录的真实数据处存储其他页面的地址，就像这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061319202455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

# InnoDB数据页结构
## 数据页结构
存放表空间头部信息的页，存放Insert Buffer信息的页
存放INODE信息的页
存放undo日志信息的页
<font color="red">存放记录(数据)的页为索引（INDEX）（数据）页</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616195427982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
| 名称 | 中文名      | 占用空间大小     |简单描述      |
|:------------:| :-------------:| :-------------:|:-------------:|
| <font color="red" size=1>File Header</font>  |文件头部 | <font color="red"  size=1>38</font>字节 | 页的一些通用信息|
| <font color="red" size=1>Page Header</font>  |页面头部 | <font color="red"  size=1>56</font>字节 | 数据页专有的一些信息|
| <font color="red" size=1>Infimum + Supremum</font>  |最小记录和最大记录 | <font color="red"  size=1>26</font>字节 | 两个虚拟的行记录|
| <font color="red" size=1>User Records</font>  |用户记录 | 不确定 | 	实际存储的行记录内容|
| <font color="red" size=1>Free Space</font>  |空闲空间 | 不确定 | 	页中尚未使用的空间|
| <font color="red" size=1>Page Directory</font>  |页面目录 | 不确定 | 		页中的某些记录的相对位置|
| <font color="red" size=1>File Trailer</font>  |文件头部 | <font color="red"  size=1>8</font>字节 | 校验页是否完整|
## 记录在页中的存储
>在页的7个组成部分中，我们自己存储的记录会按照我们指定的<font color="red" >行格式</font>存储到<font color="red" >User Records</font>部分。但是在一开始生成页的时候，其实并没有<font color="red" >User Records</font>这个部分，每当我们插入一条记录，都会从<font color="red" >Free Space</font>部分，也就是尚未使用的存储空间中申请一个记录大小的空间划分到<font color="red" >User Records</font>部分，当<font color="red" >Free Space</font>部分的空间全部被<font color="red" >User Records</font>部分替代掉之后，也就意味着这个页使用完了，如果还有新的记录插入的话，就需要去申请新的页了，这个过程的图示如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616200456516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

```sql
mysql> CREATE TABLE page_demo(
    ->     c1 INT,
    ->     c2 INT,
    ->     c3 VARCHAR(10000),
    ->     PRIMARY KEY (c1)
    -> ) CHARSET=ascii ROW_FORMAT=Compact;
Query OK, 0 rows affected (0.03 sec)
```
><font color="red" >我们把 c1 列指定为主键，所以在具体的行格式中InnoDB就没必要为我们去创建那个所谓的 row_id 隐藏列了</font>

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061620094950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

```sql
//下边我们试着向page_demo表中插入几条记录：
mysql> INSERT INTO page_demo VALUES(1, 100, 'aaaa'), (2, 200, 'bbbb'), (3, 300, 'cccc'), (4, 400, 'dddd');
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616201446538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

 - <font color="red">delete_mask</font>
 >这些被删除的记录之所以不立即从磁盘上移除，是因为移除它们之后把其他的记录在磁盘上重新排列需要性能消耗，所以只是<font color="red">打一个删除标记</font>而已，所有被删除掉的记录都会组成一个所谓的<font color="red">垃圾链表</font>，在这个链表中的记录占用的空间称之为所谓的<font color="red">可重用空间</font>，之后如果有新记录插入到表中的话，可能把这些被删除的记录占用的存储空间覆盖掉。
 - <font color="red">heap_no</font>
 >InnoDB自动给每个页里边儿加了两个记录，由于这两个记录并不是我们自己插入的，所以有时候也称为伪记录或者虚拟记录。这两个伪记录一个代表最小记录，一个代表最大记录.
对于<font color="red" >**一条完整的记录**来说，比较记录的大小就是比较主键的大小</font>。比方说我们插入的4行记录的主键值分别是：1、2、3、4，这也就意味着这4条记录的大小从小到大依次递增。
 
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061620232511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

>由于这两条记录不是我们自己定义的记录，所以它们并不存放在页的<font color="red">User Records</font>部分，他们被单独放在一个称为<font color="red">Infimum + Supremum</font>的部分，<font color="red">最小记录和最大记录的heap_no值分别是0和1</font>，也就是说它们的位置最靠前。如图所示:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616202338575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
 - <font color="red">next_record</font>
 >表示<font color="red">从当前记录的真实数据到下一条记录的真实数据的地址偏移量</font>。比方说第一条记录的next_record值为32，意味着从第一条记录的真实数据的地址处向后找32个字节便是下一条记录的真实数据。如果你熟悉数据结构的话，就立即明白了，这其实是个<font color="#FF5733 ">链表</font>，可以通过一条记录找到它的下一条记录。但是需要注意注意再注意的一点是，<font color="red ">下一条记录指得并不是按照我们插入顺序的下一条记录，而是按照主键值由小到大的顺序的下一条记录</font> 。而且规定 <font color="red ">***Infimum记录（也就是最小记录）*** 的下一条记录就是本页中主键值最小的用户记录，而本页中主键值最大的用户记录的下一条记录就是 ***Supremum记录（也就是最大记录）***</font> ，为了更形象的表示一下这个next_record起到的作用，我们用箭头来替代一下next_record中的地址偏移量：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616203500750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
删掉第2条记录后的示意图就是：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616204057827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>从图中可以看出来，删除第2条记录前后主要发生了这些变化：
>- 第2条记录并没有从存储空间中移除，而是把该条记录的delete_mask值设置为1。
>- 第2条记录的next_record值变为了0，意味着该记录没有下一条记录了。
>- 第1条记录的next_record指向了第3条记录。
>- 还有一点你可能忽略了，就是最大记录的n_owned值从5变成了4，关于这一点的变化我们稍后会详细说明的。

<font color="red">所以，不论我们怎么对页中的记录做增删改操作，InnoDB始终会维护一条记录的单链表，链表中的各个节点是按照主键值由小到大的顺序连接起来的。</font>
>next_record这个指针指向记录头信息和真实数据之间的位置呢？为啥不干脆指向整条记录的开头位置，也就是记录的额外信息开头的位置呢？<font color="#FF5733 "> 因为这个位置刚刚好，向左读取就是记录头信息，向右读取就是真实数据。我们前边还说过变长字段长度列表、NULL值列表中的信息都是逆序存放，这样可以使记录中位置靠前的字段和它们对应的字段长度信息在内存中的距离更近，可能会提高高速缓存的命中率</font>

主键值为2的记录被我们删掉了，但是存储空间却没有回收，我们再次把这条记录插入到表中，InnoDB并没有因为新记录的插入而为它申请新的存储空间，而是直接复用了原来被删除记录的存储空间。<font color="red">当数据页中存在多条被删除掉的记录时，这些记录的next_record属性将会把这些被删除掉的记录组成一个垃圾链表，以备之后重用这部分存储空间。</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616204718193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
## Page Directory（页目录）
>设计InnoDB制作了一个类似的目录，他们的制作过程是这样的：
>- 将所有正常的记录（包括最大和最小记录，不包括标记为已删除的记录）划分为几个组。
>- 每个组的最后一条记录（也就是组内最大的那条记录）的头信息中的n_owned属性表示该记录拥有多少条记录，也就是该组内共有几条记录。
>- <font color="#FF5733 ">将每个组的最后一条记录的地址偏移量单独提取出来按顺序存储到靠近页的尾部的地方，这个地方就是所谓的Page Directory，也就是页目录</font>（此时应该返回头看看页面各个部分的图）。页面目录中的<font color="#FF5733 ">这些地址偏移量被称为槽（英文名：Slot）</font>，所以这个页面目录就是由槽组成的。

比方说现在的page_demo表中正常的记录共有6条，InnoDB会把它们分成两组，第一组中只有一个最小记录，第二组中是剩余的5条记录，看下边的示意图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616205339829.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>从这个图中我们需要注意这么几点：
>- 现在页目录部分中有两个槽，也就意味着我们的记录被分成了两个组，<font color="#FF5733 ">槽1中的值是112， 代表最大记录的地址偏移量（就是从页面的0字节开始数，数112个字节）；</font>槽0中的值是99，代表最小记录的地址偏移量。
>- 注意最小和最大记录的头信息中的n_owned属性
>	- 最小记录的n_owned值为1，这就代表着以最小记录结尾的这个分组中只有1条记录，也就是最小记录本身。
>	- 最大记录的n_owned值为5，这就代表着以最大记录结尾的这个分组中只有5条记录，包括最大记录本身还有我们自己插入的4条记录。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616205815719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616210512309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>InnoDB对每个分组中的记录条数是有规定的：<font color="red">对于最小记录所在的分组只能有 1 条记录，最大记录所在的分组拥有的记录条数只能在 1～8条之间，剩下的分组中记录的条数范围只能在是 4~8 条之间</font>。所以分组是按照下边的步骤进行的：
>- 初始情况下一个数据页里只有最小记录和最大记录两条记录，它们分属于两个分组。
>- 之后每插入一条记录，都会从页目录中找到主键值比本记录的主键值大并且差值最小的槽，然后把该槽对应的记录的n_owned值加1，表示本组内又添加了一条记录，直到该组中的记录数等于8个。
>- 在一个组中的记录数等于8个后再插入一条记录时，会将组中的记录拆分成两个组，一个组中4条记录，另一个5条记录。这个过程会在页目录中新增一个槽来记录这个新增分组中最大的那条记录的偏移量。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616211242461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>所以在一个数据页中查找指定主键值的记录的过程分为两步：
>-	<font color="red"> **通过二分法确定该记录所在的槽，并找到该槽所在分组中主键值最小的那条记录**。
>- <font color="red">**通过记录的next_record属性遍历该槽所在的组中的各个记录。**</font>
## Page Header（页面头部）
<font color="red">这个部分占用固定的56个字节</font>
| 名称 | 大小（单位：bit）      | 描述      |
|:------------:| :-------------:| :-------------:|
| <font color="red" size=1>PAGE_N_DIR_SLOTS</font>  |  <font color="red"  size=1>2</font> |在页目录中的槽数量|
| <font color="red" size=1>PAGE_HEAP_TOP</font>  |  <font color="red"  size=1>2</font> | 还未使用的空间最小地址，也就是说从该地址之后就是<font color="red" size=1>Free Space</font>|
| <font color="red" size=1>PAGE_N_HEAP</font>  |  <font color="red"  size=1>2</font> |本页中的记录的数量（包括最小和最大记录以及标记为删除的记录）|
| <font color="red" size=1>PAGE_FREE</font>  |  <font color="red"  size=1>2</font> | 第一个已经标记为删除的记录地址（各个已删除的记录通过 <font color="red"  size=1>next_record</font> 也会组成一个单链表，这个单链表中的记录可以被重新利用）|
| <font color="red" size=1>PAGE_GARBAGE</font>  |  <font color="red"  size=1>2</font> | 已删除记录占用的字节数|
| <font color="red" size=1>PAGE_LAST_INSERT</font>  |  <font color="red"  size=1>2</font> |最后插入记录的位置|
| <font color="red" size=1>PAGE_DIRECTION</font>  |  <font color="red"  size=1>2</font> | 记录插入的方向|
| <font color="red" size=1>PAGE_N_RECS</font>  |  <font color="red"  size=1>2</font> | 一个方向连续插入的记录数量|
| <font color="red" size=1>PAGE_N_DIRECTION</font>  |  <font color="red"  size=1>2</font> | 该页中记录的数量（不包括最小和最大记录以及被标记为删除的记录）|
| <font color="red" size=1>PAGE_MAX_TRX_ID</font>  |  <font color="red"  size=1>8</font> | 修改当前页的最大事务ID，该值仅在二级索引中定义|
| <font color="red" size=1>PAGE_LEVEL</font>  |  <font color="red"  size=1>2</font> | 当前页在B+树中所处的层级|
| <font color="red" size=1>PAGE_INDEX_ID</font>  |  <font color="red"  size=1>8</font> | 索引ID，表示当前页属于哪个索引|
| <font color="red" size=1>PAGE_BTR_SEG_LEAF</font>  |  <font color="red"  size=1>10</font> | B+树叶子段的头部信息，仅在B+树的Root页定义|
| <font color="red" size=1>PAGE_BTR_SEG_TOP</font>  |  <font color="red"  size=1>10</font> | B+树非叶子段的头部信息，仅在B+树的Root页定义|

>- <font color="red" >PAGE_DIRECTION</font>
假如新插入的一条记录的主键值比上一条记录的主键值大，我们说这条记录的插入方向是右边，反之则是左边。用来表示最后一条记录插入方向的状态就是PAGE_DIRECTION。
>-  <font color="red" >PAGE_N_DIRECTION</font>
假设连续几次插入新记录的方向都是一致的，InnoDB会把沿着同一个方向插入记录的条数记下来，这个条数就用PAGE_N_DIRECTION这个状态表示。当然，如果最后一条记录的插入方向改变了的话，这个状态的值会被清零重新统计。

## File Header（文件头部）
File Header针对各种类型的页都通用，也就是说不同类型的页都会以File Header作为第一个组成部分， <font color="red" >它描述了一些针对各种页都通用的一些信息， 这个部分占用固定的38个字节</font>，是由下边这些内容组成的：
| 名称 | 大小（单位：bit）      | 描述      |
|:------------:| :-------------:| :-------------:|
| <font color="red" size=1>FIL_PAGE_SPACE_OR_CHKSUM</font>  |  <font color="red"  size=1>4</font> |页的校验和（checksum值）|
| <font color="red" size=1>FIL_PAGE_OFFSET</font>  |  <font color="red"  size=1>4</font> | 页号|
| <font color="red" size=1>FIL_PAGE_PREV</font>  |  <font color="red"  size=1>4</font> |上一个页的页号|
| <font color="red" size=1>FIL_PAGE_NEXT</font>  |  <font color="red"  size=1>4</font> | 下一个页的页号|
| <font color="red" size=1>FIL_PAGE_LSN</font>  |  <font color="red"  size=1>8</font> | 页面被最后修改时对应的日志序列位置（英文名是：Log Sequence Number）|
| <font color="red" size=1>FIL_PAGE_TYPE</font>  |  <font color="red"  size=1>2</font> |该页的类型|
| <font color="red" size=1>FIL_PAGE_FILE_FLUSH_LSN</font>  |  <font color="red"  size=1>8</font> | 	仅在系统表空间的一个页中定义，代表文件至少被刷新到了对应的LSN值|
| <font color="red" size=1>FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID</font>  |  <font color="red"  size=1>4</font> |页属于哪个表空间|
对照着这个表格，我们看几个目前比较重要的部分：
- <font color="red" >FIL_PAGE_SPACE_OR_CHKSUM</font>

>这个代表当前页面的校验和（checksum）。啥是个校验和？就是对于一个很长很长的字节串来说，我们会通过某种算法来计算一个比较短的值来代表这个很长的字节串，这个<font color="red" >比较短的值就称为校验和</font>。这样在比较两个很长的字节串之前先比较这两个长字节串的校验和，如果校验和都不一样两个长字节串肯定是不同的，所以省去了直接比较两个比较长的字节串的时间损耗。

- <font color="red" >FIL_PAGE_OFFSET</font>

>每一个页都有一个单独的页号，就跟你的身份证号码一样，InnoDB通过页号来可以唯一定位一个页。

- <font color="red" >FIL_PAGE_TYPE</font>
>这个代表当前页的类型，我们前边说过，InnoDB为了不同的目的而把页分为不同的类型，我们上边介绍的其实都是存储记录的数据页，其实还有很多别的类型的页，具体如下表：

| 类型名称 | 十六进制      | 描述      |
|:------------:| :-------------:| :-------------:|
| <font color="red" size=1>FIL_PAGE_TYPE_ALLOCATED</font>  |  0x0000|最新分配，还没使用|
| <font color="red" size=1>FIL_PAGE_UNDO_LOG</font>  |  0x0002|Undo日志页|
| <font color="red" size=1>FIL_PAGE_INODE</font>  |  0x0003|段信息节点|
| <font color="red" size=1>FIL_PAGE_IBUF_FREE_LIST</font>  |  0x0004|insert Buffer空闲列表|
| <font color="red" size=1>FIL_PAGE_TYPE_SYS</font>  |  0x0006|系统页|
| <font color="red" size=1>FIL_PAGE_UNDO_LOG</font>  |  0x0002|Undo日志页|
| <font color="red" size=1>FIL_PAGE_TYPE_TRX_SYS</font>  |  0x0007|事务系统数据|
| <font color="red" size=1>FIL_PAGE_TYPE_FSP_HDR</font>  |  0x0008	|表空间头部信息|
| <font color="red" size=1>FIL_PAGE_TYPE_XDES</font>  | 0x0009|扩展描述页|
| <font color="red" size=1>FIL_PAGE_TYPE_BLOB</font>  |  0x000A|溢出页|
| <font color="red" size=1>FIL_PAGE_INDEX</font>  | 0x45BF	|索引页，也就是我们所说的<font color="red" >数据页</font>|
- <font color="red" >FIL_PAGE_PREV和FIL_PAGE_NEXT</font>
<font color="red" >**并不是所有类型的页都有上一个和下一个页的属性**</font>，不过我们本集中唠叨的数据页（也就是类型为FIL_PAGE_INDEX的页）是有这两个属性的，所以<font color="red" >**所有的数据页其实是一个双链表**</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616220340273.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
## File Trailer
>为了检测一个页是否完整（也就是在同步的时候有没有发生只同步一半的尴尬情况），设计InnoDB的在每个页的尾部都加了一个File Trailer部分，这个部分由8个字节组成，可以分成2个小部分：
- 前4个字节代表页的校验和
	这个部分是和File Header中的校验和相对应的。每当一个页面在内存中修改了，在同步之前就要把它的校验和算出来，因为File Header在页面的前边，所以校验和会被首先同步到磁盘，当完全写完时，校验和也会被写到页的尾部，如果完全同步成功，则页的首部和尾部的校验和应该是一致的。如果写了一半儿断电了，那么在File Header中的校验和就代表着已经修改过的页，而在File Trailer中的校验和代表着原先的页，二者不同则意味着同步中间出了错。

- 后4个字节代表页面被最后修改时对应的日志序列位置（LSN）
<font color="red" >这个File Trailer与File Header类似，都是所有类型的页通用的。</font>
