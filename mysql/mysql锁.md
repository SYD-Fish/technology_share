## Mysql锁相关

```
查数据库的基本信息： show table status from {db_name} where name= '{table_name}'
```



### Mysql的引擎

```
查引擎： show engines
```



### mysql的隔离级别

```
RU： READ_UNCOMMITTED(1),   读未提交
RC： READ_COMMITTED(2),     读已提交
RR： REPEATABLE_READ(4),    可重复读
SERIALIZABLE(8);       序列化执行

查隔离级别： select @@tx_isolation;    show variables like '%tx_isolation%';
设置隔离级别： set SESSION/GLOBAL transaction isolation level  READ COMMITTED;
默认：RR 可重复读 REPEATABLE_READ
```

### 数据不一致的几种概念

#### 脏读

```
已知有两个事务A和B, A读取了已经被B更新但还没有被提交的数据，之后，B回滚事务，A读取的数据就是脏数据。
```

#### 不可重复读

```
已知有两个事务A和B，A 多次读取同一数据，B在A多次读取的过程中对数据作了修改并提交，导致A多次读取同一数据时，结果不一致
```

#### 幻读 

```
已知有两个事务A和B，A从一个表中读取id=2的数据不存在，然后B在该表中插入了id=2的数据，并提交，A插入该数据报错, 重新查id=2的数据还是不存在，导致出现幻读。
```

### Mysql锁

#### 共享锁：S锁

```
select order_number from t_order where order_id = 2 LOCK in SHARE MODE;
事务A获取了共享锁，事务B可以读取  但不可以修改
```

#### 排他锁：X锁

```
select order_number from t_order where order_id = 2 for update;
事务A获取了共享锁，事务B不可以读取
```

#### 行锁

#### 表锁

#### 间隙锁

#### Next_Key锁

