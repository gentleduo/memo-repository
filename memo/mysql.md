

# 索引底层数据结构

![imag][1]



# Explain工具

使用EXPLAIN关键字可以模拟优化器执行SQL语句，分析SQL语句的性能瓶颈。在select语句之前增加explain关键字，MySQL会在查询上设置一个标记，执行查询会返回执行计划的信息，而不是执行这条SQL

示例表

```mysql
DROP TABLE IF EXISTS `actor`;
CREATE TABLE `actor` (
`id` int(11) NOT NULL,
`name` varchar(45) DEFAULT NULL,
`update_time` datetime DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `actor` (`id`, `name`, `update_time`) VALUES (1,'a','2017-12-22 15:27:18'), (2,'b','2017-12-22 15:27:18'), (3,'c','2017-12-22 15:27:18');

DROP TABLE IF EXISTS `film`;
CREATE TABLE `film` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(10) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `film` (`id`, `name`) VALUES (3,'film0'),(1,'film1'),(2,'film2');

DROP TABLE IF EXISTS `film_actor`;
CREATE TABLE `film_actor` (
`id` int(11) NOT NULL,
`film_id` int(11) NOT NULL,
`actor_id` int(11) NOT NULL,
`remark` varchar(255) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_film_actor_id` (`film_id`,`actor_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `film_actor` (`id`, `film_id`, `actor_id`) VALUES (1,1,1),(2,1,2),(3,2,1);
```



## Explain中的列

### 1、id列

id列的编号是 select 的序列号，有几个 select 就有几个id，并且id的顺序是按 select 出现的顺序增长的。 id列越大执行优先级越高，id相同则从上往下执行，id为NULL最后执行。

### 2、select_type列

1. simple：简单查询。查询不包含子查询或者union
2. primary：复杂查询中最外层的 select
3. subquery：包含在 select或where 中的子查询（不在 from 子句中）
4. derived：包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为派生表（derived的英文含 义）
5. union：在 union 中的第二个和随后的 select

实验验证：

```mysql
mysql> set session optimizer_switch='derived_merge=off'; #关闭mysql5.7新特性对衍生表的合并优化
Query OK, 0 rows affected (0.00 sec)

mysql> explain select (select 1 from actor where id = 1) from (select * from film where id = 1) der;
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | <derived3> | NULL       | system | NULL          | NULL    | NULL    | NULL  |    1 |   100.00 | NULL        |
|  3 | DERIVED     | film       | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  2 | SUBQUERY    | actor      | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | Using index |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)

mysql> set session optimizer_switch='derived_merge=on'; #还原默认配置
Query OK, 0 rows affected (0.00 sec)

mysql> explain select 1 union all select 1;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | PRIMARY     | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used |
|  2 | UNION       | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
2 rows in set, 1 warning (0.00 sec)

```

### 3、table列

显示这一行的数据是关于哪张表的。当from子句中有子查询时，table列是<derivenN>格式，表示当前查询依赖id=N的查询，于是先执行id=N的查询。当有union时，UNION RESULT的table列的值为<union1,2>，1和2表示参与union的select行id。

### 4、partitions

如果查询是基于分区表的话, 会显示查询访问的分区

### 5、type列

这一列表示关联类型或访问类型，即MySQL决定如何查找表中的行，查找数据行记录的大概范围。依次从最优到最差分别为：system > const > eq_ref > ref > range > index > ALL。一般来说，得保证查询达到range级别，最好达到ref

#### NULL

mysql能够在优化阶段分解查询语句时便获取结果，在执行阶段用不着再访问表或索引。例如：在索引列中选取最小值，可以单独查找索引来完成，不需要在执行时访问表

实验验证：

```mysql
mysql> explain select min(id) from film;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)

```

#### const, system

mysql能对查询的某部分进行优化并将其转化成一个常量。用于 primary key 或 unique key 的所有列与常数比较时，所以表最多有一个匹配行，读取1次，速度比较快。system是 const的特例，当表中有一行记录(系统表)时为system

#### eq_ref

primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合条件的记录。

实验验证：

```mysql
mysql> explain select * from film_actor left join film on film_actor.film_id = film.id;
+----+-------------+------------+------------+--------+---------------+---------+---------+----------------------------+------+----------+-------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref                        | rows | filtered | Extra |
+----+-------------+------------+------------+--------+---------------+---------+---------+----------------------------+------+----------+-------+
|  1 | SIMPLE      | film_actor | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                       |    3 |   100.00 | NULL  |
|  1 | SIMPLE      | film       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | test_db.film_actor.film_id |    1 |   100.00 | NULL  |
+----+-------------+------------+------------+--------+---------------+---------+---------+----------------------------+------+----------+-------+
2 rows in set, 1 warning (0.01 sec)

```

#### ref

相比eq_ref，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会 找到多个符合条件的行。

实验验证：

```mysql
#简单 select 查询，name是普通索引（非唯一索引）
mysql> explain select * from film where name = 'film1';
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | film  | NULL       | ref  | idx_name      | idx_name | 33      | const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

#关联表查询，idx_film_actor_id是film_id和actor_id的联合索引，这里使用到了film_actor的左边前缀film_id部分。
mysql> explain select film_id from film left join film_actor on film.id = film_actor.film_id;
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+-----------------+------+----------+-------------+
| id | select_type | table      | partitions | type  | possible_keys     | key               | key_len | ref             | rows | filtered | Extra       |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+-----------------+------+----------+-------------+
|  1 | SIMPLE      | film       | NULL       | index | NULL              | idx_name          | 33      | NULL            |    3 |   100.00 | Using index |
|  1 | SIMPLE      | film_actor | NULL       | ref   | idx_film_actor_id | idx_film_actor_id | 4       | test_db.film.id |    1 |   100.00 | Using index |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+-----------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)

```

#### range

范围扫描通常出现在 in(), between ,> ,<, >= 等操作中。使用一个索引来检索给定范围的行。

#### index

扫描全索引就能拿到结果，一般是扫描某个二级索引，这种扫描不会从索引树根节点开始快速查找，而是直接对二级索引的叶子节点遍历和扫描，速度还是比较慢的。这种查询一般为使用覆盖索引，二级索引一般比较小，所以这种通常比ALL快一些。

#### all

即全表扫描，扫描你的聚簇索引的所有叶子节点。通常情况下这需要增加索引来进行优化了。

### 6、possible_keys列

显示可能应用在这张表中的索引,一个或者多个；查询涉及到的字段上若存在索引,则该索引将被列出,但不一定被查询实际使用；

### 7、key列

实际使用的索引,如果为NULL,则没有使用索引；如果想强制mysql使用或忽视某个索引，在查询中使用force index、ignore index。

### 8、key_len列

这一列显示了mysql在索引里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列。举例来说，film_actor的联合索引 idx_film_actor_id 由 film_id 和 actor_id 两个int列组成，并且每个int是4字节。通 过结果中的key_len=4可推断出查询使用了第一个列：film_id列来执行索引查找。

key_len计算规则如下：

1. 字符串：char(n)和varchar(n)，5.0.3以后版本中，n均代表字符数，而不是字节数，如果是utf-8，一个数字或字母占1个字节，一个汉字占3个字节
   1. char(n)：如果存汉字长度就是3n字节
   2. varchar(n)：如果存汉字则长度是3n + 2字节，加的2字节用来存储字符串长度，因为varchar是变长字符串
2. 数值类型
   1. tinyint：1字节
   2. smallint：2字节
   3. int：4字节
   4. bigint：8字节
3. 时间类型
   1. date：3字节
   2. timestamp：4字节
   3. datetime：8字节

如果字段允许为NULL，需要1字节记录是否为NULL。索引的最大长度是768字节，当字符串过长时，mysql会做一个类似左前缀索引的处理，将前半部分的字符提取出来做索引。

### 9、ref列

这一列显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有：const（常量），字段名（例：film.id）

### 10、rows列

这一列是mysql估计要读取并检测的行数。

### 11、filtered列

是一个百分比的值，rows * filtered / 100可以估算出将要和explain中前一个表进行连接的行数（前一个表指explain中的id值比当前表id值小的表）。

### 12、Extra

#### Using index

使用覆盖索引，如果select后面查询的字段都可以从这个索引的树中获取，这种情况一般可以说是用到了覆盖索引，extra里一般都有using index；覆盖索引一般针对的是辅助索引，整个查询结果只通过辅助索引就能拿到结果，不需要通过辅助索引树找到主键，再通过主键去主键索引树里获取其它字段值

#### Using where

使用where语句来处理结果，并且查询的列未被索引覆盖（全表扫描）

#### Using index condition

查询的列不完全被索引覆盖，where条件中是一个前导列的范围

#### Using temporary

mysql需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化

#### Using filesort

将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的

#### Select tables optimized away

使用某些聚合函数（比如 max、min）来访问存在索引的某个字段。（mysql能够在优化阶段分解查询语句时便获取结果）



# 索引优化

示例表

```mysql
DROP TABLE IF EXISTS `employees`;
CREATE TABLE `employees` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
`age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
`position` varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
`hire_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
PRIMARY KEY (`id`),
KEY `idx_name_age_position` (`name`,`age`,`position`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='员工记录表';

INSERT INTO employees(name,age,position,hire_time) VALUES('LiLei',22,'manager',NOW());
INSERT INTO employees(name,age,position,hire_time) VALUES('HanMeimei', 23,'dev',NOW());
INSERT INTO employees(name,age,position,hire_time) VALUES('Lucy',23,'dev',NOW());

‐‐ 插入一些示例数据
drop procedure if exists insert_emp;
delimiter ;;
create procedure insert_emp()
begin
declare i int;
set i=1;
while(i<=100000)do
insert into employees(name,age,position) values(CONCAT('zhuge',i),i,'dev');
set i=i+1;
end while;
end;;
delimiter ;
call insert_emp();
```



## 1、联合索引第一个字段用范围不会走索引

mysql> EXPLAIN SELECT * FROM employees WHERE name > 'LiLei' AND age = 22 AND position ='manager';
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys         | key  | key_len | ref  | rows  | filtered | Extra       |
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-------------+
|  1 | SIMPLE      | employees | NULL       | ALL  | idx_name_age_position | NULL | NULL    | NULL | 99918 |     0.50 | Using where |
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

结论：联合索引第一个字段就用范围查找不会走索引，mysql内部可能觉得第一个字段就用范围，结果集应该很大，回表效率不高，还不如就全表扫描



## 2、强制走索引

mysql> EXPLAIN SELECT * FROM employees force index(idx_name_age_position) WHERE name > 'LiLei' AND age = 22 AND position ='manager';
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+-----------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows  | filtered | Extra                 |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 74      | NULL | 49959 |     1.00 | Using index condition |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

结论：虽然使用了强制走索引让联合索引第一个字段范围查找也走索引，扫描的行rows看上去也少了点，但是最终查找效率不一定比全表 扫描高，因为回表效率不高

实验验证：

```mysql
‐‐ 关闭查询缓存
set global query_cache_size=0;
set global query_cache_type=0;
‐‐ 执行时间0.333s
SELECT * FROM employees WHERE name > 'LiLei';
‐‐ 执行时间0.444s
SELECT * FROM employees force index(idx_name_age_position) WHERE name > 'LiLei';
```



## 3、in和or在表数据量比较大的情况会走索引，在表记录不多的情况下会选择全表扫描

mysql> EXPLAIN SELECT * FROM employees WHERE name in ('LiLei','HanMeimei','Lucy') AND age = 22 AND position='manager';
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 140     | NULL |    3 |   100.00 | Using index condition |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT * FROM employees WHERE (name = 'LiLei' or name = 'HanMeimei') AND age = 22 AND position = 'manager';
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 140     | NULL |    2 |   100.00 | Using index condition |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

实验验证

将employees 表复制一张employees_copy的表，里面保留两三条记录

```mysql
DROP TABLE IF EXISTS `employees_copy`;
CREATE TABLE `employees_copy` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
`age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
`position` varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
`hire_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
PRIMARY KEY (`id`),
KEY `idx_name_age_position` (`name`,`age`,`position`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='员工记录表';

INSERT INTO employees_copy(name,age,position,hire_time) VALUES('LiLei',22,'manager',NOW());
INSERT INTO employees_copy(name,age,position,hire_time) VALUES('HanMeimei', 23,'dev',NOW());
INSERT INTO employees_copy(name,age,position,hire_time) VALUES('Lucy',23,'dev',NOW());
```

mysql> EXPLAIN SELECT * FROM employees_copy WHERE name in ('LiLei','HanMeimei','Lucy') AND age = 22 AND position = 'manager';
+----+-------------+----------------+------------+------+-----------------------+------+---------+------+------+----------+-------------+
| id | select_type | table          | partitions | type | possible_keys         | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+----------------+------------+------+-----------------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | employees_copy | NULL       | ALL  | idx_name_age_position | NULL | NULL    | NULL |    3 |   100.00 | Using where |
+----+-------------+----------------+------------+------+-----------------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT * FROM employees_copy WHERE (name = 'LiLei' or name = 'HanMeimei') AND age = 22 AND position = 'manager';
+----+-------------+----------------+------------+------+-----------------------+------+---------+------+------+----------+-------------+
| id | select_type | table          | partitions | type | possible_keys         | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+----------------+------------+------+-----------------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | employees_copy | NULL       | ALL  | idx_name_age_position | NULL | NULL    | NULL |    3 |    66.67 | Using where |
+----+-------------+----------------+------------+------+-----------------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)



