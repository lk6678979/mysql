# MERGE引擎分表
## 一、使用场景
* 单表数据很大，超过千万
* 不需要事务操作
## 二、数据库测试版本
* 5.7
## 三、注意事项
* 分表必须使用MyISAM存储引擎；
* 每个分表和总表的表结构必须相同，包括索引、字段类型、引擎和字符集
* MySQL必须具有存储分表数据文件和索引文件的目录的读写权限；
* 总表必须使用MRG_MyISAM(或者叫MERGE)存储引擎；
* 总表不会创建任何数据文件和索引文件。Merge引擎下每一张表只有一个MRG文件。MRG里面存放着分表的关系，以及插入数据的方式。它就像是一个外壳，或者是连接池，数据存放在分表里面。
## 四、建表
### 4.1 分表
1.用户1表
```
CREATE TABLE `user1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `sex` int(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8; 
```
2.用户2表
```
create table user2 like user1;
```
### 4.2 主表
```
CREATE TABLE `alluser` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `sex` int(1) NOT NULL DEFAULT '0',
  KEY `id` (`id`)
) ENGINE=MRG_MyISAM DEFAULT CHARSET=utf8 INSERT_METHOD=no UNION=(`user1`,`user2`);
```
* ENGINE = MERGE 和 ENGINE = MRG_MyISAM,都是代表使用的存储引擎是 Merge。
* INSERT_METHOD：数据插入方式  
  no：不允许插入
  LAST：添加数据的时候插入到最后一个表，这里就是user2
  FIRST 添加数据的时候插入到第一个表，这里就是user1
  
### 4.3 修改分区表
```
ALTER TABLE alluser UNION(`user1`,`user2`,`user3`);
```
## 五、操作
### 5.1
* 先在user1表中增加一条数据，然后再在user2表中增加一条数据，查看 alluser中的数据。
```
insert into user1(name,sex) values ('张三',1);
insert into user2(name,sex) values ('李四',2);
select * from alluser;
```
结果：
|id|name|sex|
|-|-|-|
|1||张三|1|
|1|李四|2|




  
