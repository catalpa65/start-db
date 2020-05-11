## MySQL高级
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
* explain分析：
    * id 查询次序。大的先执行。id相同，顺序执行
    * select_type 有6种，SIMPLE，PRIMARY，SUBQUERY，DERIVED，UNION，UNION RESULT
    * table 操作哪张表
    * type 访问类型排列：system>const >eq_ref(单条匹配)>ref(多条匹配)>range(检索指定范围的行)>index(全索引扫描)>ALL 
    一般要达到range级别
        
    