## 4、like KK% 一般情况都会走索引（索引下推）

mysql> EXPLAIN SELECT * FROM employees WHERE name like 'LiLei%' AND age = 22 AND position ='manager';
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 140     | NULL |    1 |     5.00 | Using index condition |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

这里给引入一个概念，索引下推（Index Condition Pushdown，ICP）, like KK%其实就是用到了索引下推优化
那什么是索引下推了？对于辅助的联合索引(name,age,position)，正常情况按照最左前缀原则，SELECT * FROM employees WHERE name like 'LiLei%' AND age = 22 AND position ='manager' 这种情况只会走name字段索引，因为根据name字段过滤完，得到的索引行里的age和position是无序的，无法很好的利用索引。
在MySQL5.6之前的版本，这个查询只能在联合索引里匹配到名字是'LiLei'开头的索引，然后拿这些索引对应的主键逐个回表，到主键索引上找出相应的记录，再比对age和position这两个字段的值是否符合。MySQL5.6引入了索引下推优化，可以在索引遍历过程中，对索引中包含的所有字段先做判断，过滤掉不符合条件的记录之后再回表，可以有效的减少回表次数。使用了索引下推优化后，上面那个查询在联合索引里匹配到名字是'LiLei'开头的索引之后，同时还会在索引里过滤age和position这两个字段，拿着过滤完剩下的索引对应的主键id再回表查整行数据。索引下推会减少回表次数，对于innodb引擎的表索引下推只能用于二级索引，innodb的主键索引（聚簇索引）树叶子节点上保存的是全行数据，所以这个时候索引下推并不会起到减少查询全行数据的效果。

为什么联合索引中范围条件右边的列没有用索引下推优化？ 估计应该是Mysql认为范围查找过滤的结果集过大，like KK% 在绝大多数情况来看，过滤后的结果集比较小，所以这里Mysql选择给 like KK% 用了索引下推优化，当然这也不是绝对的，有时like KK% 也不一定就会走索引下推。



