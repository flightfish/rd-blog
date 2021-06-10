# 字符集和比较规则
# 一些重要的字符集

 ## ASCII字符集(1字节)
 共收录128个字符，包括空格、标点符号、数字、大小写字母和一些不可见字符。由于总共才128个字符，所以可以使用1个字节来进行编码，我们看一些字符的编码方式：
 
```bash
'L' ->  01001100（十六进制：0x4C，十进制：76）
'M' ->  01001101（十六进制：0x4D，十进制：77）
```
## ISO 8859-1字符集（latin1）
共收录256个字符，是在ASCII字符集的基础上又扩充了128个西欧常用字符(包括德法两国的字母)，也可以使用1个字节来进行编码。
##  GB2312字符集
收录了汉字以及拉丁字母、希腊字母、日文平假名及片假名字母、俄语西里尔字母。其中收录汉字6763个，其他文字符号682个。同时这种字符集又兼容ASCII字符集，所以在编码方式上显得有些奇怪：


- 如果该字符在ASCII字符集中，则采用1字节编码。
- 否则采用2字节编码。
- 这种表示一个字符需要的字节数可能不同的编码方式称为变长编码方式。
   比方说字符串'爱u'，其中'爱'需要用2个字节进行编码，编码后的十六进制表示为0xB0AE，'u'需要用1个字节进行编码，编码后的十六进制表示为0x75，所以拼合起来就是0xB0AE75。

## GBK字符集 
GBK字符集只是在收录字符范围上对GB2312字符集作了扩充，编码方式上兼容GB2312。
## utf8字符集
收录地球上能想到的所有字符，而且还在不断扩充。这种字符集兼容ASCII字符集，采用变长编码方式，编码一个字符需要使用1～4个字节，比方说这样：

```bash
'L' ->  01001100（十六进制：0x4C）
'啊' ->  111001011001010110001010（十六进制：0xE5958A）
```
utf8只是Unicode字符集的一种编码方案，Unicode字符集可以采用utf8、utf16、utf32这几种编码方案，utf8使用1～4个字节编码一个字符，utf16使用2个或4个字节编码一个字符，utf32使用4个字节编码一个字符。

# MySQL中支持的字符集和排序规则

```bash
utf8mb3：阉割过的utf8字符集，只使用1～3个字节表示字符。
utf8mb4：正宗的utf8字符集，使用1～4个字节表示字符。
```
***在MySQL中utf8是utf8mb3的别名***，所以之后在MySQL中提到utf8就意味着使用1~3个字节来表示一个字符，如果大家有使用4字节编码一个字符的情况，比如存储一些emoji表情啥的，那请使用utf8mb4

## 字符集查看
```bash
SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061318004158.png)
## 比较规则的查看

```bash
SHOW COLLATION [LIKE 匹配的模式];
SHOW COLLATION LIKE 'utf8\_%';
```
utf8_general_ci是一种通用的比较规则
| 后缀 |  描述|
|--|--|
|  _ai| 不区分重音 |
|  _as| 区分重音 |
|  _ci| 不区分大小写 |
|  _cs| 区分大小写 |
|  _bin| 以二进制方式比较 |


# MySQL有4个级别的字符集和比较规则，分别是：
- 服务器级别
    character_set_server	服务器级别的字符集
    collation_server	服务器级别的比较规则
- 数据库级别
    character_set_database	当前数据库的字符集
    collation_database	当前数据库的比较规则
- 表级别
```mysql
    CREATE TABLE 表名 (列的信息)
        [[DEFAULT] CHARACTER SET 字符集名称]
        [COLLATE 比较规则名称]]
    
    ALTER TABLE 表名
        [[DEFAULT] CHARACTER SET 字符集名称]
        [COLLATE 比较规则名称]
```

- 列级别
```mysql
CREATE TABLE 表名(
    列名 字符串类型 [CHARACTER SET 字符集名称] [COLLATE 比较规则名称],
    其他列...
);

ALTER TABLE 表名 MODIFY 列名 字符串类型 [CHARACTER SET 字符集名称] [COLLATE 比较规则名称];
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325211419974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTg2MjMwOA==,size_16,color_FFFFFF,t_70)




