# B+树索引

>InnoDB数据页的7个组成部分，知道了各个数据页可以组成一个<font color="red">双向链表</font>，而每个数据页中的记录会按照主键值从小到大的顺序组成一个<font color="red">单向链表</font>，每个数据页都会为存储在它里边儿的记录生成一个<font color="red">页目录</font>，在通过主键查找某条记录的时候可以在页目录中使用二分法快速定位到对应的槽，然后再遍历该槽对应分组中的记录即可快速找到指定的记录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616222154374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
## 没有索引的查找
><font color="red">由于我们并不能快速的定位到记录所在的页，所以只能从第一个页沿着双向链表一直往下找，在每一个页中根据我们刚刚唠叨过的查找方式去查找指定的记录。</font>
## 索引

```sql
为了故事的顺利发展，我们先建一个表：
mysql> CREATE TABLE index_demo(
    ->     c1 INT,
    ->     c2 INT,
    ->     c3 CHAR(1),
    ->     PRIMARY KEY(c1)
    -> ) ROW_FORMAT = Compact;
Query OK, 0 rows affected (0.03 sec)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061622273786.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616222823716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616222833503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
### 一个简单的索引方案

```sql
为了故事的顺利发展，我们这里需要做一个假设：假设我们的每个数据页最多能存放3条记录（实际上一个数据页非常大，可以存放下好多记录）。
有了这个假设之后我们向index_demo表插入3条记录：
mysql> INSERT INTO index_demo VALUES(1, 4, 'u'), (3, 9, 'd'), (5, 3, 'y');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 
```
那么这些记录已经按照主键值的大小串联成一个单向链表了，如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616224039910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

```sql
从图中可以看出来，index_demo表中的3条记录都被插入到了编号为10的数据页中了。此时我们再来插入一条记录：
mysql> INSERT INTO index_demo VALUES(4, 4, 'a');
Query OK, 1 row affected (0.00 sec)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616224152603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
<font color="red">**新分配的数据页编号可能并不是连续的，也就是说我们使用的这些页在存储空间里可能并不挨着。**</font>它们只是通过维护着上一个页和下一个页的编号而建立了链表关系。另外，页10中用户记录最大的主键值是5，而页28中有一条记录的主键值是4，因为5 > 4，所以这就不符合<font color="red">下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值的要求</font>，所以在插入主键值为4的记录的时候需要伴随着一次<font color="red">记录移动</font>，也就是把主键值为5的记录移动到页28中，然后再把主键值为4的记录插入到页10中，这个过程的示意图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616224242239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>这个过程表明了在对页中的记录进行增删改操作的过程中，我们必须通过一些诸如记录移动的操作来始终保证这个状态一直成立：<font color="red">下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值</font>。这个过程我们也可以称为<font color="red">页分裂。</font>
- 给所有的页建立一个目录项。
	<font color="red">页由于数据页的编号可能并不是连续的</font>，所以在向index_demo表中插入许多条记录后，可能是这样的效果：
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616224626523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>因为这些16KB的页在物理存储上可能并不挨着，所以如果想从这么多页中根据主键值快速定位某些记录所在的页，我们需要给它们做个目录，<font color="red">每个页对应一个目录项</font>，每个目录项包括下边两个部分：
>- 页的用户记录中最小的主键值，我们用key来表示。
>- 页号，我们用page_no表示。