## 5、Mysql如何选择合适的索引

mysql> EXPLAIN select * from employees where name > 'a';
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys         | key  | key_len | ref  | rows  | filtered | Extra       |
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-------------+
|  1 | SIMPLE      | employees | NULL       | ALL  | idx_name_age_position | NULL | NULL    | NULL | 99918 |    50.00 | Using where |
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

如果用name索引需要遍历name字段联合索引树，然后还需要根据遍历出来的主键值去主键索引树里再去查出最终数据，成本比全表扫描还高，可以用覆盖索引优化，这样只需要遍历name字段的联合索引树就能拿到所有结果，如下：

mysql> EXPLAIN select name,age,position from employees where name > 'a' ;
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+--------------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows  | filtered | Extra                    |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+--------------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 74      | NULL | 49959 |   100.00 | Using where; Using index |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+--------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> EXPLAIN select * from employees where name > 'zzz' ;
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 74      | NULL |    1 |   100.00 | Using index condition |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

对于上面这两种 name>'a' 和 name>'zzz' 的执行结果，mysql最终是否选择走索引或者一张表涉及多个索引，mysql最 终如何选择索引，可以用trace工具来一查究竟，开启trace工具会影响mysql性能，所以只能临时分析sql使用，用 完之后立即关闭

trace工具用法：

```mysql
mysql> set session optimizer_trace="enabled=on",end_markers_in_json=on; -- 开启trace
mysql> select * from employees where name > 'a' order by position;
mysql> SELECT * FROM information_schema.OPTIMIZER_TRACE;

select * from employees where name > 'a' order by position | {
  "steps": [
    {
      "join_preparation": {  -- 第一阶段：SQL准备阶段，格式化sql
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `employees`.`id` AS `id`,`employees`.`name` AS `name`,`employees`.`age` AS `age`,`employees`.`position` AS `position`,`employees`.`hire_time` AS `hire_time` from `employees` where (`employees`.`name` > 'a') order by `employees`.`position`"
          }
        ] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": {  ‐‐ 第二阶段：SQL优化阶段
        "select#": 1,
        "steps": [
          {
            "condition_processing": {  -- 条件处理
              "condition": "WHERE",
              "original_condition": "(`employees`.`name` > 'a')",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                }
              ] /* steps */
            } /* condition_processing */
          },
          {
            "substitute_generated_columns": {
            } /* substitute_generated_columns */
          },
          {
            "table_dependencies": [  -- 表依赖详情
              {
                "table": "`employees`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ] /* depends_on_map_bits */
              }
            ] /* table_dependencies */
          },
          {
            "ref_optimizer_key_uses": [
            ] /* ref_optimizer_key_uses */
          },
          {
            "rows_estimation": [  -- 预估表的访问成本
              {
                "table": "`employees`",
                "range_analysis": {
                  "table_scan": {  -- 全表扫描情况
                    "rows": 99918,  -- 扫描行数
                    "cost": 20275  -- 查询成本
                  } /* table_scan */,
                  "potential_range_indexes": [  -- 查询可能使用的索引
                    {
                      "index": "PRIMARY",  -- 主键索引
                      "usable": false,
                      "cause": "not_applicable"
                    },
                    {
                      "index": "idx_name_age_position",  -- 辅助索引
                      "usable": true,
                      "key_parts": [
                        "name",
                        "age",
                        "position",
                        "id"
                      ] /* key_parts */
                    }
                  ] /* potential_range_indexes */,
                  "setup_range_conditions": [
                  ] /* setup_range_conditions */,
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  } /* group_index_range */,
                  "analyzing_range_alternatives": {  -- 分析各个索引使用成本
                    "range_scan_alternatives": [
                      {
                        "index": "idx_name_age_position",
                        "ranges": [
                          "a < name"  -- 索引使用范围
                        ] /* ranges */,
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": false,  -- 使用该索引获取的记录是否按照主键排序
                        "using_mrr": false,
                        "index_only": false,  -- 是否使用覆盖索引
                        "rows": 49959,  -- 索引扫描行数
                        "cost": 59952,  -- 索引扫描成本
                        "chosen": false,  -- 是否选择该索引
                        "cause": "cost"
                      }
                    ] /* range_scan_alternatives */,
                    "analyzing_roworder_intersect": {
                      "usable": false,
                      "cause": "too_few_roworder_scans"
                    } /* analyzing_roworder_intersect */
                  } /* analyzing_range_alternatives */
                } /* range_analysis */
              }
            ] /* rows_estimation */
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ] /* plan_prefix */,
                "table": "`employees`",
                "best_access_path": {  -- 最优访问路径
                  "considered_access_paths": [
                    {
                      "rows_to_scan": 99918,
                      "access_type": "scan",  -- 访问类型：为scan，全表扫描
                      "resulting_rows": 99918,
                      "cost": 20273,
                      "chosen": true,  -- 确定选择
                      "use_tmp_table": true
                    }
                  ] /* considered_access_paths */
                } /* best_access_path */,
                "condition_filtering_pct": 100,
                "rows_for_plan": 99918,
                "cost_for_plan": 20273,
                "sort_cost": 99918,
                "new_cost_for_plan": 120191,
                "chosen": true
              }
            ] /* considered_execution_plans */
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "(`employees`.`name` > 'a')",
              "attached_conditions_computation": [
              ] /* attached_conditions_computation */,
              "attached_conditions_summary": [
                {
                  "table": "`employees`",
                  "attached": "(`employees`.`name` > 'a')"
                }
              ] /* attached_conditions_summary */
            } /* attaching_conditions_to_tables */
          },
          {
            "clause_processing": {
              "clause": "ORDER BY",
              "original_clause": "`employees`.`position`",
              "items": [
                {
                  "item": "`employees`.`position`"
                }
              ] /* items */,
              "resulting_clause_is_simple": true,
              "resulting_clause": "`employees`.`position`"
            } /* clause_processing */
          },
          {
            "reconsidering_access_paths_for_index_ordering": {
              "clause": "ORDER BY",
              "steps": [
              ] /* steps */,
              "index_order_summary": {
                "table": "`employees`",
                "index_provides_order": false,
                "order_direction": "undefined",
                "index": "unknown",
                "plan_changed": false
              } /* index_order_summary */
            } /* reconsidering_access_paths_for_index_ordering */
          },
          {
            "refine_plan": [
              {
                "table": "`employees`"
              }
            ] /* refine_plan */
          }
        ] /* steps */
      } /* join_optimization */
    },
    {
      "join_execution": {  -- 第三阶段：SQL执行阶段
        "select#": 1,
        "steps": [
          {
            "filesort_information": [
              {
                "direction": "asc",
                "table": "`employees`",
                "field": "position"
              }
            ] /* filesort_information */,
            "filesort_priority_queue_optimization": {
              "usable": false,
              "cause": "not applicable (no LIMIT)"
            } /* filesort_priority_queue_optimization */,
            "filesort_execution": [
            ] /* filesort_execution */,
            "filesort_summary": {
              "rows": 100003,
              "examined_rows": 100003,
              "number_of_tmp_files": 30,
              "sort_buffer_size": 262056,
              "sort_mode": "<sort_key, packed_additional_fields>"
            } /* filesort_summary */
          }
        ] /* steps */
      } /* join_execution */
    }
  ] /* steps */
}

-- 结论：全表扫描的成本低于索引扫描，所以mysql最终选择全表扫描。（由cost来决定最终选哪种方式）
mysql> select * from employees where name > 'zzz' order by position;
mysql> SELECT * FROM information_schema.OPTIMIZER_TRACE;

-- 查看trace字段可知索引扫描的成本低于全表扫描，所以mysql最终选择索引扫描

mysql> set session optimizer_trace="enabled=off"; -- 关闭trace;

```



