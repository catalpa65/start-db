# 公开课《MySQL数据库高级》https://www.bilibili.com/video/BV1KW411u7vy
## MySQL架构介绍 
## 索引优化分析
* MyISAM 和InnoDB区别：InnoDB支持主外键，支持事务，支持行锁，缓存索引和真实数据
    MyISAM不支持主外键，不支持事务，锁是表锁，只缓存索引。都是默认安装的
* 索引是帮助MySQL高效获取数据的数据结构，索引本身也很大，往往以文件的形式存储在磁盘上
    索引可以大大提高查询数据，同时会降低更新表的速度
```sql
id name email wxNum

select * from user where name='xxx'
create index idx_user_name on user(name)    ---单值索引

select * from user where name='xxx' and email='xxx'
create index idx_user_name on user(name,email)  ---复合索引
alter user add index idx_user_name on(name,email)   ---另一种写法
```
* 主键自动创建唯一索引
* 频繁作为查询条件的字段应该创建索引
* 频繁更新的字段不适合创建索引（每次更新都要更新索引）
* 高并发下倾向建复合索引
* 需要排序的字段可以通过索引加快速度
* 某个数据列大量重复

* 数据量太少不用建索引 300W一下
* 经常新增删除的表不用建索引
* 数据列重复且分布平均，建索引没有太大的效果

* explain分析：
    * id 查询次序。大的先执行。id相同，顺序执行
    * select_type 有6种：
        SIMPLE 简单的select查询，不包含子查询或者union
        PRIMARY 查询中若包含任何复杂的子查询，最外层查询则被标记为primary
        SUBQUERY 在select或where列表中包含了子查询
        DERIVED 在from列表中包含的子查询会被标记为derived(衍生) MySQL会递归执行这些子查询，把结果放在衍生表中
        UNION union之后的select语句会被标记为union
        UNION RESULT 合并两个语句的结果集
    * table 操作哪张表
    * type 访问类型排列：
        system > const > eq_ref > ref > range > index(全索引扫描) > ALL(全表扫描)
        system 表只有一行记录，平时不会出现，可以忽略不计
        const 通过索引一次就找到了，相当于单表中的主键id
        eq_ref 单条匹配
        ref 多条匹配
        range 检索指定氛围的行,一般要达到range级别
        index 全索引扫描，index与all的区别为index只遍历索引树。这通常比ALL块，因为索引文件比数据文件小的多
    * possible_keys 可能用到的索引
    * key 实际使用的索引
    * key_len 索引字段字节数，越小越好
    * ref 显示索引的哪一列被使用了
    * rows 大致估算需要读取多少行
    * extra 其他重要信息
        Using filesort 无法使用索引进行排序，使用文件排序
        Using temporary 用了临时表保存数据，常见于order by group by
        Using index 表示相应的select操作使用了覆盖索引

* 范围条件不做索引，会引起索引失效
* 永远用小结果集驱动大结果集
* 优先优化内层循环
* 尽量保证被驱动的表使用索引

* 如何避免索引失效？
    * 全值匹配我最爱，索引可以覆盖查询字段（顺序不重要，能自动优化）
    * 最左原则，如果索引了多列，查询应该从索引最左列开始并且不跳过索引中的列
    * 不要在索引列上做任何操作（计算，函数，类型转换），会导致索引失效而全表扫描
    * 存储引擎不能使用索引中范围条件后面的列
    * 减少使用 * 
    * 使用  != is null isnotnull 无法使用索引，会导致全表扫描
    * like以通配符开头 '%xxx' 会造成索引失效，解决方案是尽量使用覆盖索引
    * 字符串不加单引号索引会失效
    * 少用or,会引起索引失效
    
## 查询截取分析
* 观察，至少跑一天，看看生产的慢SQL情况
* 开启慢查询日志，设置阈值
* explain + 慢SQL分析
* show profile
* DBA 数据库服务参数调优

查询优化
* 小表驱动大表，如果B是小表用in，如果B是大表用exists

慢查询
* 设置阈值，记录日志

## MySQL锁机制
************************** myisam engine 表锁（适用读多）****************************
读锁：sessionA加了读锁（读锁非独占）
sessionA sessionB都可以读被加锁表（sessionA也不能读其他表）
sessionA sessionB都不能修改被加锁表，除非锁释放

写锁：sessionA加了写锁（写锁独占）
sessionA可以读被加锁表  sessionB不能读被加锁表，除非锁释放（sessionA也不能读其他表）
sessionA 可以修改 sessionB不能修改被加锁表，除非锁释放

总结：对于myisam引擎，读操作（自动加读锁），不会阻塞各线程对表的读请求。只有当读锁释放后才会执行其他进程的写操作
对于myisam引擎，写操作（自动加写锁），只有本线程能执行读写操作，其他线程操作本表会阻塞
总之，读锁会阻塞写，写锁读写都会阻塞
*******************************************************************************


************************** InnoDB engine 行锁*********************************
* 注意如果索引失效（比如该用引号没用），会造成行数变表锁
* 间隙锁：查询过程中使用范围查找的话，他会锁定整个范围内所有值，即使这个值不存在。这就要主要查询范围不要太大，
    影响性能
* 单独锁定某一行 for update
总结：InnoDB与myisam的区别是 InnoDB采用了行锁，并且支持事务（默认事务隔离级别可重复读）
*******************************************************************************
## 主从复制



# 高性能MySQL(第三版)
## 第一章 MySQL架构与历史
## 第二章 MySQL基准测试
## 第三章 服务器性能剖析
## 第四章 Schema与数据类型优化
## 第五章 创建高性能索引
## 第六章 查询性能优化
## 第七章 MySQL高级特性
## 第八章 优化服务器设置
## 第九章 操作系统与硬件优化
## 第十章 复制
## 第十一章 可扩展的MySQL
## 第十二章 高可用性
## 第十三章 云端的MySQL
## 第十四章 应用层优化
## 第十五章 备份与恢复
## 第十六章 MySQL用户工具
