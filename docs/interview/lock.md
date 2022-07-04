# 面试汇总

## 锁

- 悲观锁 ：坏事一定会发生，所以先预防
- 乐观锁"坏事未必会发生，时候补偿
    - 自旋锁 一种常见的乐观锁
        - ABA问题 主要通过版本号或者标志位解决
        - 保障CAS原子性命令（lock指令）
        
- 排它锁：只有一个线程可以访问的
- 共享锁：可以允许多个线程访问
- 读写锁
    - 读锁 读的时候不允许写，允许读（共享锁）
    - 写锁 写的时候不允许写，不允许读（排它锁）
    
- 统一锁：大粒度锁
    > 锁定A等待B，锁定B等待A==>死锁，通过设计A+B统一成大锁
    > 
- 分段锁：分成一段一段小粒度锁



## mysql
[你了解乐观锁和悲观锁吗？](https://www.cnblogs.com/kismetv/p/10787228.html)
[MySQL的索引结构为什么使用B+树？](https://www.cnblogs.com/kismetv/p/11582214.html)
[你懂 MySQL 事务日志吗？](https://zhuanlan.zhihu.com/p/267225276)
[MySQL不会丢失数据的秘密，就藏在它的 7种日志里](https://mp.weixin.qq.com/s/-v6CHvvAwtuznG-bzZKQ0w)