## 6、索引深入优化

### Case1：

```mysql
mysql> explain select * from employees where name = 'Lilei' and position = 'dev' order by age;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |    10.00 | Using index condition |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```

分析： 利用最左前缀法则：中间字段不能断，因此查询用到了name索引，从key_len=74也能看出，age索引列用 在排序过程中，因为Extra字段里没有using filesort。（复合索引中某一列能否用到索引取决于该列的数据在通过前面的索引列过滤出来后是否有序，例如此例中通过name='Lilei'过滤出的数据中age是有序的所以可以将age索引列用在排序过程中，即在name固定的情况下age是有序的）

### Case 2：

```mysql
mysql> explain select * from employees where name = 'Lilei' order by position;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |   100.00 | Using index condition; Using filesort |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

分析：从explain的执行结果来看：key_len=74，查询使用了name索引，由于用了position进行排序，跳过了 age，出现了Using filesort。

### Case 3：

```mysql
mysql> explain select * from employees where name = 'Lilei' order by age,position;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |   100.00 | Using index condition |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```

分析：查找只用到索引name，age和position用于排序，无Using filesort。

### Case 4：

```mysql
mysql> explain select * from employees where name = 'Lilei' order by position,age;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |   100.00 | Using index condition; Using filesort |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

分析：和Case 3中explain的执行结果一样，但是出现了Using filesort，因为索引的创建顺序为name,age,position，但是排序的时候age和position颠倒位置了。

### Case 5：

```mysql
mysql> explain select * from employees where name = 'Lilei' and age = 18 order by position,age;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-----------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref         | rows | filtered | Extra                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 78      | const,const |    1 |   100.00 | Using index condition |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```

分析：与Case 4对比，在Extra中并未出现Using filesort，因为age为常量，在排序中被优化，所以索引未颠倒，不会出现Using filesort。

### Case 6：

```mysql
mysql> explain select * from employees where name = 'Lilei' order by age asc,position desc;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |   100.00 | Using index condition; Using filesort |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

分析：虽然排序的字段列与索引顺序一样，且order by默认升序，这里position desc变成了降序，导致与索引的排序方式不同，从而产生Using filesort。Mysql8以上版本有降序索引可以支持该种查询方式。

### Case 7：

```mysql
mysql> explain select * from employees where name in ( 'Lilei', 'zhuge' ) order by age,position;
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 74      | NULL |    2 |   100.00 | Using index condition; Using filesort |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+---------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

分析：对于排序来说，多个相等条件也是范围查询，导致后面的索引失效，从而产生Using filesort。

### 优化总结：

1. MySQL支持两种方式的排序filesort和index，Using index是指MySQL扫描索引本身完成排序。index 效率高，filesort效率低。
2. order by满足两种情况会使用Using index。
   1. order by语句使用索引最左前列。
   2. 使用where子句与order by子句条件列组合满足索引最左前列
3. 尽量在索引列上完成排序，遵循索引建立（索引创建的顺序）时的最左前缀法则。
4. 如果order by的条件不在索引列上，就会产生Using filesort。
5. 能用覆盖索引尽量用覆盖索引
6. group by与order by很类似，其实质是先排序后分组，遵照索引创建顺序的最左前缀法则。对于group by的优化如果不需要排序的可以加上order by null禁止排序。注意，where高于having，能写在where中 的限定条件就不要去having限定了。



## 7、Using filesort文件排序原理详解

### filesort文件排序方式

1. 单路排序：是一次性取出满足条件行的所有字段，然后在sort buffer中进行排序；用trace工具可 以看到sort_mode信息里显示< sort_key, additional_fields >或者< sort_key, packed_additional_fields >
2. 双路排序（又叫回表排序模式）：是首先根据相应的条件取出相应的排序字段和可以直接定位行 数据的行 ID，然后在 sort buffer 中进行排序，排序完后需要再次取回其它需要的字段；用trace工具 可以看到sort_mode信息里显示< sort_key, rowid >

MySQL 通过比较系统变量 max_length_for_sort_data(默认1024字节) 的大小和需要查询的字段总大小来 判断使用哪种排序模式。

1. 如果 字段的总长度小于max_length_for_sort_data ，那么使用 单路排序模式；
2. 如果 字段的总长度大于max_length_for_sort_data ，那么使用 双路排序模∙式。

