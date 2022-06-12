<a name="ZB0gr"></a>
## <br />
<a name="MO4zL"></a>
## Redis 基础数据结构
<a name="sF5Bw"></a>
### string (字符串)
字符串结构使用非常广泛，一个常见的用途就是缓存用户信息。我们将用户信息结构体 使用 JSON 序列化成字符串，然后将序列化后的字符串塞进 Redis 来缓存。同样，取用户信息会经过一次反序列化的过程。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/1204728/1653642897928-e21d3704-e6c8-4572-b544-ec3322cf9e24.png#clientId=ue20b33b7-4ac3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u3d47966b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=169&originWidth=572&originalType=binary&ratio=1&rotation=0&showTitle=false&size=3006&status=done&style=none&taskId=u1754464f-284d-4942-99d1-c3f0a637571&title=)<br />Redis 的字符串是动态字符串，内部为当前字 符串实际分配的空间 capacity 一般要高于实际字符串长度 len。当字符串长度小于 1M 时， 扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。需要注意的是 字符串最大长度为 512M。
<a name="ZchbZ"></a>
#### 过期和 set 命令扩展
可以对 key 设置过期时间，到点自动删除，这个功能常用来控制缓存的失效时间。
<a name="w8Dhp"></a>
#### 计数
如果 value 值是一个整数，还可以对它进行自增操作。自增是有范围的，它的范围是signed long 的最大最小值，超过了这个值，Redis 会报错。字符串是由多个字节组成，每个字节又是由 8 个 bit 组成，如此便可以将一个字符串看 成很多 bit 的组合，这便是 bitmap「位图」数据结构，
<a name="Ky07z"></a>
### list (列表)
Redis 的列表相当于 Java 语言里面的 LinkedList，注意它是链表而不是数组。这意味着 list 的插入和删除操作非常快，时间复杂度为 O(1)，但是索引定位很慢，时间复杂度为 O(n)，这点让人非常意外。<br />Redis 的列表结构常用来做异步队列使用。将需要延后处理的任务结构体序列化成字符 串塞进 Redis 的列表，另一个线程从这个列表中轮询数据进行处理。
<a name="CqHaG"></a>
#### 慢操作
lindex 相当于 Java 链表的 get(int index)方法，它需要对链表进行遍历，性能随着参数 index 增大而变差。
<a name="oslBH"></a>
#### 快速列表
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1204728/1653643236492-7501bcb6-5a7d-4ffd-972e-98ec2fccedf8.png#clientId=ue20b33b7-4ac3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=61&id=Y6jdT&margin=%5Bobject%20Object%5D&name=image.png&originHeight=121&originWidth=1280&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16251&status=done&style=none&taskId=u87b40620-8b22-44d8-a31a-7521f74fe9e&title=&width=640)<br />如果再深入一点，你会发现 Redis 底层存储的还不是一个简单的 linkedlist，而是称之为 快速链表 quicklist 的一个结构。<br />首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是 ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的 时候才会改成 quicklist。

一种是**压缩列表**（ziplist），另一种是**双向循环链表**。<br />当列表中存储的数据量比较小的时候，列表就可以采用压缩列表的方式实现。具体需要同时满足下面两个条件：

- 列表中保存的单个数据（有可能是字符串类型的）小于 64 字节；
- 列表中数据个数少于 512 个。
<a name="swHKq"></a>
### hash (字典)
字典类型用来存储一组数据对。每个数据对又包含键值两部分。字典类型也有两种实现方式。一种是我们刚刚讲到的**压缩列表**，另一种是**散列表**。<br />同样，只有当存储的数据量比较小的情况下，Redis 才使用压缩列表来实现字典类型。具体需要满足两个条件：

- 字典中保存的键和值的大小都要小于 64 字节；
- 字典中键值对的个数要小于 512 个。

<a name="Eyjs2"></a>
### set (集合)
集合这种数据类型用来存储一组不重复的数据。这种数据类型也有两种实现方法，一种是基于有**序数组**，另一种是基于**散列表**。<br />当要存储的数据，同时满足下面这样两个条件的时候，Redis 就采用有序数组，来实现集合这种数据类型。

- 存储的数据都是整数；
- 存储的数据元素个数不超过 512 个。
<a name="uhDPa"></a>
### zset (有序列表)
跟 Redis 的其他数据类型一样，有序集合也并不仅仅只有**跳表**这一种实现方式。当数据量比较小的时候，Redis 会用**压缩列表**来实现有序集合。<br />具体点说就是，使用压缩列表来实现有序集合的前提，有这样两个：

- 所有数据的大小都要小于 64 字节；
- 元素个数要小于 128 个。
<a name="U7GFW"></a>
## 容器型数据结构的通用规则
list/set/hash/zset 这四种数据结构是容器型数据结构，它们共享下面两条通用规则： 

- create if not exists

如果容器不存在，那就创建一个，再进行操作。比如 rpush 操作刚开始是没有列表的， Redis 就会自动创建一个，然后再 rpush 进去新元素。

- drop if no elements

如果容器里元素没有了，那么立即删除元素，释放内存。这意味着 lpop 操作到最后一 个元素，列表就消失了。
<a name="dqNKq"></a>
### 过期时间
Redis 所有的数据结构都可以设置过期时间，时间到了，Redis 会自动删除相应的对象。 需要注意的是过期是以对象为单位，比如一个 hash 结构的过期是整个 hash 对象的过期， 而不是其中的某个子 key。<br />还有一个需要特别注意的地方是如果一个字符串已经设置了过期时间，然后你调用了 set 方法修改了它，它的过期时间会消失。

<a name="XXLmB"></a>
## Redis存储结构体信息是用hash还是string
适合用string存储的情况

- 每次需要访问大量的字段
- 某些键的值存储差异，不能存储为字符串的时候

适合用hash存储的情况

- 在大多数情况中只需要访问少量字段
- 自己始终知道哪些字段可用，防止使用mget时获取不到想要的数据


![mysql_server](https://cdn.nlark.com/yuque/0/2022/png/1204728/1653642897928-e21d3704-e6c8-4572-b544-ec3322cf9e24.png)
