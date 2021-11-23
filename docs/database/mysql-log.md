# redo日志-重做日志

# redo日志是个啥

与在事务提交时将所有修改过的内存中的页面刷新到磁盘中相比，只将该事务执行过程中产生的redo日志刷新到磁盘的好处如下：
- redo日志占用的空间非常小
- redo日志是顺序写入磁盘的


[MySQL日记](https://blog.csdn.net/qq_41055045/article/details/108681970) 
[MySQL不会丢失数据的秘密，就藏在它的 7种日志里](https://mp.weixin.qq.com/s/-v6CHvvAwtuznG-bzZKQ0w) 