# mysql

# 客户端与服务器连接的过程
- TCP/IP
- 命名管道和共享内存
- Unix域套接字文件

![服务器处理客户端请求](./img/mysql_connection.png)



## 优化
### MySQL的limit用法和分页查询的性能分析及优化
```mysql
SELECT * FROM articles WHERE category_id = 123 ORDER BY id LIMIT 10000, 10;
/*就是越往后分页，LIMIT语句的偏移量就会越大，速度也会明显变慢。*/
```
- 子查询的分页方式延迟关联
```mysql
SELECT * FROM articles WHERE  id >=  
(SELECT id FROM articles  WHERE category_id = 123 ORDER BY id LIMIT 10000, 1) LIMIT 10 ;
```
- JOIN分页方式
```mysql
SELECT * FROM `content` AS t1   
JOIN (SELECT id FROM `content` ORDER BY id desc LIMIT ".($page-1)*$pagesize.", 1) AS t2   
WHERE t1.id <= t2.id ORDER BY t1.id desc LIMIT $pagesize; 
```
>join分页和子查询分页的效率
> 优化思想：避免数据量大时扫描过多的记录。通过使用覆盖索引查询返回需要的主键,再根据主键关联原表获得需要的数据。

## mysql数据实时同步到Elasticsearch
### mysql binlog日志
>mysql的binlog日志主要用于数据库的主从复制与数据恢复。binlog中记录了数据的增删改查操作，主从复制过程中，主库向从库同步binlog日志，从库对binlog日志中的事件进行重放，从而实现主从同步。

mysql binlog日志有三种模式
- ROW: 记录每一行数据被修改的情况，但是日志量太大
- STATEMENT: 记录每一条修改数据的SQL语句，减少了日志量，但是SQL语句使用函数或触发器时容易出现主从不一致
- MIXED: 结合了ROW和STATEMENT的优点，根据具体执行数据操作的SQL语句选择使用ROW或者STATEMENT记录日志

### 同步方式
#### binlog
>优点：业务与 ES 数据耦合度低，业务逻辑中不需要关心 ES 数据的写入；<br>
>缺点：Binlog 模式只能使用 ROW 模式，且引入了新的同步服务，增加了开发量以及维护成本，也增大了 ES 同步的风险。

 >要通过mysql binlog将数据同步到ES集群，只能使用ROW模式，因为只有ROW模式才能知道mysql中的数据的修改内容。 
 - go-mysql-elasticsearch开源工具同步数据到ES
 >go-mysql-elasticsearch的基本原理是：如果是第一次启动该程序，首先使用mysqldump工具对源mysql数据库进行一次全量同步，通过elasticsearch client执行操作写入数据到ES；然后实现了一个mysql client,作为slave连接到源mysql,源mysql作为master会将所有数据的更新操作通过binlog event同步给slave， 通过解析binlog event就可以获取到数据的更新内容，之后写入到ES.
  
 使用限制
 >- mysql binlog必须是ROW模式
 >- 要同步的mysql数据表必须包含主键，否则直接忽略，这是因为如果数据表没有主键，UPDATE和DELETE操作就会因为在ES中找不到对应的document而无法进行同步
 >- 不支持程序运行过程中修改表结构
>- 要赋予用于连接mysql的账户RELOAD权限以及REPLICATION权限, SUPER权限
- mypipe
> mypipe是一个mysql binlog同步工具，在设计之初是为了能够将binlog event发送到kafka, 当前版本可根据业务的需要也可以自定以将数据同步到任意的存储介质，

 使用限制
 >- mysql binlog必须是ROW模式
 >- mypipe只是将binlog日志内容解析后编码成Avro格式推送到kafka broker, 并不是将数据推送到kafka，如果需要同步到ES集群，可以从kafka消费数据后，再写入ES
 >- 消费kafka中的消息(mysql insert, update, delete操作及具体的数据)，需要对消息内容进行Avro解析，获取到对应的数据操作内容，进行下一步处理；mypipe封装了一个KafkaGenericMutationAvroConsumer类，可以直接继承该类使用，或者自行解析
>- 要赋予用于连接mysql的账户REPLICATION权限
>- mypipe只支持binlog同步，不支持存量数据同步，也即mypipe程序启动后无法对mysql中已经存在的数据进行同步

- cancal 
>优点：基于binlog 支持增删改实时同步，阿里开源<br>
>缺点：不支持全量同步

#### ES API
> 优点：简洁明了，能够灵活的控制数据的写入；<br/>
> 缺点：与业务耦合严重，强依赖于业务系统的写入方式。

## 说说mysql主从同步怎么做的吧？
> - master提交完事务后，写入binlog
>- slave连接到master，获取binlog
>- master创建dump线程，推送binglog到slave
>- slave启动一个IO线程读取同步过来的master的binlog，记录到relay log中继日志中
> - slave再开启一个sql线程读取relay log事件并在slave执行，完成同步
>- slave记录自己的binglog
![mysql主从同步](/_picture/mysql主从同步.jpeg)

### 全同步复制
主库写入binlog后强制同步日志到从库，所有的从库都执行完成后才返回给客户端，但是很显然这个方式的话性能会受到严重影响。
### 半同步复制
和全同步不同的是，半同步复制的逻辑是这样，从库写入日志成功后返回ACK确认给主库，主库收到至少一个从库的确认就认为写操作完成。
 
 

# MySQL架构

- 连接层：最上层是一些客户端和连接服务。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。
  服务器也会为安全接入的每个客户端验证它所具有的操作权限。
- 服务层：第二层服务层，主要完成大部分的核心服务功能， 包括查询解析、分析、优化、缓存、以及所有的内置函数，所有跨存储引擎的功能也都在这一层实现，包括触发器、存储过程、视图等。

- 引擎层：第三层存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取存储层：
- 第四层为数据存储层，主要是将数据存储在运行于该设备的文件系统之上，并完成与存储引擎的交互。

>画出 MySQL 架构图，这种变态问题都能问的出来 MySQL 的查询流程具体是？or 一条SQL语句在MySQL中如何执行的？

   客户端请求 ---> 连接器（验证用户身份，给予权限） ---> 查询缓存（存在缓存则直接返回，不存在则执行后续操作） ---> 分析器（对SQL进行词法分析和语法分析操作） ---> 优化器（主要对执行的sql优化选择最优的执行方案方法） ---> 执行器（执行时会先看用户是否有执行权限，有才去使用这个引擎提供的接口） ---> 去引擎层获取数据返回（如果开启查询缓存则会缓存查询结果）


![mysql_server](/_picture/mysql_server.jpeg)



