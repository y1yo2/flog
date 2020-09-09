# MySQL 的 JOIN

参考文章：

- https://www.cnblogs.com/fudashi/p/7491039.html
- https://www.cnblogs.com/fudashi/p/7506877.html#
- https://www.cnblogs.com/fudashi/p/7508272.html
- https://www.cnblogs.com/fudashi/p/7521915.html
- https://www.cnblogs.com/fudashi/p/7541991.html



## 基本用法

MySQl 连接两张表的方式有：笛卡尔积、自然连接、内连接、左连接、右连接、外连接



假设 A、B表各有两条数据：

| A表.id |
| ------ |
| 1      |
| 2      |

| B表.id |
| ------ |
| 2      |
| 3      |



### 笛卡尔积（全连接）

将 A表和 B表的每一条记录强行拼在一起。

例如：A表有 n条记录，B表有 m条记录，笛卡尔积 为 n*m条记录

```mysql
-- 笛卡尔积：CROSS JOIN
SELECT * FROM table1 CROSS JOIN table2;

SELECT * FROM table1, table2;

SELECT * FROM table1 INNER JOIN table2;
```

| A.id | B.id |
| ---- | ---- |
| 1    | 2    |
| 1    | 3    |
| 2    | 2    |
| 2    | 3    |



### 自然连接

将两个表中相同名称的列进行匹对，进行 `INNER JOIN`

特点：

- 关联的表具有一对或多对同名的列
- 连接时不需要使用 `ON` 或 `USING`

```mysql
-- 自然连接：NATURAL JOIN
SELECT * FROM table1 NATURAL JOIN table2;
```

| id   |
| ---- |
| 2    |

总结：`NATURAL JOIN` 根据列的名称进行关联，如果两个表多个列的名称相同，则这些列都会作为 `INNER JOIN` 的关联条件。



### 内连接

求两个表的交集。具体来讲，从笛卡尔积中选出 `ON` 条件成立的记录。

写法如下：

```mysql
-- 内连接：INNER JOIN
SELECT * FROM table1 INNER JOIN table2 ON table1.id = table2.id;

SELECT * FROM table1, table2 WHERE table1.id = table2.id;

SELECT * FROM table1 STRAIGHT_JOIN table2 ON table1.id = table2.id;

SELECT * FROM table1 JOIN table2 ON table1.id = table2.id;
```

| A.id | B.id |
| ---- | ---- |
| 2    | 2    |



### 左连接

以左表为准，按条件关联右表的记录。

```mysql
-- 左连接：LEFT JOIN
SELECT * FROM table1 LEFT JOIN table2 ON table1.id = table2.id;
```

| A.id | B.id |
| ---- | ---- |
| 1    | null |
| 2    | 2    |



### 右连接

以右表为准，按条件关联左表的记录。

```mysql
-- 右连接：RIGHT JOIN
SELECT * FROM table1 RIGHT JOIN table2 ON table1.id = table2.id;
```

| A.id | B.id |
| ---- | ---- |
| 2    | 2    |
| null | 3    |



### 外连接

左连接与右连接的并集。

```mysql
-- 外连接：OUTER JOIN
SELECT * FROM table1 LEFT JOIN table2 ON table1.id = table2.id
UNION
SELECT * FROM table1 RIGHT JOIN table2 ON table1.id = table2.id;
```

| A.id | B.id |
| ---- | ---- |
| 1    | null |
| 2    | 2    |
| null | 3    |

总结：当表的字段不一样时，只能使用左连接与右连接的并集，进行外连接查询。

​			当关联的条件字段一样时，可以先使用 union，查询条件字段的并集，再用并集左连接其他表。

​			前提是所有表的关联字段相同，先查关联字段的并集。



### USING 代替 ON

当 `ON` 子句采用相同列名，可以使用 `USING` 简化。

```mysql
-- USING 代替 ON
SELECT * FROM table1 INNER JOIN table2 USING(id);
```

| id   |
| ---- |
| 2    |

 注意：SELECT *时，USING会去除USING指定的列，而ON不会。



