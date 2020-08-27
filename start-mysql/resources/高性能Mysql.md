# 高性能MySQL(第三版)
## 第一章 MySQL架构与历史
### 1.1 MySQL逻辑架构
* MySQL最重要的特性是它的存储引擎架构,这种架构将数据查询,存储及其他系统任务相分离.这种设计可以根据性能特性等需求灵活选择存储方式
* 

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













# MySQL高级
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
    