MySQL引擎与建表规范



引擎

InnoDB：事务，行锁，外键，维护索引效率差

MyISAM：不支持事务，表锁，访问快

Memory：数据在内存，丢失，hash索引寻找快（不适合范围）



三范式

1. 列不可分

   例子：信息列存电话与QQ

2. 非关键列都依赖关键列

   例子：学生id，课程id，成绩，姓名。姓名依赖学生，成绩依赖学习、课程；产生冗余。

3. 非关键列不能决定 其他非关键列

   例子：学生id，姓名，课程id，课程名称。课程id决定课程名称；产生冗余。







参考资料

https://juejin.im/post/6844903874684272648#heading-5