所以我们为上边几个页做好的目录就像这样子：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616224836468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
我们只需要把几个目录项在物理存储器上连续存储，比如把他们放到一个数组里，就可以<font color="red">实现根据主键值快速查找某条记录的功能了</font>。
### InnoDB中的索引方案
>InnoDB的复用了之前存储用户记录的数据页来存储目录项，为了和用户记录做一下区分，我们把这些用来表示目录项的记录称为目录项记录。那InnoDB怎么区分一条记录是普通的用户记录还是目录项记录呢？别忘了记录头信息里的record_type属性，它的各个取值代表的意思如下：
>- 0：普通的用户记录
>- 1：目录项记录
>- 2：最小记录
>- 3：最大记录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616231245174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
> <font color="red">再次强调一遍目录项记录和普通的用户记录的不同点</font>：
> - <font color="red">目录项记录</font>的<font color="red">record_type</font>值是1，而<font color="red">普通用户记录</font>的 <font color="red">record_type</font>值是0。
>- <font color="red">**目录项记录只有主键值和页的编号两个列**</font>，而普通的用户记录的列是用户自己定义的，可能包含很多列，另外还有InnoDB自己添加的隐藏列。
>- 还记得我们之前在唠叨记录头信息的时候说过一个叫<font color="red">min_rec_mask</font>的属性么，只有在存储<font color="red">目录项记录</font>的页中的主键值最小的<font color="red">目录项记录</font>的<font color="red">min_rec_mask</font>值为1，其他别的记录的<font color="red">min_rec_mask</font>值都是0

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616232214251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616232322735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061623233810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>不论是存放用户记录的数据页，还是存放目录项记录的数据页，我们都把它们存放到<font color="red">B+</font>树这个数据结构中了，所以我们也称这些数据页为<font color="red">节点</font>。从图中可以看出来，我们的<font color="red">***实际用户记录其实都存放在B+树的最底层的节点上***</font>，这些节点也被称为<font color="red">叶子节点</font>或<font color="red">叶节点</font>，其余用来存放目录项的节点称为<font color="red">非叶子节点</font>或者<font color="red">内节点</font>，其中<font color="red">B+</font>树最上边的那个节点也称为<font color="red">根节点</font>。

我们用到的B+树都不会超过4层，那我们通过主键值去查找某条记录最多只需要做4个页面内的查找（查找3个目录项页和一个用户记录页），又因为在每个页面内有所谓的Page Directory（页目录）
### 聚簇索引
我们上边介绍的B+树本身就是一个目录，或者说本身就是一个索引。它有两个特点：

 

 1. 使用记录主键值的大小进行记录和页的排序，这包括三个方面的含义：
 	- 页内的记录是按照主键的大小顺序排成一个单向链表。
	- 各个存放用户记录的页也是根据页中用户记录的主键大小顺序排成一个双向链表。
	- 存放目录项记录的页分为不同的层次，在同一层次中的页也是根据页中目录项记录的主键大小顺序排成一个双向链表。
 2. B+树的叶子节点存储的是完整的用户记录。
	所谓完整的用户记录，就是指这个记录中存储了所有列的值（包括隐藏列）。

我们把具有这两种特性的<font color="red">B+</font>树称为<font color="red">聚簇索引</font>，所有完整的用户记录都存放在这个<font color="red">聚簇索引</font>的叶子节点处。这种聚簇索引并不需要我们在<font color="red">MySQL</font>语句中显式的使用<font color="red">INDEX</font>语句去创建（后边会介绍索引相关的语句），<font color="red">**InnoDB存储引擎会自动的为我们创建聚簇索引**</font>。另外有趣的一点是，在InnoDB存储引擎中，聚簇索引就是数据的存储方式<font color="red">（所有的用户记录都存储在了叶子节点）</font>，也就是所谓的<font color="red">**索引即数据，数据即索引**</font>。
### 二级索引
比方说我们用c2列的大小作为数据页、页中记录的排序规则，再建一棵B+树，效果如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616234210459.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>这个B+树与上边介绍的聚簇索引有几处不同：
>- 使用记录c2列的大小进行记录和页的排序，这包括三个方面的含义：
>   - 页内的记录是按照c2列的大小顺序排成一个单向链表。
>   - 各个存放用户记录的页也是根据页中记录的c2列大小顺序排成一个双向链表。
>   - 存放目录项记录的页分为不同的层次，在同一层次中的页也是根据页中目录项记录的c2列大小顺序排成一个双向链表。
>- B+树的叶子节点存储的并不是完整的用户记录，而只是<font color="red">c2列+主键</font>这两个列的值。
>- 目录项记录中不再是<font color="red">主键+页号</font>的搭配，而变成了<font color="red">c2列+页号</font>的搭配。

