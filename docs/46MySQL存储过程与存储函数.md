存储过程与存储函数

> 什么是存储过程？

它是一组用于完成某些功能的 SQL语句集。

用户通过 自定义命名和入参出参，定义存储过程。

内部逻辑可使用多个 SQL和 自定义的变量完成复杂的业务逻辑。



> 什么是存储函数？

存储函数和存储过程一样，都是sql和语句组成的代码块。



> 两者有什么区别呢？

存储过程用 call调用，存储函数可以直接调用。

存储函数必须有一条 return语句，返回值。存储过程可以 没有或者有多个返回值。

创建语句的区别：PROCEDURE 和 FUNCTION



> 为什么要使用存储过程？有什么优缺点。

1. 可以通过变量 和 语句 完成 多sql的复杂功能。
2. 存储在数据库，创建时编译，后续使用不需要编译。
3. 与先读取数据到程序，再进行逻辑处理比较，降低了 网络等IO的消耗。
4. 可直接修改存储过程的逻辑，不用重启服务器，便于维护。

缺点：

1. 不同数据库语法不一致，移植性差
2. 调试不方便
3. 每个存储过程都要维护复杂的业务处理，数量多后维护成本高。





> 总结

一般来讲，存储过程和存储函数的区别在于存储函数可以有一个返回值，而存储过程没有返回值 。    

存储过程和存储函数都可以通过out指定一个或者多个输出参数，我们可以利用out参数，在存储过程和存储函数中实现返回多个值。

原则：

**如果只有一个返回值，用存储函数，否则用存储过程。**



存储过程：

```mysql
-- 临时定义结束符为"//"
DELIMITER //
create PROCEDURE 过程名( in|out|inout 参数名 数据类型 , ...)
begin
	 # 声明一个默认值为unknown的val_name局部变量
    declare val_name varchar(32) default 'unknown';
	# 为局部变量赋值
    set val_name = 'Centi';
	sql语句;
end//
-- 将结束符重新定义回结束符为";"
DELIMITER ;
call 过程名(参数值);

# 删除已存在存储过程——se()
drop procedure if exists se;
```



存储函数：

```mysql
delimiter ?
CREATE FUNCTION fun_name (par_name type[,...])
RETURNS type
DETERMINISTIC
begin
   declare name VARCHAR(50);
   set name=(select name from animal where id=animalId);
   return (name);
end?
delimiter;

-- 调用
select fun_name(1)
```



> 存储总结

**存储过程：** 存储过程是最常见的存储程序，存储过程是能够接受输入和输出参数并且能够在请求时被执行的程序单元。

**存储函数：** 存储函数和存储过程很相像，但是它的执行结果会返回一个值。最重要的是存储函数可以被用来充当标准的 SQL 语句，允许程序员有效的扩展 SQL 语言的能力。

**触发器：** 触发器是用来响应激活或者触发数据库行为事件的存储程序。通常，触发器用来作为数据库操作语言的响应而被调用，触发器可以被用来作为数据校验和自动反向格式化。







参考资料：

https://juejin.im/post/6844904185725468685

https://juejin.im/post/6844903848394702856