```mysql
mysql> set session optimizer_trace="enabled=on",end_markers_in_json=on; -- 开启trace
mysql>  select * from employees where name like 'zhuge%' order by position;
mysql> select * from information_schema.OPTIMIZER_TRACE;

select * from employees where name like 'zhuge%' order by position | {
  "steps": [
    {
      "join_preparation": {
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `employees`.`id` AS `id`,`employees`.`name` AS `name`,`employees`.`age` AS `age`,`employees`.`position` AS `position`,`employees`.`hire_time` AS `hire_time` from `employees` where (`employees`.`name` like 'zhuge%') order by `employees`.`position`"
          }
        ] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": {
        "select#": 1,
        "steps": [
          {
            "condition_processing": {
              "condition": "WHERE",
              "original_condition": "(`employees`.`name` like 'zhuge%')",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`employees`.`name` like 'zhuge%')"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`employees`.`name` like 'zhuge%')"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "(`employees`.`name` like 'zhuge%')"
                }
              ] /* steps */
            } /* condition_processing */
          },
          {
            "substitute_generated_columns": {
            } /* substitute_generated_columns */
          },
          {
            "table_dependencies": [
              {
                "table": "`employees`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ] /* depends_on_map_bits */
              }
            ] /* table_dependencies */
          },
          {
            "ref_optimizer_key_uses": [
            ] /* ref_optimizer_key_uses */
          },
          {
            "rows_estimation": [
              {
                "table": "`employees`",
                "range_analysis": {
                  "table_scan": {
                    "rows": 94465,
                    "cost": 19184
                  } /* table_scan */,
                  "potential_range_indexes": [
                    {
                      "index": "PRIMARY",
                      "usable": false,
                      "cause": "not_applicable"
                    },
                    {
                      "index": "idx_name_age_position",
                      "usable": true,
                      "key_parts": [
                        "name",
                        "age",
                        "position",
                        "id"
                      ] /* key_parts */
                    }
                  ] /* potential_range_indexes */,
                  "setup_range_conditions": [
                  ] /* setup_range_conditions */,
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  } /* group_index_range */,
                  "analyzing_range_alternatives": {
                    "range_scan_alternatives": [
                      {
                        "index": "idx_name_age_position",
                        "ranges": [
                          "zhuge\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000 <= name <= zhuge▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒"
                        ] /* ranges */,
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": false,
                        "using_mrr": false,
                        "index_only": false,
                        "rows": 47232,
                        "cost": 56679,
                        "chosen": false,
                        "cause": "cost"
                      }
                    ] /* range_scan_alternatives */,
                    "analyzing_roworder_intersect": {
                      "usable": false,
                      "cause": "too_few_roworder_scans"
                    } /* analyzing_roworder_intersect */
                  } /* analyzing_range_alternatives */
                } /* range_analysis */
              }
            ] /* rows_estimation */
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ] /* plan_prefix */,
                "table": "`employees`",
                "best_access_path": {
                  "considered_access_paths": [
                    {
                      "rows_to_scan": 94465,
                      "access_type": "scan",
                      "resulting_rows": 94465,
                      "cost": 19182,
                      "chosen": true,
                      "use_tmp_table": true
                    }
                  ] /* considered_access_paths */
                } /* best_access_path */,
                "condition_filtering_pct": 100,
                "rows_for_plan": 94465,
                "cost_for_plan": 19182,
                "sort_cost": 94465,
                "new_cost_for_plan": 113647,
                "chosen": true
              }
            ] /* considered_execution_plans */
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "(`employees`.`name` like 'zhuge%')",
              "attached_conditions_computation": [
              ] /* attached_conditions_computation */,
              "attached_conditions_summary": [
                {
                  "table": "`employees`",
                  "attached": "(`employees`.`name` like 'zhuge%')"
                }
              ] /* attached_conditions_summary */
            } /* attaching_conditions_to_tables */
          },
          {
            "clause_processing": {
              "clause": "ORDER BY",
              "original_clause": "`employees`.`position`",
              "items": [
                {
                  "item": "`employees`.`position`"
                }
              ] /* items */,
              "resulting_clause_is_simple": true,
              "resulting_clause": "`employees`.`position`"
            } /* clause_processing */
          },
          {
            "reconsidering_access_paths_for_index_ordering": {
              "clause": "ORDER BY",
              "steps": [
              ] /* steps */,
              "index_order_summary": {
                "table": "`employees`",
                "index_provides_order": false,
                "order_direction": "undefined",
                "index": "unknown",
                "plan_changed": false
              } /* index_order_summary */
            } /* reconsidering_access_paths_for_index_ordering */
          },
          {
            "refine_plan": [
              {
                "table": "`employees`"
              }
            ] /* refine_plan */
          }
        ] /* steps */
      } /* join_optimization */
    },
    {
      "join_execution": {  -- ‐Sql执行阶段
        "select#": 1,
        "steps": [
          {
            "filesort_information": [
              {
                "direction": "asc",
                "table": "`employees`",
                "field": "position"
              }
            ] /* filesort_information */,
            "filesort_priority_queue_optimization": {
              "usable": false,
              "cause": "not applicable (no LIMIT)"
            } /* filesort_priority_queue_optimization */,
            "filesort_execution": [
            ] /* filesort_execution */,
            "filesort_summary": {  -- 文件排序信息
              "rows": 100000,  -- 预计扫描行数
              "examined_rows": 100003,  -- 参与排序的行
              "number_of_tmp_files": 30,  -- ‐使用临时文件的个数，这个值如果为0代表全部使用的sort_buffer内存排序，否则使用的磁盘文件排序
              "sort_buffer_size": 262056,  -- 排序缓存的大小，单位Byte
              "sort_mode": "<sort_key, packed_additional_fields>"  -- 排序方式，这里用的单路排序
            } /* filesort_summary */
          }
        ] /* steps */
      } /* join_execution */
    }
  ] /* steps */
} 

mysql> set max_length_for_sort_data = 10;  -- employees表所有字段长度总和肯定大于10字节
mysql> select * from employees where name like 'zhuge%' order by position;
mysql> select * from information_schema.OPTIMIZER_TRACE;

select * from employees where name like 'zhuge%' order by position | {
  "steps": [
    {
      "join_preparation": {
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `employees`.`id` AS `id`,`employees`.`name` AS `name`,`employees`.`age` AS `age`,`employees`.`position` AS `position`,`employees`.`hire_time` AS `hire_time` from `employees` where (`employees`.`name` like 'zhuge%') order by `employees`.`position`"
          }
        ] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": {
        "select#": 1,
        "steps": [
          {
            "condition_processing": {
              "condition": "WHERE",
              "original_condition": "(`employees`.`name` like 'zhuge%')",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`employees`.`name` like 'zhuge%')"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`employees`.`name` like 'zhuge%')"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "(`employees`.`name` like 'zhuge%')"
                }
              ] /* steps */
            } /* condition_processing */
          },
          {
            "substitute_generated_columns": {
            } /* substitute_generated_columns */
          },
          {
            "table_dependencies": [
              {
                "table": "`employees`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ] /* depends_on_map_bits */
              }
            ] /* table_dependencies */
          },
          {
            "ref_optimizer_key_uses": [
            ] /* ref_optimizer_key_uses */
          },
          {
            "rows_estimation": [
              {
                "table": "`employees`",
                "range_analysis": {
                  "table_scan": {
                    "rows": 94465,
                    "cost": 19184
                  } /* table_scan */,
                  "potential_range_indexes": [
                    {
                      "index": "PRIMARY",
                      "usable": false,
                      "cause": "not_applicable"
                    },
                    {
                      "index": "idx_name_age_position",
                      "usable": true,
                      "key_parts": [
                        "name",
                        "age",
                        "position",
                        "id"
                      ] /* key_parts */
                    }
                  ] /* potential_range_indexes */,
                  "setup_range_conditions": [
                  ] /* setup_range_conditions */,
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  } /* group_index_range */,
                  "analyzing_range_alternatives": {
                    "range_scan_alternatives": [
                      {
                        "index": "idx_name_age_position",
                        "ranges": [
                          "zhuge\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000 <= name <= zhuge▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒"
                        ] /* ranges */,
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": false,
                        "using_mrr": false,
                        "index_only": false,
                        "rows": 47232,
                        "cost": 56679,
                        "chosen": false,
                        "cause": "cost"
                      }
                    ] /* range_scan_alternatives */,
                    "analyzing_roworder_intersect": {
                      "usable": false,
                      "cause": "too_few_roworder_scans"
                    } /* analyzing_roworder_intersect */
                  } /* analyzing_range_alternatives */
                } /* range_analysis */
              }
            ] /* rows_estimation */
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ] /* plan_prefix */,
                "table": "`employees`",
                "best_access_path": {
                  "considered_access_paths": [
                    {
                      "rows_to_scan": 94465,
                      "access_type": "scan",
                      "resulting_rows": 94465,
                      "cost": 19182,
                      "chosen": true,
                      "use_tmp_table": true
                    }
                  ] /* considered_access_paths */
                } /* best_access_path */,
                "condition_filtering_pct": 100,
                "rows_for_plan": 94465,
                "cost_for_plan": 19182,
                "sort_cost": 94465,
                "new_cost_for_plan": 113647,
                "chosen": true
              }
            ] /* considered_execution_plans */
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "(`employees`.`name` like 'zhuge%')",
              "attached_conditions_computation": [
              ] /* attached_conditions_computation */,
              "attached_conditions_summary": [
                {
                  "table": "`employees`",
                  "attached": "(`employees`.`name` like 'zhuge%')"
                }
              ] /* attached_conditions_summary */
            } /* attaching_conditions_to_tables */
          },
          {
            "clause_processing": {
              "clause": "ORDER BY",
              "original_clause": "`employees`.`position`",
              "items": [
                {
                  "item": "`employees`.`position`"
                }
              ] /* items */,
              "resulting_clause_is_simple": true,
              "resulting_clause": "`employees`.`position`"
            } /* clause_processing */
          },
          {
            "reconsidering_access_paths_for_index_ordering": {
              "clause": "ORDER BY",
              "steps": [
              ] /* steps */,
              "index_order_summary": {
                "table": "`employees`",
                "index_provides_order": false,
                "order_direction": "undefined",
                "index": "unknown",
                "plan_changed": false
              } /* index_order_summary */
            } /* reconsidering_access_paths_for_index_ordering */
          },
          {
            "refine_plan": [
              {
                "table": "`employees`"
              }
            ] /* refine_plan */
          }
        ] /* steps */
      } /* join_optimization */
    },
    {
      "join_execution": {
        "select#": 1,
        "steps": [
          {
            "filesort_information": [
              {
                "direction": "asc",
                "table": "`employees`",
                "field": "position"
              }
            ] /* filesort_information */,
            "filesort_priority_queue_optimization": {
              "usable": false,
              "cause": "not applicable (no LIMIT)"
            } /* filesort_priority_queue_optimization */,
            "filesort_execution": [
            ] /* filesort_execution */,
            "filesort_summary": {
              "rows": 100000,
              "examined_rows": 100003,
              "number_of_tmp_files": 20,
              "sort_buffer_size": 262136,
              "sort_mode": "<sort_key, rowid>"  -- 排序方式，这里用的双路排序
            } /* filesort_summary */
          }
        ] /* steps */
      } /* join_execution */
    }
  ] /* steps */
}

mysql> set session optimizer_trace="enabled=off";  -- 关闭trace
```

