### @Transactional注解 详解

#### 问题：

```
一个public方法M， 对表T有查询，全删除，全增加操作。 其中删除和增加操作在方法M的私有方法p里
两个并发线程来了，线程1 操作正确，线程2查询时为null。导致数据丢失。
```

#### 环境：

```
使用注解@Transactional(rollbackFor=Exception.class)
数据库sqlserver 查看隔离级别（DBCC Useroptions） read committed
```

#### 猜想：

```
1、私有方法p可能导致事务不传递
2、私有方法有try catch 语块，导致事务不生效
3、@Transactional默认的隔离级别是否为读未提交？
4、方法M外层的调用方法没有事务注解，是否也会影响事务的生效？
```

#### 验证

1、事务是否生效？

```
1.1@Transactional注解是基于AOP原理实现的，先找到它的拦截器TransactionInterceptor的invoke方法。
1.2 在createTransactionIfNecessary 和 commitTransactionAfterReturning打断点
1.3 启动程序，观察事务的开始时间和结束时间。发现事务生效且走完完整的M方法，所以1，2，4猜想不正确
```

![image-20211223185142257](C:\Users\MLT\AppData\Roaming\Typora\typora-user-images\image-20211223185142257.png)

2、看事务的执行情况

```
2.1 让程序停在全增加操作之前
2.2 通过db查询，发现查不到数据。重新看程序的查询语句发现少了with(nolock)，加上后发现查不到数据，链接一致等待
```

3、充分验证

```
3.1为使得验证更加符合生产问题。开启多线程。使得线程1 在全赠操作之前睡眠30s
3.2观察线程2查询得到的数据结果。
3.3将with(nolock)去掉，重复3.1再观察线程2查询得到的数据结果
```

#### 结论

```
查询语句 select * from xx with(nolock) where ***  with(nolock) 使得查询到拥有排他锁未结束事务的脏数据。
```

### 扩展

#### 1、事务的基本操作

```
建立连接
开启事务
执行
事务提交/事务回滚
关闭连接
```

#### 2、TransactionManager 和Transactional

```
spring对事务的管理主要通过 TransactionManager 和Transactional。springboot 将TransactionManager自动注入这里不详细赘述
```

#### 3、TransactionManager 

```
事务管理主要设计三个类PlatformTransactionManager、TransactionDefinition、TransactionStatus
PlatformTransactionManager :事务管理器(用来管理事务，包含事务的提交，回滚)：
TransactionStatus getTransaction(TransactionDefinition definition):获取事务状态信息
void commit(TransactionStatus status):提交事务
void rollback(TransactionStatus status):回滚事务

TransactionDefinition :事务定义信息(隔离，传播，超时，只读)

TransactionStatus :事务具体运行状态
```

![TransactionManager](C:\Users\MLT\Desktop\transationManager.png)

#### 4、Transactional

```
@Transactional注解和类似aop做的切面拦截

```

![](C:\Users\MLT\Downloads\transaction.png)



### 使用建议

```
sqlserver 默认使用 read committed 
mysql 数据库默认引擎innodb， 默认的数据隔离机制  REPEATABLE READ

查看 sqlserver 数据库的隔离级别： DBCC Useroptions
查看 mysql 数据库的隔离级别： show variables like '%tx_isolation%' 或 select @@tx_isolation;

@Transactional标注的pulic方法及其其中的private方法 应该遵循抛出数据库操作异常，而非捕获数据库操作异常。

propagation 



isolation
    DEFAULT(-1),       默认使用数据库隔离级别
    READ_UNCOMMITTED(1),   读未提交
    READ_COMMITTED(2),     读已提交
    REPEATABLE_READ(4),    可重复读
    SERIALIZABLE(8);       序列化执行

```





#### 请关注订阅号：程序员之自我成长

![qrcode_for_gh_8bc7ff61a6df_344](C:\Users\MLT\Downloads\qrcode_for_gh_8bc7ff61a6df_344.jpg)
