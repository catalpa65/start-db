# 公开课《MySQL数据库高级》https://www.bilibili.com/video/BV1KW411u7vy
```
SQL https://blog.csdn.net/Pzzzz_wwy/article/details/106600571
```
## MySQL架构介绍 
## 索引优化分析
* 简介
```
存储引擎:
MyISAM 和InnoDB区别，InnoDB支持主外键，事务，行锁，缓存索引和真实数据
MyISAM不支持主外键，不支持事务，锁是表锁，只缓存索引

索引:
索引是帮助MySQL高效获取数据的数据结构，索引本身也很大，往往以文件的形式存储在磁盘上
索引可以大大提高查询数据，同时会降低更新表的速度

索引类型:
create index idx_user_name on user(name)    ---单值索引 
create index idx_user_name on user(name,email)  ---复合索引
alter user add index idx_user_name on(name,email)   ---另一种写法
``` 
* explain分析 
```
* id 表的读取顺序：大的先执行。相同的顺序执行
* select_type 查询类型有6种：
    SIMPLE 简单select查询，不包含任何子查询或者union
    PRIMARY 查询中包含任何子查询，最外层查询则被标记为primary
    SUBQUERY select或where中的子查询
    DERIVED 衍生表
    UNION union后面的select语句会被标记为union
    UNION RESULT union查询的结果集
* table 表名
* type 访问类型排列 system > const > eq_ref > ref(尽可能达到ref类型) > range > index > ALL 
    system 表只有一行记录
    const 通过索引一次就找到了，比如     select * form t1 where id=1
    eq_ref 单条匹配 只有一条匹配成功比如 select * form t1,t2 where t1.id = t2.id
    ref    多条匹配 有多条匹配成功       select * form t1,t2 where t1.id = t2.id
    range  检索指定范围的行              select * form t1 where id in(1,2,3)
    index  全索引扫描，走了索引
    all    全数据文件扫描，没走索引
* possible_keys 可能用到的索引
* key           实际使用的索引
* key_len       索引字节数，越小越好
* ref           查询实际值
* rows          预计要查找多少行
* extra 额外信息
    Using filesort 无法使用索引进行排序，使用文件排序（中间兄弟不能断）
    Using temporary 用了临时表保存数据，常见于order by group by
    Using index 表示相应的select操作使用了覆盖索引（上面两种情况是倒霉了，这种情况是好的）
```
* 是否适合建立索引
```
需要：
1.主键自动创建唯一索引，外键需要
2.查询，排序，统计，分组字段
3.左关联索引加右表，右关联索引加左表（左表固定，查询速度取决于右表。尽量用小结果集驱动大的结果集）
不适合：
1.频繁更新的字段不适合创建索引（每次更新都要更新索引）
2.大量重复的字段
```
* 表建立索引之后，SQL优化原则
```
* 尽量覆盖索引
* 最左原则：带头大哥不能死，中间兄弟不能断
    (不要在索引列上做任何操作(计算，函数，自动转换(字符串不加单引号)))
    (范围查询之后也会失效，包括!= isnull isnotnull or关键字也会)
* like '%xxx' '%xxx%' 会索引失效，这种'xxx%'会好一点。或者用覆盖索引也行
* 减少使用 *，查询索引列的字段会更快

* order by group by不会自动优化顺序
* 小表驱动大表in 大表驱动小表exits
```
select * from A where id in (select id from B)
select * from A where exits (select 1 from B where A.id=B.id)
```
```
## 查询截取分析
* 设置阈值，抓取慢SQL
* explain
* show profile
* DBA 数据库服务器相关参数调优

## MySQL锁机制
## 主从复制