根据这个以c2列大小排序的B+树只能确定我们要查找记录的主键值，所以如果我们想根据c2列的值查找到完整的用户记录的话，仍然需要到聚簇索引中再查一遍，这个过程也被称为<font color="red">回表</font>。也就是根据c2列的值查询一条完整的用户记录需要使用到2棵B+树！！！

>这种按照<font color="red">非主键列</font>建立的B+树需要一次回表操作才可以定位到完整的用户记录，所以这种B+树也被称为<font color="red">二级索引（英文名secondary index），或者辅助索引</font>。由于我们使用的是c2列的大小作为B+树的排序规则，所以我们也称这个B+树为为<font color="red">c2列建立的索引</font>。

### 联合索引
>我们也可以同时以多个列的大小作为排序规则，也就是同时为多个列建立索引，比方说我们想让B+树按照c2和c3列的大小进行排序，这个包含两层含义：
>- 先把各个记录和页按照c2列进行排序。
>- 在记录的c2列相同的情况下，采用c3列进行排序


为c2和c3列建立的索引的示意图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616234953247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)

千万要注意一点，<font color="red">以c2和c3列的大小为排序规则建立的B+树称为联合索引，本质上也是一个二级索引。它的意思与分别为c2和c3列分别建立索引的表述是不同的</font>，不同点如下：
- 建立联合索引只会建立如上图一样的1棵B+树。
- 为c2和c3列分别建立索引会分别以c2和c3列的大小为排序规则建立2棵B+树。
### InnoDB的B+树索引的注意事项
#### 根页面万年不动窝
>我们前边介绍B+树索引的时候，为了大家理解上的方便，先把存储用户记录的叶子节点都画出来，然后接着画存储目录项记录的内节点，实际上B+树的形成过程是这样的：
>- 每当为某个表创建一个B+树索引（<font color="red">聚簇索引不是人为创建的，默认就有</font>）的时候，都会为这个索引创建一个根节点页面。最开始表中没有数据的时候，每个B+树索引对应的根节点中既没有用户记录，也没有目录项记录。
>- 随后向表中插入用户记录时，先把用户记录存储到这个根节点中。
>- 当根节点中的可用空间用完时继续插入记录，此时会将根节点中的所有记录复制到一个新分配的页，比如页a中，然后对这个新页进行<font color="red">页分裂</font>的操作，得到另一个新页，比如页b。这时新插入的记录根据键值（也就是聚簇索引中的主键值，二级索引中对应的索引列的值）的大小就会被分配到页a或者页b中，而根节点便升级为存储目录项记录的页。

这个过程需要大家特别注意的是：<font color="red">***一个B+树索引的根节点自诞生之日起，便不会再移动***</font>。这样只要我们对某个表建立一个索引，那么它的根节点的页号便会被记录到某个地方，然后凡是InnoDB存储引擎需要用到这个索引的时候，都会从那个固定的地方取出根节点的页号，从而来访问这个索引。

