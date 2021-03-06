# mysql进阶

## like
`SELECT` 语句中使用 `WHERE` 子句来获取指定的记录, `WHERE` 子句中可以使用等号 = 来设定获取数据的条件

有时候我们需要获取 某 字段含有 某个 字符的所有记录，这时我们就需要在  `WHERE` 子句中使用 `LIKE` 子句。

`LIKE` 子句中使用百分号 `%`字符来表示任意字符，如果没有使用百分号 %, LIKE 子句与等号 = 的效果是一样的。
```js
// 寻找username中已ly结尾的数据
select * from test where username like '%ly';
```
## UNION 操作符

`MySQL UNION` 操作符用于连接两个以上的 `SELECT` 语句的结果组合到一个结果集合中。多个 `SELECT` 语句会删除重复的数据。

MySQL UNION 操作符语法格式：

```js
select expression1, expression2, ... expression_n
from tables
[where conditions]
union [all | distinct]
select expression1, expression2, ... expression_n
from tables
[where conditions];
```

参数:

1. expression1, expression2, ... expression_n: 要检索的列。

2. tables: 要检索的数据表。

3. where conditions: 可选， 检索条件。

4. distinct: 可选，删除结果集中重复的数据。默认情况下 union 操作符已经删除了重复数据，所以 distinct 修饰符对结果没啥影响。

5. all: 可选，返回所有结果集，包含重复数据。

```js
select * from test where username like '%ly' union select * from test where time='2019-06-24';
```

```js
// 有重复值
select * from test where username like '%ly' union all select * from test where time='2019-06-24';
```
## 排序

`order by` 子句将查询数据排序

```js
select field1, field2,...fieldN table_name1, table_name2...
order by field1 [asc [desc][默认 asc]], [field2...] [asc [desc][默认 asc]]
```
```js
// 降序
 select * from test order by username desc;
```
```js
// 升序
 select * from test order by username;
```

## 分组