### 单路排序的详细过程：

1. 从索引name找到第一个满足 name like ‘zhuge%’ 条件的主键 id
2. 根据主键 id 取出整行，取出所有字段的值，存入 sort_buffer 中
3. 从索引name找到下一个满足 name like  ‘zhuge%’ 条件的主键 id
4. 重复步骤 2、3 直到不满足 name like ‘zhuge%’
5. 对 sort_buffer 中的数据按照字段 position 进行排序
6. 返回结果给客户端

### 双路排序的详细过程：

1. 从索引 name 找到第一个满足 name like ‘zhuge%’ 的主键id
2. 根据主键 id 取出整行，把排序字段 position 和主键 id 这两个字段放到 sort buffer 中
3. 从索引 name 取下一个满足 name like ‘zhuge%’ 记录的主键 id
4.  重复 3、4 直到不满足 name like ‘zhuge%’
5. 对 sort_buffer 中的字段 position 和主键 id 按照字段 position 进行排序
6. 遍历排序好的 id 和字段 position，按照 id 的值回到原表中取出 所有字段的值返回给客户端

其实对比两个排序模式，单路排序会把所有需要查询的字段都放到 sort buffer 中，而双路排序只会把主键 和需要排序的字段放到 sort buffer 中进行排序，然后再通过主键回到原表查询需要的字段。 如果 MySQL 排序内存 sort_buffer 配置的比较小并且没有条件继续增加了，可以适当把 max_length_for_sort_data 配置小点，让优化器选择使用双路排序算法，可以在sort_buffer 中一次排序更 多的行，只是需要再根据主键回到原表取数据。 如果 MySQL 排序内存有条件可以配置比较大，可以适当增大 max_length_for_sort_data 的值，让优化器 优先选择全字段排序(单路排序)，把需要的字段放到 sort_buffer 中，这样排序后就会直接从内存里返回查 询结果了。 所以，MySQL通过 max_length_for_sort_data 这个参数来控制排序，在不同场景使用不同的排序模式， 从而提升排序效率。注意，如果全部使用sort_buffer内存排序一般情况下效率会高于磁盘文件排序，但不能因为这个就随便增 大sort_buffer(默认1M)，mysql很多参数设置都是做过优化的，不要轻易调整。



## 8、索引设计原则

### 1、代码先行，索引后上

一般应该等到主体业务功能开发完毕，把涉及到该表相关sql都要拿出来分析之后再建立 索引。

### 2、联合索引尽量覆盖条件

比如可以设计一个或者两三个联合索引(尽量少建单值索引)，让每一个联合索引都尽量去包含sql语句里的 where、order by、group by的字段，还要确保这些联合索引的字段顺序尽量满足sql查询的最左前缀原 则。

### 3、不要在小基数字段上建立索引

索引基数是指这个字段在表里总共有多少个不同的值，比如一张表总共100万行记录，其中有个性别字段， 其值不是男就是女，那么该字段的基数就是2。如果对这种小基数字段建立索引的话，还不如全表扫描了，因为你的索引树里就包含男和女两种值，根本没 法进行快速的二分查找，那用索引就没有太大的意义了。一般建立索引，尽量使用那些基数比较大的字段，就是值比较多的字段，那么才能发挥出B+树快速二分查 找的优势来。

### 4、长字符串我们可以采用前缀索引

尽量对字段类型较小的列设计索引，比如说什么tinyint之类的，因为字段类型较小的话，占用磁盘空间也会 比较小，此时你在搜索的时候性能也会比较好一点。 当然，这个所谓的字段类型小一点的列，也不是绝对的，很多时候你就是要针对varchar(255)这种字段建立 索引，哪怕多占用一些磁盘空间也是有必要的。 对于这种varchar(255)的大字段可能会比较占用磁盘空间，可以稍微优化下，比如针对这个字段的前20个 字符建立索引，就是说，对这个字段里的每个值的前20个字符放在索引树里，类似于alter table employees add index idx_name(name(20))。 此时你在where条件里搜索的时候，如果是根据name字段来搜索，那么此时就会先到索引树里根据name 字段的前20个字符去搜索，定位到之后前20个字符的前缀匹配的部分数据之后，再回到聚簇索引提取出来 完整的name字段值进行比对。 但是假如你要是order by name，那么此时你的name因为在索引树里仅仅包含了前20个字符，所以这个排 序是没法用上索引的， group by也是同理。所以这里大家要对前缀索引有一个了解。

### 5、where与order by冲突时优先where

在where和order by出现索引设计冲突时，到底是针对where去设计索引，还是针对order by设计索引？到 底是让where去用上索引，还是让order by用上索引? 一般这种时候往往都是让where条件去使用索引来快速筛选出来一部分指定的数据，接着再进行排序。 因为大多数情况基于索引进行where筛选往往可以最快速度筛选出你要的少部分数据，然后做排序的成本可 能会小很多。

### 6、基于慢sql查询做优化

可以根据监控后台的一些慢sql，针对这些慢sql查询做特定的索引优化。关于慢sql查询不清楚的可以参考这篇文章：https://blog.csdn.net/qq_40884473/article/details/89455740



## 9、分页查询优化

很多时候我们业务系统实现分页功能可能会用如下sql实现

 select * from employees limit 10000,10;

表示从表 employees 中取出从 10001 行开始的 10 行记录。看似只查询了 10 条记录，实际这条 SQL 是先读取 10010 条记录，然后抛弃前 10000 条记录，然后读到后面 10 条想要的数据。因此要查询一张大表比较靠后的数据，执行效率 是非常低的。

### 1、根据自增且连续的主键排序的分页查询

select * from employees where id > 90000 limit 5;

显然改写后的 SQL 走了索引，而且扫描的行数大大减少，执行效率更高。但是，这条改写的SQL 在很多场景并不实用，因为表中可能某些记录被删后，主键空缺，导致结果不一致。

所以这种改写得满 足以下两个条件：

1. 主键自增且连续
2. 结果是按照主键排序的

### 2、根据非主键字段排序的分页查询

可以让排序和分页操作通过索引覆盖先查出主键，然后根据主键查到对应的记录，

select * from employees e inner join (select id from employees order by name limit 90000,5) ed on e.id = ed.id;



## 10、Join关联查询优化

示例表

