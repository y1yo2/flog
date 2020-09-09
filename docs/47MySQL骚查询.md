知识总结：

mysql 的执行顺序：

```mysql
select *, sum(A.cost), (A.m + B.m) as 'math' from A left join B on A.id = B.aid 
 where A.type = 1
 group by A.id
 having A.id > 50
 order by A.id
```

1. **from** 子句组装来自不同数据源的数据；（先 **join** 再 **on**）
2. **where** 基于指定的条件对记录进行筛选
3. **group by** 将数据划分为多个分组
4. **聚集函数**进行计算
5. **having** 子句筛选分组
6. 计算 **表达式**
7. **select** 字段
8. **order by** 对结果集进行排序



具体例子分析：

1. FROM <left_table> : A
2. <join_type> JOIN <right_table> : left join B
3. ON <join_condition> : A.id=B.aid
4. WHERE <where_condition> : A.type=1
5. GROUP BY <group_by_list> : A.id
6. WITH {CUBE | ROLLUP} : sum(A.cost)
7. HAVING <having_condition> : A.id>50
8. 
9. SELECT 
10. DISTINCT
11. ORDER BY <order_by_list>



以上每个步骤都会产生一个虚拟表，该虚拟表被用作下一个步骤的输入。这些虚拟表对调用者(客户端应用程序或者外部查询)不可用。只有最后一步生成的表才会会给调用者。如果没有在查询中指定某一个子句，将跳过相应的步骤。



窗口函数：

https://note.youdao.com/ynoteshare1/index.html?id=c823f9d1fc4491c3850963f37ae94f3b&type=note

count() over(partition by ... order by ...)

max() over(partition by ... order by ...)

min() over(partition by ... order by ...)

sum() over(partition by ... order by ...)

avg() over(partition by ... order by ...)

PARTITION 中文是分割的意思，ORDER 是排序的意思，所以翻译一下就是先把一组数据按照制定的字段进行分割成各种组，然后组内按照某个字段排序。



题目与答案

参考资料：

SQL 进阶技巧 - 蛙课网的文章 - 知乎 https://zhuanlan.zhihu.com/p/139856374

https://zhuanlan.zhihu.com/p/138798721



1.一条 sql 同时count/sum 不同类型的值：例如，费用表同时算 总费用、类型1费用、类型2费用。

```mysql
select pid, count(cost), 
count(case when type=1 then cost else 0 end) as '费用1', 
count(case when type=2 then cost else 0 end) as '费用2'
form t_cost group by pid;
```

> count/sum + case when



2.一条 sql 同时显示 count/sum 值和其中的明细：例如，成绩表同时看 总成绩 和每科成绩。

```mysql
select username, count(mark) over(partition by username) 
 from t_mark
 group by username;
```

> count/sum/avg/max/min + over(partition by, order by)



3.对当前工资为 1 万以上的员工，加薪 10%。对当前工资低于 1 万的员工，加薪 20%。

```mysql
update t_salaries 
 set salary= 
 CASE WHEN salary>=10000 THEN salary*1.1
 WHEN salary<100000 THEN salary*1.2
 ELSE END;
```

> update set + case when



4.如果列的值为 null，则展示其他

```mysql
SELECT 
    COALESCE(city, 'N/A')
  FROM
    customers;
```

> coalesce 



5.

表A：

city，userid，ARP

表B：

userid，cost

  1.写出表一中各地市客户数、总费用（ARPU之和） 的SQL语句

```mysql
select count(userid), sum(ARP) from A group by city;
```

  2.请写出表一中各地市ARPU(0,30),[30,50),[50-80),[80以上)客户数分别是多少的SQL语句

```mysql
select city, 
sum(case when ARP>0 and ARP <30 then 1 else 0 end), 
sum(case when ARP>=30 and ARP <50 then 1 else 0 end), 
sum(case when ARP>=50 and ARP <80 then 1 else 0 end), 
sum(case when ARP>=80 then 1 else 0 end), 
from A group by city;
```

  3.表二中用户有重复的记录，请写出提取2条及以上用户的SQL语句

```mysql
select userid
from A 
group by userid 
having count(userid)>=2;
```



6.