## JOIN 的原理

### 连接算法

#### Nested Loop Join（NLJ）算法：

NLJ 基础算法，嵌套循环算法。

循环外层是驱动表，循环内层是被驱动表。驱动表会驱动被驱动表进行连接操作。

```java
foreach row1 from table1
    foreach row2 from table2
    	if row2 match row1 // row2与row1满足连接条件
            join row1 and row2 into result // 连接row1和row2，加入结果集
```

首先加载 table1，取出第一条记录，再加载 table2，与 table2 的记录逐个匹配，连接匹配的记录并加入结果集。



#### Block Nested Loop Join（BNLJ）算法：

BNLJ，块嵌套循环算法。

对 NLJ 算法的优化：建立一个缓存区，一次性从驱动表取多条记录，然后扫描被驱动表。被驱动表的每条记录都尝试与缓冲区的记录匹配。如果匹配则连接记录并加入结果集。

缓冲区越大，驱动表一次取出的记录越多。减少被驱动表的加载次数从而提高效率。



### 性能优化

#### 1. 内循环次数 

当 table1 有1000条记录，table2 有100条记录。比较 t1驱动t2 和 t2驱动t1 的效率。

虽然循环执行次数都是 1000*100，但是 t1驱动t2，要加载 table2 1000次；t2驱动t1，要加载 table1 100次。

所以 t2驱动t1 效率更好。所以**小表驱动大表能减少表的加载次数，提高连接效率**



设置缓冲区同理，合理的缓冲区能减少内循环次数（被驱动表的加载次数）。
所以**设置合理的缓冲区大小能减少表的加载次数，提高连接效率**



#### 2. 快速查询

加载被驱动表后，需要扫描被驱动表的每一条记录。如何加快查询表的每一条记录？可以建立索引。

**被驱动表建立索引能提高效率**



#### 3. 排序

t2驱动t1 ，连接条件是 t2.id = t1.id，先对 id 排序。方式一：`ORDER BY t2.id` ，方式二： `ORDER BY t1.id` 。

方式一时，先对 table2 排序，再进行表连接。
方式二时，先进行 t2驱动t1 的表连接，再对结果集排序，效率更低。

所以**优先选择驱动表的属性排序可以提高效率**



## JOIN 的优化实践

#### 选择驱动表

小表驱动大表，减少内循环次数。

左连接、右连接可以根据业务需求认定谁是驱动表，谁是被驱动表。但是内连接不同。

MySQL 自带的 Optimizer 会优化内连接。自动选择小表驱动大表。



#### 索引的建立

建立索引加快被驱动表的查询速度。

左连接中，左表是驱动表，右表是被驱动表，所以在右表建立索引。

右连接中，右表是驱动表，左表是被驱动表，所以在左表建立索引。

内连接时，由于 MySQL Optimizer 优化，自动选择小表驱动大表，所以在大表建立索引。

注意，在小表建立索引，MySQL Optimizer 会认为用大表驱动小表效率更快，转用大表驱动小表。



#### 排序字段的选择

分为**对连接属性排序**和**对非连接属性排序**两种情况。

##### 对连接属性排序

例如 t1内连接t2，t1.id=t2.id ，要求对 id 属性排序。

找出驱动表和被驱动表，因为 t1 是大表，所以 t2驱动t1。所以对驱动表 t2 的 id 进行排序。



##### 对非连接属性排序

例如 t1内连接t2，t1.id=t2.id ，要求对 t1的 meg 属性排序。

因为 t1 是大表，所以 t2驱动t1。如果 对t1的 meg属性排序，必然导致对连接后结果集进行排序 **Using temporary**（比Using filesort更严重）。

| TODO备注：Using temporary 和 Using filesort



###### 能不能不用MySQL Optimizer，用大表驱动小表呢？

这就是 `STRAIGHT_JOIN` 的作用。

```mysql
SELECT * FROM t1 STRAIGHT_JOIN t2 ON t1.id = t2.id ORDER BY t1.meg;
```

使用 `STRAIGHT_JOIN` 强制指定内连接时的驱动表。



