#### 内节点中目录项记录的唯一性
例如：
| c1 | c2      |c3|
|:--------:| :-------------:|:-------------:|
| 1 | 1 |'u' |
| 3 | 1 |'d' |
| 5 | 1 |'y' |
| 7 |1 |'a' |
如果二级索引中目录项记录的内容只是索引列 + 页号的搭配的话，那么为c2列建立索引后的B+树应该长这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200617141825696.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>为了让新插入记录能找到自己在那个页里，<font color="red">我们需要保证在B+树的同一层内节点的目录项记录除页号这个字段以外是唯一的</font>。所以对于二级索引的内节点的<font color="red">目录项记录</font>的内容实际上是由<font color="red">三个部分</font>构成的：
>- 索引列的值
>- 主键值
>- 页号

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200617142104627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
#### <font color="red">一个页面最少存储2条记录</font>
### MyISAM中的索引方案简单介绍
MyISAM记录也需要记录头信息来存储一些额外数据，我们以上边唠叨过的index_demo表为例，看一下这个表中的记录使用MyISAM作为存储引擎在存储空间中的表示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200617142417402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
- 使用MyISAM存储引擎的表会把索引信息另外存储到一个称为<font color="red">索引文件</font>的另一个文件中。MyISAM会单独为表的主键创建一个索引，只不过在<font color="red">索引的叶子节点中存储的不是完整的用户记录，而是主键值 + 行号的组合</font>。也就是先通过索引找到对应的行号，再通过行号去找对应的记录！

	这一点和InnoDB是完全不相同的，在InnoDB存储引擎中，我们只需要根据主键值对聚簇索引进行一次查找就能找到对应的记录，而在MyISAM中却需要进行一次回表操作，意味着<font color="red">***MyISAM中建立的索引相当于全部都是二级索引***</font>！

- 如果有需要的话，我们也可以对其它的列分别建立索引或者建立联合索引，原理和InnoDB中的索引差不多，不过在叶子节点处存储的是相应的列 + 行号。这些索引也全部都是二级索引。

>MyISAM的回表操作是十分快速的，因为是拿着地址偏移量直接到文件中取数据的，反观InnoDB是通过获取主键之后再去聚簇索引里边儿找记录，虽然说也不慢，但还是比不上直接用地址去访问。<font color="red">InnoDB中的索引即数据，数据即索引，而MyISAM中却是索引是索引、数据是数据</font>。
### MySQL中创建和删除索引的语句
>InnoDB和MyISAM会自动为主键或者声明为UNIQUE的列去自动建立B+树索引.

```sql
CREATE TALBE 表名 (
    各种列的信息 ··· , 
    [KEY|INDEX] 索引名 (需要被索引的单个列或多个列)
)
ALTER TABLE 表名 ADD [INDEX|KEY] 索引名 (需要被索引的单个列或多个列);
ALTER TABLE 表名 DROP [INDEX|KEY] 索引名;

//比方说我们想在创建index_demo表的时候就为c2和c3列添加一个联合索引
CREATE TABLE index_demo(
    c1 INT,
    c2 INT,
    c3 CHAR(1),
    PRIMARY KEY(c1),
    INDEX idx_c2_c3 (c2, c3)
);
ALTER TABLE index_demo DROP INDEX idx_c2_c3;
```

# B+树索引的使用
## 索引的代价
### 空间上的代价
>每建立一个索引都要为它建立一棵B+树，每一棵B+树的每一个节点都是一个数据页，一个页默认会占用16KB的存储空间，一棵很大的B+树由许多数据页组成，那可是很大的一片存储空间呢。
### 时间上的代价
>增、删、改操作可能会对节点和记录的排序造成破坏，所以存储引擎需要额外的时间进行一些记录移位，页面分裂、页面回收啥的操作来维护好节点和记录的排序。如果我们建了许多索引，每个索引对应的B+树都要进行相关的维护操作。
## B+树索引适用的条件

