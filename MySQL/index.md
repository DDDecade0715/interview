#### Sql 语句应该考虑哪些安全性

1. 防止 sql 注入，对特殊字符进行转义和过滤
2. 用最小权限原则，最好不用 root 连接数据库
3. 当 sql 运行出错时，不要把错误信息展示给用户

#### Sql 索引

1. 普通索引
2. 唯一索引
3. 主键索引
4. 复合索引

#### MySQL 索引的优缺点

1. 优点
   - 加快查询速度
   - 可以加速表与表之间的连接
   - 在使用分组和排序进行检索的时候，可以减少查询中分组和排序的时间
   - 减少服务器对数据的扫描
   - 帮助服务器避免排序和临时表
   - 将随机 I/O 变成顺序 I/O
2. 缺点
   - 占用磁盘空间
   - 对于写的操作，会降低速度
   - 创建和维护索引要消耗时间，这种时间随着数据量的增加而增加

#### MySQL 什么时候会产生临时表

1. union 查询
2. order by 和 group by 子句不一样
3. distinct 加上 order by 查询
4. from 中的子查询
5. 表连接中，order by 的列不是驱动表中的

#### 什么时候应该创建索引

1. 经常需要搜索的列上，可以加快搜索的速度
2. 做为主键的列上
3. 经常用在 where 子句上的列
4. 经常用做排序和分组的列
5. 经常用在连接的列上，主要是一些外键，可以加快连接的速度

#### 什么时候不需要创建索引

1. 查询中很少使用的列
2. 只有很少数据值的列
3. 经常进行增删改的列，频繁更新的列
4. 数据重复且平均分布的列，比如 男女

#### 什么时候索引失效

1. 条件中带有 or
2. like 模糊查询中以 % 开头
3. 如果列是字符串，没有用引号引用起来
4. 使用函数或者四则运算
5. 判断索引不是某个值 !=
6. not in 查询

#### char 和 varchar 的区别

- char 是定长，不管存储是否达到设定的值，都按设定的值存储，效率较高
- varchar 是变长，存储的字符要比 char 长
- char 的效率比 varchar 要高

#### 优化 SQL 语句

1. 尽量选择较少的列查询
2. 在 where 后面频繁使用的字段加上索引
3. 避免是用 select * 查询
4. 避免在索引列上做运算，使用 not in 和 <> 等操作
5. 合理的使用 limit
6. 合适表分割，在查询较慢的地方使用 explain 分析查询语句

#### MySQL 中的事务（ACID）

1. **原子性（A）**：一组 sql 要么全部成功要么全部失败
2. **一致性（C）**：一致的从一种状态改变为另一种状态
3. **隔离型（I）**：一个事务未完成之前不会被另一个事务读取
4. **持久性（D）**：一旦食物提交，数据就会永久写入系统

#### MySQL 优化

1. 选取合适的字段属性，字段的宽度尽可能的小
2. 用连接查询来替代子查询
3. 使用事务
4. 使用外键
5. 建立索引
6. 优化查询语句
7. 锁定表

#### 数据库的三大范式

1. 保证每一列都是不可再分的属性值，保证每一列的原子性，减少冗余
2. 保证每一列都必须依赖主键
3. 保证每一列都与主键有直接关系，而不是间接关系

#### 数据库内部实现机制

1. 

#### MySQL 脏读/虚读/幻读解决方法

解决方法：**通过事务的隔离级别（读未提交/读已提交/可重复读/串行化）**

- 脏读：指的是一个线程中的事务读取到另一个线程中未提交的数据（**读已提交**）
- 虚读：指的是一个线程中的事务读取到另一个线程中提交的 update 的数据（**可重复读/加锁**）
- 幻读：指的是一个线程中的事务读取到另一个线程中提交的 inster 的数据（**串行化**）

#### 数据库的存储引擎

- MyISAM
  - 插入数据快，空间和内存使用比较低
  - 不支持事务
  - 数据存储在文件中
  - 支持表级锁
- InnoDB
  - 支持事务/行锁/外键
  - 数据存储在共享表空间
  - 支持奔溃后修复
- Memory
  - 所有数据在内存中，数据处理速度快
  - 对表的大小有要求，不能建立太大的表
  - 安全性不高
- Archive
  - 适合查询和存储

#### 悲观锁和乐观锁

- 乐观锁：每次去拿数据都会认为别人不会修改数据，但是在更新数据的时候会判断在此期间有没有更新这个数据，**适合多读的场景**
- 悲观锁：每次去拿数据都会认为别人会修改数据，所以每次都拿数据都会上锁，**适合多写的操作**

#### 主从复制

##### 1. 基本过程

1. 主库在事务提交时会把数据更作事件记录在二进制文件 binlog 中，主库上的 sync_binlog 参数控制 binlog 日志刷新到磁盘中
2. 主库推送二进制文件 binlog 到从库的中继日志 relay-log，之后从库根据 relay log 日志做数据变更操作

##### 2. 主要的三个线程

- Binlog dump 线程（主库）
- I/O 线程（从库）
- SQL 线程（从库）

    当从库上启动复制，首先创建 I/O 线程连接主库，主库随后创建 Binlog dump 线程读取数据库事件发送给 I/O 线程，I/O 线程读取到事件数据之后更新到从库的中继日志 Relay log 中，之后从库的 SQL 线程读取中继日志中更新的数据库事件并应用

##### 3. 三种复制方式

1. 基于 SQL 语句的复制

   每条修改数据的 SQL 都会保存在 binlog 日志中

2. 基于行的复制

   每行的数据变化都会记录到 binlog 日志中

3. 混合复制模式

   基于语句和行混合使用

##### 4. 复制的三种常见架构

1. 一主多从

   一个主库多个从库，读写分离，主库主要负责写和实时性较高的读的操作，从库主要负责读取的操作

2. 多级复制

   两个 master，多个 slave，其中一个 master 主要负责推送 binlog 日志到 slave

   **优点**：解决了主库的 I/O 负载和网络压力

   **缺点**：数据延时比较大（优化：master2 上选择 BLACKHOLE 引擎来降低延时，原理是 BLACKHOLE 表的数据不会写回到磁盘上，永远是空表，只用来记录 bin log 日志）

3. 双主复制

   两个 master 库，适合 DBA 做维护，master1 和 master2 互为主从，写的操作访问 master1 ，读的操作访问 master1 或者 master2

##### 5. 复制类型

- 异步复制

  数据库事务提交之后，在主库写入 bin log 日志就可以成功返回给客户端

- 半同步复制

  数据库事务提交之后，bin log 不仅要写在主库上，还要同时向从库推送，等到从库收到 bin log 日志后才会返回成功给客户端，如果从库长时间没有返回，则自动调整为异步复制