```mysql
DROP TABLE IF EXISTS `t1`;
DROP TABLE IF EXISTS `t2`;

CREATE TABLE `t1` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`a` int(11) DEFAULT NULL,
`b` int(11) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_a` (`a`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

create table t2 like t1;

-- 插入一些示例数据
-- 往t1表插入1万行记录
drop procedure if exists insert_t1;
delimiter ;;
create procedure insert_t1()
begin
declare i int;
set i=1;
while(i<=10000)do
insert into t1(a,b) values(i,i);
set i=i+1;
end while;
end;;
delimiter ;
call insert_t1();

-- 往t2表插入100行记录
drop procedure if exists insert_t2;
delimiter ;;
create procedure insert_t2()
begin
declare i int;
set i=1;
while(i<=100)do
insert into t2(a,b) values(i,i);
set i=i+1;
end while;
end;;
delimiter ;
call insert_t2();
```

### 1、 嵌套循环连接 Nested-Loop Join(NLJ) 算法

一次一行循环地从第一张表（称为驱动表）中读取行，在这行数据中取到关联字段，根据关联字段在另一张表（被驱动 表）里取出满足条件的行，然后取出两张表的结果合集。

```mysql
mysql> EXPLAIN select * from t1 inner join t2 on t1.a= t2.a;
+----+-------------+-------+------------+------+---------------+-------+---------+--------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key   | key_len | ref          | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+-------+---------+--------------+------+----------+-------------+
|  1 | SIMPLE      | t2    | NULL       | ALL  | idx_a         | NULL  | NULL    | NULL         |  100 |   100.00 | Using where |
|  1 | SIMPLE      | t1    | NULL       | ref  | idx_a         | idx_a | 5       | test_db.t2.a |    1 |   100.00 | NULL        |
+----+-------------+-------+------------+------+---------------+-------+---------+--------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

从执行计划中可以看到这些信息：

1. 驱动表是 t2，被驱动表是 t1。先执行的就是驱动表(执行计划结果的id如果一样则按从上到下顺序执行sql)；优 化器一般会优先选择小表做驱动表。所以使用 inner join 时，排在前面的表并不一定就是驱动表。
2. 当使用left join时，左表是驱动表，右表是被驱动表，当使用right join时，右表时驱动表，左表是被驱动表， 当使用inner join时，mysql会选择数据量比较小的表作为驱动表，大表作为被驱动表。
3. 使用了 NLJ算法。一般 join 语句中，如果执行计划 Extra 中未出现 Using join buffer 则表示使用的 join 算 法是 NLJ。

上面sql的大致流程如下：

1. 从表 t2 中读取一行数据（如果t2表有查询过滤条件的，会从过滤结果里取出一行数据）；
2. 从第 1 步的数据中，取出关联字段 a，到表 t1 中查找；
3. 取出表 t1 中满足条件的行，跟 t2 中获取到的结果合并，作为结果返回给客户端；
4. 重复上面 3 步。

整个过程会读取 t2 表的所有数据(扫描100行)，然后遍历这每行数据中字段 a 的值，根据 t2 表中 a 的值索引扫描 t1 表 中的对应行(扫描100次 t1 表的索引，1次扫描可以认为最终只扫描 t1 表一行完整数据，也就是总共 t1 表也扫描了100 行)。因此整个过程扫描了 200 行。如果被驱动表的关联字段没索引，使用NLJ算法性能会比较低(下面有详细解释)，mysql会选择Block Nested-Loop Join 算法。

### 2、 基于块的嵌套循环连接 Block Nested-Loop Join(BNL)算法

把驱动表的数据读入到 join_buffer 中，然后扫描被驱动表，把被驱动表每一行取出来跟 join_buffer 中的数据做对比。

```mysql
mysql> EXPLAIN select * from t1 inner join t2 on t1.b= t2.b;
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+----------------------------------------------------+
|  1 | SIMPLE      | t2    | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   100 |   100.00 | NULL                                               |
|  1 | SIMPLE      | t1    | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 10142 |    10.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.00 sec)
```

Extra 中 的Using join buffer (Block Nested Loop)说明该关联查询使用的是 BNL 算法

上面sql的大致流程如下：

1. 把 t2 的所有数据放入到 join_buffer 中
2. 把表 t1 中每一行取出来，跟 join_buffer 中的数据做对比
3. 返回满足 join 条件的数据

整个过程对表 t1 和 t2 都做了一次全表扫描，因此扫描的总行数为10000(表 t1 的数据总量) + 100(表 t2 的数据总量) = 10100。并且 join_buffer 里的数据是无序的，因此对表 t1 中的每一行，都要做 100 次判断，所以内存中的判断次数是 100 * 10000= 100 万次。这个例子里表 t2 才 100 行，要是表 t2 是一个大表，join_buffer 放不下怎么办呢？join_buffer 的大小是由参数 join_buffer_size 设定的，默认值是 256k。如果放不下表 t2 的所有数据话，策略很简单， 就是分段放。比如 t2 表有1000行记录， join_buffer 一次只能放800行数据，那么执行过程就是先往 join_buffer 里放800行记录，然 后从 t1 表里取数据跟 join_buffer 中数据对比得到部分结果，然后清空 join_buffer ，再放入 t2 表剩余200行记录，再 次从 t1 表里取数据跟 join_buffer 中数据对比。所以就多扫了一次 t1 表。

被驱动表的关联字段没索引为什么要选择使用 BNL 算法而不使用 Nested-Loop Join 呢？如果上面第二条sql使用 Nested-Loop Join，那么扫描行数为 100 * 10000 = 100万次，这个是磁盘扫描。很显然，用BNL磁盘扫描次数少很多，相比于磁盘扫描，BNL的内存计算会快得多。因此MySQL对于被驱动表的关联字段没索引的关联查询，一般都会使用 BNL 算法。如果有索引一般选择 NLJ 算法，有 索引的情况下 NLJ 算法比 BNL算法性能更高

### 3、对于关联sql的优化

1. 关联字段加索引，让mysql做join操作时尽量选择NLJ算法
2. 小表驱动大表，写多表连接sql时如果明确知道哪张表是小表可以用straight_join写法固定连接驱动方式，省去 mysql优化器自己判断的时间

straight_join解释：straight_join功能同join类似，但能让左边的表来驱动右边的表，能改表优化器对于联表查询的执 行顺序。比如：select * from t2 straight_join t1 on t2.a = t1.a; 代表指定mysql选着 t2 表作为驱动表。

1. straight_join只适用于inner join，并不适用于left join，right join。（因为left join，right join已经代表指 定了表的执行顺序）
2. 尽可能让优化器去判断，因为大部分情况下mysql优化器是比人要聪明的。使用straight_join一定要慎重，因 为部分情况下人为指定的执行顺序并不一定会比优化引擎要靠谱。

对于小表定义的明确：在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据 量，数据量小的那个表，就是“小表”，应该作为驱动表。



## 11、in和exsits优化

原则：小表驱动大表，即小的数据集驱动大的数据集

### in：当B表的数据集小于A表的数据集时，in优于exists

select * from A where id in (select id from B)
#等价于：
for(select id from B){
    select * from A where A.id = B.id
}

### exists：当A表的数据集小于B表的数据集时，exists优于in

将主查询A的数据，放到子查询B中做条件验证，根据验证结果（true或false）来决定主查询的数据是否保留

select * from A where exists (select 1 from B where B.id = A.id)
#等价于:
for(select * from A){
    select * from B where B.id = A.id
}
#A表与B表的ID字段应建立索引

1、EXISTS (subquery)只返回TRUE或FALSE,因此子查询中的SELECT * 也可以用SELECT 1替换,官方说法是实际执行时会 忽略SELECT清单,因此没有区别

2、EXISTS子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比

3、EXISTS子查询往往也可以用JOIN来代替，何种最优需要具体问题具体分析



## 12、count(*)查询优化

-- 临时关闭mysql查询缓存，为了查看sql多次执行的真实时间
mysql> set global query_cache_size=0;
mysql> set global query_cache_type=0;

mysql> EXPLAIN select count(1) from employees;
mysql> EXPLAIN select count(id) from employees;
mysql> EXPLAIN select count(name) from employees;
mysql> EXPLAIN select count(*) from employees;

注意：以上4条sql只有根据某个字段count不会统计字段为null值的数据行

四个sql的执行计划一样，说明这四个sql执行效率应该差不多

字段有索引：count(*)≈count(1)>count(字段)>count(主键 id) //字段有索引，count(字段)统计走二级索引，二 级索引存储数据比主键索引少，所以count(字段)>count(主键 id)

字段无索引：count(*)≈count(1)>count(主键 id)>count(字段) //字段没有索引count(字段)统计走不了索引， count(主键 id)还可以走主键索引，所以count(主键 id)>count(字段)

count(1)跟count(字段)执行过程类似，不过count(1)不需要取出字段统计，就用常量1做统计，count(字段)还需要取出 字段，所以理论上count(1)比count(字段)会快一点。

count(*) 是例外，mysql并不会把全部字段取出来，而是专门做了优化，不取值，按行累加，效率很高，所以不需要用 count(列名)或count(常量)来替代 count(*)。

为什么对于count(id)，mysql最终选择辅助索引而不是主键聚集索引？因为二级索引相对主键索引存储数据更少，检索 性能应该更高，mysql内部做了点优化(应该是在5.7版本才优化)。

常见优化方法

1. 查询mysql自己维护的总行数，对于myisam存储引擎的表做不带where条件的count查询性能是很高的，因为myisam存储引擎的表的总行数会被 mysql存储在磁盘上，查询不需要计算，对于innodb存储引擎的表mysql不会存储表的总记录行数(因为有MVCC机制，后面会讲)，查询count需要实时计算。
2. show table status 如果只需要知道表总行数的估计值可以用如下sql查询，性能很高。
3. 将总数维护到Redis里。插入或删除表数据行的时候同时维护redis里的表总行数key的计数值(用incr或decr命令)，但是这种方式可能不准，很难 保证表操作和redis操作的事务一致性。
4. 插入或删除表数据行的时候同时维护计数表，让他们在同一个事务里操作。



## 13、MySQL数据类型

### 1、数值类型

| 类型         | 大小  | 范围（有符号）  | 范围（无符号） | 用途           |
| ------------ | ----- | --------------- | -------------- | :------------- |
| TINYINT      | 1字节 | (-128, 127)     | (0, 255)       | 小整数值       |
| SMALLINT     | 2字节 | (-32768, 32767) | (0, 65535)     | 大整数值       |
| MEDIUMINT    | 3字节 |                 |                | 大整数值       |
| INT或INTEGER | 4字节 |                 |                | 大整数值       |
| BIGINT       | 8字节 |                 |                | 极大整数值     |
| FLOAT        | 4字节 |                 |                | 单精度浮点数值 |
| DOUBLE       | 8字节 |                 |                | 双精度浮点数值 |
| DECIMAL      |       |                 |                | 小数值         |

优化建议

1. 如果整形数据没有负数，如ID号，建议指定为UNSIGNED无符号类型，容量可以扩大一倍。
2. 建议使用TINYINT代替ENUM、BITENUM、SET。
3. 避免使用整数的显示宽度(参看文档最后)，也就是说，不要用INT(10)类似的方法指定字段显示宽度，直接用 INT。
4. DECIMAL最适合保存准确度要求高，而且用于计算的数据，比如价格。但是在使用DECIMAL类型的时候，注意 长度设置。
5. 建议使用整形类型来运算和存储实数，方法是，实数乘以相应的倍数后再操作。
6. 整数通常是最佳的数据类型，因为它速度快，并且能使用AUTO_INCREMENT。

### 2、日期和时间

| 类型      | 大小 (字节) | 范围                                     | 格式                | 用途                     |
| --------- | ----------- | ---------------------------------------- | ------------------- | ------------------------ |
| DATE      | 3           | 1000-01-01到9999-12-31                   | YYYY-MM-DD          | 日期值                   |
| TIME      | 3           | '-838:59:59'到'838:59:59'                | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1           | 1901到2155                               | YYYY                | 年份值                   |
| DATETIME  | 8           | 1000-01-01 00:00:00到9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| TIMESTAMP | 4           | 1970-01-01 00:00:00到2038-01-19 03:14:07 | YYYYMMDDhhmmss      | 混合日期和时间值，时间戳 |

优化建议

1. MySQL能存储的最小时间粒度为秒。
2. 建议用DATE数据类型来保存日期。MySQL中默认的日期格式是yyyy-mm-dd。
3. 用MySQL的内建类型DATE、TIME、DATETIME来存储时间，而不是使用字符串。
4. 当数据格式为TIMESTAMP和DATETIME时，可以用CURRENT_TIMESTAMP作为默认（MySQL5.6以后）， MySQL会自动返回记录插入的确切时间。
5. TIMESTAMP是UTC时间戳，与时区相关
6. DATETIME的存储格式是一个YYYYMMDD HH:MM:SS的整数，与时区无关，你存了什么，读出来就是什么
7. 除非有特殊需求，一般的公司建议使用TIMESTAMP，它比DATETIME更节约空间，但是像阿里这样的公司一般 会用DATETIME，因为不用考虑TIMESTAMP将来的时间上限问题。
8. 有时人们把Unix的时间戳保存为整数值，但是这通常没有任何好处，这种格式处理起来不太方便，我们并不推荐 它。

### 3、字符串

| 类型       | 大小             | 用途                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| CHAR       | 0-255字节        | 定长字符串，char(n)当插入的字符串实际长度不足n时， 插入空格进行补充保存。在进行检索时，尾部的空格会被去掉。 |
| VARCHAR    | 0-65535字节      | 变长字符串，varchar(n)中的n代表最大列长度，插入的 字符串实际长度不足n时不会补充空格 |
| TINYBLOB   | 0-255字节        | 不超过 255 个字符的二进制字符串                              |
| TINYTEXT   | 0-255字节        | 短文本字符串                                                 |
| BLOB       | 0-65535字节      | 二进制形式的长文本数据                                       |
| TEXT       | 0-65535字节      | 长文本数据                                                   |
| MEDIUMBLOB | 0-16777215字节   | 二进制形式的中等长度文本数据                                 |
| MEDIUMTEXT | 0-16777215字节   | 中等长度文本数据                                             |
| LONGBLOB   | 0-4294967295字节 | 二进制形式的极大文本数据                                     |
| LONGTEXT   | 0-4294967295字节 | 极大文本数据                                                 |

优化建议

1.  字符串的长度相差较大用VARCHAR；字符串短，且所有值都接近一个长度用CHAR。
2. CHAR和VARCHAR适用于包括人名、邮政编码、电话号码和不超过255个字符长度的任意字母数字组合。那些 要用来计算的数字不要用VARCHAR类型保存，因为可能会导致一些与计算相关的问题。换句话说，可能影响到计 算的准确性和完整性。
3. 尽量少用BLOB和TEXT，如果实在要用可以考虑将BLOB和TEXT字段单独存一张表，用id关联。
4. BLOB系列存储二进制字符串，与字符集无关。TEXT系列存储非二进制字符串，与字符集相关。
5. BLOB和TEXT都不能有默认值。

### 4、INT显示宽度

我们经常会使用命令来创建数据表，而且同时会指定一个长度，如下。但是，这里的长度并非是TINYINT类型存储的最大 长度，而是显示的最大长度。

CREATE TABLE `user`(
    `id` TINYINT(2) UNSIGNED
);

这里表示user表的id字段的类型是TINYINT，可以存储的最大数值是255。所以，在存储数据时，如果存入值小于等于 255，如200，虽然超过2位，但是没有超出TINYINT类型长度，所以可以正常保存；如果存入值大于255，如500，那么 MySQL会自动保存为TINYINT类型的最大值255。在查询数据时，不管查询结果为何值，都按实际输出。这里TINYINT(2)中2的作用就是，当需要在查询结果前填充0时， 命令中加上ZEROFILL就可以实现，如：

`id` TINYINT(2) UNSIGNED ZEROFILL

这样，查询结果如果是5，那输出就是05。如果指定TINYINT(5)，那输出就是00005，其实实际存储的值还是5，而且存 储的数据不会超过255，只是MySQL输出数据时在前面填充了0。换句话说，在MySQL命令中，字段的类型长度TINYINT(2)、INT(11)不会影响数据的插入，只会在使用ZEROFILL时有 用，让查询结果前填充0。



























[1]:data:image/png;base64,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