```sql
//为了故事的顺利发展，我们需要先创建一个表，这个表是用来存储人的一些基本信息的
CREATE TABLE person_info(
    id INT NOT NULL auto_increment,
    name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    phone_number CHAR(11) NOT NULL,
    country varchar(100) NOT NULL,
    PRIMARY KEY (id),
    KEY idx_name_birthday_phone_number (name, birthday, phone_number)
);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200617150318845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
>从图中可以看出，这个idx_name_birthday_phone_number索引对应的B+树中页面和记录的排序方式就是这样的：
>- 先按照name列的值进行排序。
>- 如果name列的值相同，则按照birthday列的值进行排序。
>- 如果birthday列的值也相同，则按照phone_number的值进行排序。
>
><font color="red">这个排序方式十分、特别、非常、巨、very very very重要，因为只要页面和记录是排好序的，我们就可以通过二分法来快速定位查找</font>。

### 全值匹配

```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1990-09-27' AND phone_number = '15123983239';
SELECT * FROM person_info WHERE birthday = '1990-09-27' AND phone_number = '15123983239' AND name = 'Ashburn';
//搜索条件的顺序对查询结果没影响，MySQL有一个叫查询优化器的东东，会分析这些搜索条件并且按照可以使用的索引中列的顺序来决定先使用哪个搜索条件，后使用哪个搜索条件。
```
### 匹配左边的列

```sql
SELECT * FROM person_info WHERE name = 'Ashburn';
//或者包含多个左边的列也行：
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1990-09-27';
```
<font color="red">如果我们想使用联合索引中尽可能多的列，搜索条件中的**各个列必须是联合索引中从最左边连续的列**</font>
### 匹配列前缀

```sql
//联合索引对应的B+树中的记录的name列的排列就是这样的
Aaron
Aaron
...
Aaron
Asa
Ashburn
...
Ashburn
Baird
Barlow
...
Barlow

```
我们<font color="red">只匹配它的前缀也是可以快速定位记录的</font>，比方说我们想查询名字以'As'开头的记录，那就可以这么写查询语句：
```sql
SELECT * FROM person_info WHERE name LIKE 'As%';
```
但是需要注意的是，如果只给出后缀或者中间的某个字符串。比如这样：

```sql
//MySQL就无法快速定位记录位置了
SELECT * FROM person_info WHERE name LIKE '%As%';
```
### 匹配范围值
idx_name_birthday_phone_number索引的B+树示意图，<font color="red">所有记录都是按照索引列的值从小到大的顺序排好序的。</font>

```sql
SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow';
```
<font color="red">**如果对多个列同时进行范围查找的话，只有对索引最左边的那个列进行范围查找的时候才能用到B+树索引**</font>

```sql
//通过name进行范围查找的记录中可能并不是按照birthday列进行排序的。
SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow' AND birthday > '1980-01-01';
```
### 精确匹配某一列并范围匹配另外一列

```sql
//如果左边的列是精确查找，则右边的列可以进行范围查找
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000';