``group by` 语句根据一个或多个列对结果集进行分组

```js
select column_name, function(column_name)
from table_name
where column_name operator value
group by column_name;
```

```js
CREATE TABLE `group` (
  `id` int(11) NOT NULL,
  `name` char(10) NOT NULL DEFAULT '',
  `date` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf
```

插入数据
```js
insert into `group` values('1', '小明', '2016-04-22 15:25:33'),('2', '小王', '2016-04-20 15:25:47'),('3', '小丽', '2016-04-19 15:26:02'),('4', '小王', '2016-04-07 15:26:14'),('5', '小明', '2016-04-11 15:26:40'),('6', '小明', '2016-04-04 15:26:54');
```

```js
select * from `group`;
/*
+----+--------+---------------------+
| id | name   | date                |
+----+--------+---------------------+
|  1 | 小明   | 2016-04-22 15:25:33 |
|  2 | 小王   | 2016-04-20 15:25:47 |
|  3 | 小丽   | 2016-04-19 15:26:02 |
|  4 | 小王   | 2016-04-07 15:26:14 |
|  5 | 小明   | 2016-04-11 15:26:40 |
|  6 | 小明   | 2016-04-04 15:26:54 |
+----+--------+---------------------+
*/
```
```js
select name from `group` group by name;
/*
+--------+
| name   |
+--------+
| 小明   |
| 小王   |
| 小丽   |
+--------+
3 rows in set (0.01 sec)
*/
```

```js
// 将数据表按名字进行分组，并统计每个人有多少条记录
select name, count(*) from `group` group by name;
/**
+--------+----------+
| name   | count(*) |
+--------+----------+
| 小明   |        3 |
| 小王   |        2 |
| 小丽   |        1 |
+--------+----------+
3 rows in set (0.00 sec)
*/
```


## 连接的使用

假设：

```js
select * from `join`;
/*
+----+------------+---------------------+
| id | name       | date                |
+----+------------+---------------------+
|  1 | hbb        | 2016-04-22 15:25:33 |
|  2 | hbbaly     | 2016-04-20 15:25:47 |
|  3 | hbbaly1213 | 2016-04-19 15:26:02 |
|  4 | hbb        | 2016-04-07 15:26:14 |
+----+------------+---------------------+
4 rows in set (0.00 sec)
*/
mysql> select * from `group`;
/*
+----+--------+---------------------+
| id | name   | date                |
+----+--------+---------------------+
|  1 | 小明   | 2016-04-22 15:25:33 |
|  2 | 小王   | 2016-04-20 15:25:47 |
|  3 | 小丽   | 2016-04-19 15:26:02 |
|  4 | 小王   | 2016-04-07 15:26:14 |
|  5 | 小明   | 2016-04-11 15:26:40 |
|  6 | 小明   | 2016-04-04 15:26:54 |
+----+--------+---------------------+
6 rows in set (0.00 sec)
*/
```
### inner join
inner join: **取得是并集**
```js
// 查询 group 与 join中date一样的 id; **取得是并集**
select a.id, a.date from `group` a inner join `join` b on b.date = a.date;
/*
+----+---------------------+
| id | date                |
+----+---------------------+
|  1 | 2016-04-22 15:25:33 |
|  2 | 2016-04-20 15:25:47 |
|  3 | 2016-04-19 15:26:02 |
|  4 | 2016-04-07 15:26:14 |
+----+---------------------+
4 rows in set (0.00 sec)
*/

```
以上语句等同于：

```js
select a.id, a.date from `group`a, `join` b where b.date = a.date;
/*
+----+---------------------+
| id | date                |
+----+---------------------+
|  1 | 2016-04-22 15:25:33 |
|  2 | 2016-04-20 15:25:47 |
|  3 | 2016-04-19 15:26:02 |
|  4 | 2016-04-07 15:26:14 |
+----+---------------------+
4 rows in set (0.00 sec)
*/
```

### left join

```js
select a.id, a.date from `group` a left join `join` b on b.date = a.date;
/*
+----+---------------------+
| id | date                |
+----+---------------------+
|  1 | 2016-04-22 15:25:33 |
|  2 | 2016-04-20 15:25:47 |
|  3 | 2016-04-19 15:26:02 |
|  4 | 2016-04-07 15:26:14 |
|  5 | 2016-04-11 15:26:40 |
|  6 | 2016-04-04 15:26:54 |
+----+---------------------+
6 rows in set (0.00 sec)
*/
```
只要是join有符合的，就会返回`group`的全部数据

### right join

```js
select a.id, a.date from `group` a right join `join` b on b.date = a.date;
/*
+------+---------------------+
| id   | date                |
+------+---------------------+
|    1 | 2016-04-22 15:25:33 |
|    2 | 2016-04-20 15:25:47 |
|    3 | 2016-04-19 15:26:02 |
|    4 | 2016-04-07 15:26:14 |
+------+---------------------+
4 rows in set (0.00 sec)
*/
```

只要是`group`有符合的，就会返回`join`的全部数据

## NULL 值处理

为了处理这种情况，MySQL提供了三大运算符:

- `IS NULL`: 当列的值是 NULL,此运算符返回 true。
- `IS NOT NULL`: 当列的值不为 NULL, 运算符返回 true。
- `<=>`: 比较操作符（不同于=运算符），当比较的的两个值为 NULL 时返回 true。

```js
// 创建相应表
create table `null`(
  name varchar(40) not null,
  age int
);
```
掠过添加数据步骤

```js
 select * from `null`;
 /*
+------------+------+
| name       | age  |
+------------+------+
| hbb        |   25 |
| hbb        | NULL |
| hbbaly     | NULL |
| hbbaly1214 | NULL |
| hbbaly1214 |   26 |
+------------+------+
5 rows in set (0.00 sec)
*/
```

```js
 select * from `null` where age = null;
// Empty set (0.00 sec)

select * from `null` where age != null;
// Empty set (0.00 sec)

```

可以看出： **= 和 != 运算符是不起作用的**

```js
select * from `null` where age is null;
/*
+------------+------+
| name       | age  |
+------------+------+
| hbb        | NULL |
| hbbaly     | NULL |
| hbbaly1214 | NULL |
+------------+------+
3 rows in set (0.00 sec)
*/
```

```js
select * from `null` where age is not null;
/*
+------------+------+
| name       | age  |
+------------+------+
| hbb        |   25 |
| hbbaly1214 |   26 |
+------------+------+
2 rows in set (0.00 sec)
*/
```

## 正则表达式

以下正则模式可应用于 REGEXP 操作符中：

![reg](../.vuepress/public/img/mysql-4.png 'reg')

```js
// 已ly结尾的name
select * from `null` where name regexp 'ly$';
/*
+--------+------+
| name   | age  |
+--------+------+
| hbbaly | NULL |
+--------+------+
1 row in set (0.00 sec)
*/
```
```js
 select * from `null` where name regexp '[13456]$|ly$';
 /*
+------------+------+
| name       | age  |
+------------+------+
| hbbaly     | NULL |
| hbbaly1214 | NULL |
| hbbaly1214 |   26 |
+------------+------+
3 rows in set (0.00 sec)
*/
```

## 事务

`MySQL` 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！

在 `MySQL` 中只有使用了 `Innodb` 数据库引擎的数据库或表才支持事务。
事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。

事务用来管理 `insert,update,delete` 语句

一般来说，事务是必须满足4个条件（ACID）
- 原子性（Atomicity，或称不可分割性）、
- 一致性（Consistency）
- 隔离性（Isolation，又称独立性）
- 持久性（Durability）


原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

在 `MySQL` 命令行的默认设置下，事务都是自动提交的，即执行 `SQL` 语句后就会马上执行 `COMMIT` 操作。因此要显式地开启一个事务务须使用命令 `BEGIN` 或 `START TRANSACTION`，或者执行命令 `SET AUTOCOMMIT=0`，用来禁止使用当前会话的自动提交。

事务控制语句：
`BEGIN` 或 `START TRANSACTION` 显式地开启一个事务；

`COMMIT` 也可以使用 `COMMIT WORK`，不过二者是等价的。COMMIT 会提交事务，并使已对数据库进行的所有修改成为永久性的；

`ROLLBACK` 也可以使用 `ROLLBACK WORK`，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；

`SAVEPOINT identifier`，`SAVEPOINT` 允许在事务中创建一个保存点，一个事务中可以有多个 `SAVEPOINT`；

`RELEASE SAVEPOINT identifier` 删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；

`ROLLBACK TO identifier` 把事务回滚到标记点；

`SET TRANSACTION` 用来设置事务的隔离级别。`InnoDB` 存储引擎提供事务的隔离级别有`READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE`。

MYSQL 事务处理主要有两种方法：
1、用 BEGIN, ROLLBACK, COMMIT来实现
- BEGIN 开始一个事务
- ROLLBACK 事务回滚
- COMMIT 事务确认
2、直接用 SET 来改变 MySQL 的自动提交模式:

- SET AUTOCOMMIT=0 禁止自动提交
- SET AUTOCOMMIT=1 开启自动提交


```js
create table test(id int); // 创建test表
// Query OK, 0 rows affected (0.01 sec)

mysql> select * from test;
// Empty set (0.00 sec)

mysql> begin; // 开始事务
// Query OK, 0 rows affected (0.00 sec)

mysql> insert into test values(4);
// Query OK, 1 row affected (0.00 sec)

mysql> select * from test;
/*
+------+
| id   |
+------+
|    4 |
+------+
1 row in set (0.00 sec)
*/
mysql> commit;  // 提交事务
// Query OK, 0 rows affected (0.00 sec)

mysql> select * from test;
/*
+------+
| id   |
+------+
|    4 |
+------+
1 row in set (0.00 sec)
*/
mysql> begin; // 开始事务
// Query OK, 0 rows affected (0.00 sec)

mysql> insert into test values(5);
// Query OK, 1 row affected (0.00 sec)

mysql> select * from test;
/*
+------+
| id   |
+------+
|    4 |
|    5 |
+------+
2 rows in set (0.00 sec)
*/
mysql> rollback; // 回滚，刚才的insert没有效果
// Query OK, 0 rows affected (0.00 sec)

mysql> select * from test;
/*
+------+
| id   |
+------+
|    4 |
+------+
1 row in set (0.00 sec)
*/

```
## 临时表

**TEMPORARY**

`MySQL` 临时表在我们需要保存一些临时数据时是非常有用的。临时表只在当前连接可见，当关闭连接时，Mysql会自动删除表并释放所有空间。

临时表在`MySQL 3.23`版本中添加，如果你的`MySQL`版本低于 `3.23`版本就无法使用`MySQL`的临时表。不过现在一般很少有再使用这么低版本的MySQL数据库服务了。

```js
create temporary table tem(id int);
// Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
/*
+----------------+
| Tables_in_test |
+----------------+
| group          |
| join           |
| null           |
| test           |
+----------------+
4 rows in set (0.00 sec)
*/
```
当你使用 show tables命令显示数据表列表时，你将无法看到 tem表

默认情况下，当你断开与数据库的连接后，临时表就会自动被销毁。当然你也可以在当前`MySQL`会话使用 `drop table`命令来手动删除临时表。

```js
drop table tem;
// Query OK, 0 rows affected (0.00 sec)

mysql> select * from tem;
// ERROR 1146 (42S02): Table 'test.tem' doesn't exist
```

## 复制表

如果我们需要完全的复制`MySQL`的数据表，包括表的结构，索引，默认值等。 如果仅仅使用`CREATE TABLE ... SELECT` 命令，是无法实现的。

本章节将为大家介绍如何完整的复制`MySQL`数据表，步骤如下：

1. 使用 `SHOW CREATE TABLE` 命令获取创建数据表(`CREATE TABLE`) 语句，该语句包含了原数据表的结构，索引等。
2. 复制以下命令显示的SQL语句，修改数据表名，并执行SQL语句，通过以上命令 将完全的复制数据表结构。
3. 如果你想复制表的内容，你就可以使用 `INSERT INTO ... SELECT` 语句来实现。

```js
create table copy(id int);
// Query OK, 0 rows affected (0.02 sec)

mysql> select * from copy;
// Empty set (0.00 sec)

mysql> select * from test;
/*
+------+
| id   |
+------+
|    4 |
+------+
1 row in set (0.00 sec)
*/
mysql> insert into copy(id) select id from test; // 也可以这样 insert into copy select * from test 拷贝全部的数据
// Query OK, 1 row affected (0.01 sec)
// Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from copy;
/*
+------+
| id   |
+------+
|    4 |
+------+
1 row in set (0.00 sec)
*/
```

## 序列使用

### 使用 AUTO_INCREMENT
  忽略不写
### 重置序列

如果你删除了数据表中的多条记录，并希望对剩下数据的AUTO_INCREMENT列进行重新排列，那么你可以通过删除自增的列，然后重新添加来实现。 不过该操作要非常小心，如果在删除的同时又有新记录添加，有可能会出现数据混乱。操作如下所示：

```js
mysql> ALTER TABLE insect DROP id; // 删除id
mysql> ALTER TABLE insect
    -> ADD id INT UNSIGNED NOT NULL AUTO_INCREMENT FIRST, // 重新添加第一条id
    -> ADD PRIMARY KEY (id);
```

### 设置序列的开始值
一般情况下序列的开始值为1，但如果你需要指定一个开始值100，那我们可以通过以下语句来实现：
```js
mysql> CREATE TABLE insect
    -> (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY (id),
    -> name VARCHAR(30) NOT NULL, 
    -> date DATE NOT NULL,
    -> origin VARCHAR(30) NOT NULL
)engine=innodb auto_increment=100 charset=utf8;
```
或者你也可以在表创建成功后，通过以下语句来实现：
```js
mysql> ALTER TABLE insect AUTO_INCREMENT = 100;
```
## 处理重复数据

`MySQL` 数据表中设置指定的字段为 `PRIMARY KEY`（主键） 或者 `UNIQUE`（唯一） 索引来保证数据的唯一性。

`INSERT IGNORE INTO` 与 `INSERT INTO` 的区别就是 `INSERT IGNORE `会忽略数据库中已经存在的数据，如果数据库没有数据，就插入新的数据，如果有数据的话就跳过这条数据。这样就可以保留数据库中已经存在数据，达到在间隙中插入数据的目的。
```js
 CREATE TABLE UNIQYE(
    last char(20) not null,
    first char(20) not null,
    sex char(10),
    PRIMARY KEY(first, last)
  );
// Query OK, 0 rows affected (0.08 sec)
//你想设置表中字段 first，last 数据不能重复，你可以设置双主键模式来设置数据的唯一性， 如果你设置了双主键，那么那个键的默认值不能为 NULL，可设置为 NOT NULL。
mysql> insert into UNIQYE(first,last) values('hbb','hbbaly');
// Query OK, 1 row affected (0.01 sec)

//设置了唯一索引，那么在插入重复数据时，SQL 语句将无法执行成功,并抛出错。
mysql> insert into UNIQYE(first,last) values('hbb','hbbaly');
// ERROR 1062 (23000): Duplicate entry 'hbb-hbbaly' for key 'PRIMARY'

// 执行后不会出错，也不会向数据表中插入重复数据：
mysql> insert ignore into UNIQYE(first,last) values('hbb','hbbaly');
// Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql>select * from UNIQYE;
// +--------+-------+------+
// | last   | first | sex  |
// +--------+-------+------+
// | hbbaly | hbb   | NULL |
// +--------+-------+------+
// 1 row in set (0.00 sec)
```

## 统计重复数据

```js
select * from UNIQYE;
/*
+---------------+-------+------+
| last          | first | sex  |
+---------------+-------+------+
| hbbaly        | hbb   | NULL |
| hbbaly1314    | hbb   | NULL |
| hbbaly1314520 | hbb   | NULL |
| hbbaly1314    | hbbly | NULL |
| hbbaly1314520 | hbbly | NULL |
+---------------+-------+------+
5 rows in set (0.00 sec)
*/

mysql> select count(*) as num, first, last from UNIQYE group by first, last;
/*
+-----+-------+---------------+
| num | first | last          |
+-----+-------+---------------+
|   1 | hbb   | hbbaly        |
|   1 | hbb   | hbbaly1314    |
|   1 | hbb   | hbbaly1314520 |
|   1 | hbbly | hbbaly1314    |
|   1 | hbbly | hbbaly1314520 |
+-----+-------+---------------+
5 rows in set (0.00 sec)
*/
mysql> select count(*) as num, first from UNIQYE group by first;
/*
+-----+-------+
| num | first |
+-----+-------+
|   3 | hbb   |
|   2 | hbbly |
+-----+-------+
2 rows in set (0.01 sec)
*/
mysql> select count(*) as num, last from UNIQYE group by last;
/*
+-----+---------------+
| num | last          |
+-----+---------------+
|   1 | hbbaly        |
|   2 | hbbaly1314    |
|   2 | hbbaly1314520 |
+-----+---------------+
3 rows in set (0.00 sec)
*/
```

## 过滤重复数据

如果你需要读取不重复的数据可以在 SELECT 语句中使用 DISTINCT 关键字来过滤重复数据。
```js
select distinct first from UNIQYE;
/*
+-------+
| first |
+-------+
| hbb   |
| hbbly |
+-------+
2 rows in set (0.00 sec)
*/

// group by 也可以过滤
mysql> select count(*) as num, first from UNIQYE group by first;
/*
+-----+-------+
| num | first |
+-----+-------+
|   3 | hbb   |
|   2 | hbbly |
+-----+-------+
2 rows in set (0.00 sec)
*/
```

## 删除重复数据

如果你想删除数据表中的重复数据，你可以使用以下的SQL语句：
```js
mysql> CREATE TABLE tmp SELECT last_name, first_name, sex FROM person_tbl  GROUP BY (last_name, first_name, sex);
mysql> DROP TABLE person_tbl;
mysql> ALTER TABLE tmp RENAME TO person_tbl;
```

## SQL 注入

如果您通过网页获取用户输入的数据并将其插入一个MySQL数据库，那么就有可能发生SQL注入安全的问题。

防止SQL注入，我们需要注意以下几个要点：

1. 永远不要信任用户的输入。对用户的输入进行校验，可以通过正则表达式，或限制长度；对单引号和 双"-"进行转换等。
2. 永远不要使用动态拼装sql，可以使用参数化的sql或者直接使用存储过程进行数据查询存取。
3. 永远不要使用管理员权限的数据库连接，为每个应用使用单独的权限有限的数据库连接。
4. 不要把机密信息直接存放，加密或者hash掉密码和敏感的信息。
5. 应用的异常信息应该给出尽可能少的提示，最好使用自定义的错误信息对原始错误信息进行包装
6. sql注入的检测方法一般采取辅助软件或网站平台来检测，软件一般采用sql注入检测工具jsky，网站平台就有亿思网站安全平台检测工具。MDCSOFT SCAN等。采用MDCSOFT-IPS可以有效的防御SQL注入，XSS攻击等。