```
>这个查询的条件可以分为3个部分：
>- name = 'Ashburn'，对name列进行精确查找，<font color="red">当然可以使用B+树索引了。
>- birthday > '1980-01-01' AND birthday < '2000-12-31'，由于name列是精确查找，所以通过name = 'Ashburn'条件查找后得到的结果的name值都是相同的，它们会再按照birthday的值进行排序。<font color="red">所以此时对birthday列进行范围查找是可以用到B+树索引的。
>- phone_number > '15100000000'，通过birthday的范围查找的记录的birthday的值可能不同，<font color="red">所以这个条件无法再利用B+树索引了，只能遍历上一步查询得到的记录。

```sql
//这个搜索三个值可以用到联合索引
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1980-01-01' AND phone_number > '15100000000';
```

### 用于排序
><font color="red">ORDER BY子句后的列如果不加ASC或者DESC默认是按照ASC排序规则排序的，也就是升序排序的。</font>
```bash
SELECT * FROM person_info ORDER BY name, birthday, phone_number LIMIT 10;
```
#### 使用联合索引进行排序注意事项
<font color="red">ORDER BY的子句后边的列的顺序也必须按照索引列的顺序给出</font>
#### 不可以使用索引进行排序的几种情况
##### ASC、DESC混用
##### 排序列包含非同一个索引的列

```sql
//eg
SELECT * FROM person_info ORDER BY name, country LIMIT 10;
```
##### 排序列使用了复杂的表达式

```sql
//eg 使用了UPPER函数修饰过的列就不是单独的列啦，这样就无法使用索引进行排序啦。
SELECT * FROM person_info ORDER BY UPPER(name) LIMIT 10;
```
### 用于分组

```sql
SELECT name, birthday, phone_number, COUNT(*) FROM person_info GROUP BY name, birthday, phone_number
```
## 回表的代价
用索引idx_name_birthday_phone_number的查询有这么两个特点：

- 会使用到两个B+树索引，一个二级索引，一个聚簇索引。

- 访问二级索引使用<font color="red">顺序I/O</font>，访问聚簇索引使用<font color="red">随机I/O</font>。

<font color="red">**需要回表的记录越多，使用二级索引的性能就越低**</font>，甚至让某些查询宁愿使用全表扫描也不使用二级索引。

### 覆盖索引
为了彻底告别回表操作带来的性能损耗，我们建议：<font color="red">**最好在查询列表里只包含索引列**</font>。覆盖索引不需要回表。
<font color="red">**我们很不鼓励用*号作为查询列表，最好把我们需要查询的列依次标明**</font>
## 如何挑选索引
### 只为用于搜索、排序或分组的列创建索引
>WHERE子句中的列、连接子句中的连接列，或者出现在ORDER BY或GROUP BY子句中的列创建索引
### 考虑列的基数
><font color="red">列的基数</font>指的是某一列中不重复数据的个数，比方说某个列包含值2, 5, 8, 2, 5, 8, 2, 5, 8，虽然有9条记录，但该列的基数却是3。也就是说，<font color="red">在记录行数一定的情况下，列的基数越大，该列中的值越分散，列的基数越小，该列中的值越集中。</font>
><font color="red">**最好为那些列的基数大的列建立索引，为基数太小列的建立索引效果可能不好。**</font>
### 索引列的类型尽量小
><font color="red">该类型表示的数据范围的大小。尽量让索引列使用较小的类型</font>
>能使用INT就不要使用BIGINT
### 索引字符串值的前缀
>只对字符串的前几个字符进行索引也就是说在二级索引的记录中只保留字符串前几个字符。

```sql
//eg
CREATE TABLE person_info(
    name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    phone_number CHAR(11) NOT NULL,
    country varchar(100) NOT NULL,
    KEY idx_name_birthday_phone_number (name(10), birthday, phone_number)
);   
```
name(10)就表示在建立的B+树索引中只保留记录的前10个字符的编码，这种<font color="red">只索引字符串值的前缀的策略是我们非常鼓励的，尤其是在字符串类型能存储的字符比较多的时候</font>。
### 索引列前缀对排序的影响
```sql
SELECT * FROM person_info ORDER BY name LIMIT 10;
```
>因为二级索引中不包含完整的name列信息，所以无法对前十个字符相同，后边的字符不同的记录进行排序，也就是使用索引列前缀的方式无法支持使用索引排序，只好乖乖的用文件排序喽
### 让索引列在比较表达式中单独出现
<font color="red">**如果索引列在比较表达式中不是以单独列的形式出现，而是以某个表达式，或者函数调用形式出现的话，是用不到索引的。**</font>

```sql
WHERE my_col * 2 < 4//my_col不是单独列 存储引擎会依次遍历所有的记录
WHERE my_col < 4/2//my_col单独列  存储引擎使用B+树
```
### 主键插入顺序
假设某个数据页存储的记录已经满了，它存储的主键值在1~100之间：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200617170134574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
如果此时再插入一条主键值为9的记录，那它插入的位置就如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200617170152672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)
<font color="red">**页面分裂和记录移位意味着：性能损耗**</font>
所以我们建议：<font color="red">**让主键具有AUTO_INCREMENT**</font>，让存储引擎自己为表生成主键，而不是我们手动插入 。
### 冗余和重复索引

```sql
CREATE TABLE person_info(
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    phone_number CHAR(11) NOT NULL,
    country varchar(100) NOT NULL,
    PRIMARY KEY (id),
    KEY idx_name_birthday_phone_number (name(10), birthday, phone_number),
    KEY idx_name (name(10))
);   
```
