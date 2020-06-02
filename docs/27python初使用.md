### 学习网址：https://www.jianshu.com/u/e2556947ceb7



### 题目一：

列出1到20的数字，若是3的倍数就用different代替，若是5的倍数就用orange代替，

若既是3的倍数又是5的倍数就用differentorange代替。



使用知识：

#### 切片

```python
切片操作基本表达式：str[start_index:end_index:step]
```

step：正负数均可，其绝对值大小决定了切取数据时的‘‘步长”，正负号决定了“切取方向”，正表示“从左往右”取值，负表示“从右往左”取值。当step省略时，默认为1，即从左往右以增量1取值。

start_index：表示**起始索引（包含该索引本身）**；该参数省略时，表示从对象“端点”开始取值，至于是从“起点”还是从“终点”开始，则由step参数的正负决定，step为正从“起点”开始，为负从“终点”开始。

end_index：表示**终止索引（不包含该索引本身）**；该参数省略时，表示一直取到数据“端点”，至于是到“起点”还是到“终点”，同样由step参数的正负决定，step为正时直到“终点”，为负时直到“起点”。



#### **Python中的and和or**:

字符串与 and、or的关系

```python
print('' and 2) // 无输出
print(1 and 2)  // 输出2
print('' or 2)  // 输出2
print('test' or 2) // 输出test
```





### 题目二：

有一串字符串人名:

names=' Kunpen Ji, Li XIAO, Caron Li, Donl SHI, Ji ZHAO, Fia YUAN Y, Weue DING, Xiu XU, Haiying WANG, Hai LIN, Jey JIANG, Joson WANG E, Aiyang ZHANG, Hay MENG, Jak ZHANG E, Chang Zhang, Coro ZHANG',

我希望能做到下面3点

问题1：排序,按照名字A-Z排序

问题2：找出里面姓”ZHANG”有几个

问题3：找出名字里面最长的人



问题1：排序,按照名字A-Z排序

使用知识：

#### str.split()

str.split(); 默认使用 `,` 作为分割符



#### sorted()

**sorted()**: https://www.runoob.com/python/python-func-sorted.html 

**sort 与 sorted 区别：**

> **sort 与 sorted 区别：**
>
> sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。
>
> list 的 sort 方法返回的是对已经存在的列表进行操作，无返回值，而内建函数 sorted 方法返回的是一个新的 list，而不是在原来的基础上进行的操作。



问题2：找出里面姓”ZHANG”有几个

使用知识：字符串操作之 join() ；

```python
str = ''.join(list_name)#使用join()方法，把字符串列表转换为字符串，无分隔符

str = '-'.join(list_name)#使用join()方法，把字符串列表转换为字符串，使用'-'作为分隔符
```



问题3：找出名字里面最长的人

使用知识： 

#### max() 方法

max() 方法返回给定参数的最大值，参数可以为序列。 

```python
a=[(1,2),(2,3),(3,4)]                #假设列表里面是元组构成元素呢
max(a)                               #按照元素里面元组的第一个元素的排列顺序，输出最大值（如果第一个元素相同，则比较第二个元素，输出最大值）据推理是按ascii码进行排序的
(3, 4)

a=[('a',1),('A',1)]                  #实验推测是按ascii码进行排序，比较  a  和   A 的值，得出a > A   ,  因为ascii 码里面，按照排列顺序 小 a在 A的后面
max(a)
('a', 1)

a={1:2,2:2,3:1,4:'aa'}               #比较字典里面的最大值，会输出最大的键值
max(a)
```



#### python的各种推导式

python的各种推导式（列表推导式、字典推导式、集合推导式）  https://www.jianshu.com/p/25f9c230caca 

##### 一、列表推导式，使用[]生成list

基本格式

[表达式 for 变量 in 列表]   或者  [表达式 for 变量 in 列表 if 条件]

```python
names = ['Bob','Tom','alice','Jerry','Wendy','Smith']    
[name.upper() for name in names if len(name)>3]  # ['ALICE', 'JERRY', 'WENDY', 'SMITH'] 
```



列表推导式总共有两种形式：

① `[x for x in data if condition]`

  此处if主要起条件判断作用，data数据中只有满足if条件的才会被留下，最后统一生成为一个数据列表



② `[exp1 if condition else exp2 for x in data]`

  此处if...else主要起赋值作用，当data中的数据满足if条件时将其做exp1处理，否则按照exp2处理，最后统一生成为一个数据列表



#####  二、字典推导式 

```python
{ key_expr: value_expr for value in collection if condition }
```



##### 三、集合推导式

```python
{ expr for value in collection if condition }
```



### 题目三

 **九宫格：**  1至9九个数字，横竖都有3个格，思考怎么使每行、每列两个对角线上的三数之和都等于15 



使用知识： 

#### 判断对象是否可迭代

```python
if isinstance(obj, Iterable):
```

 collections是Python内建的一个集合模块，提供了许多有用的集合类。 

 https://www.liaoxuefeng.com/wiki/1016959663602400/1017681679479008 

通过 `from collections import Iterable` 引入 `Iterable` 类型

​	    `from collections.abc import Iterable `

 3.6前的版本不需要带.abc，3.7需要加.abc，3.8后不允许使用不加abc的情况。 



`Iterator` 和 `Iterable` 的区别： https://www.jianshu.com/p/8c0e97371462 

凡是可以for循环的，都是Iterable（可迭代，确定数据多少）

凡是可以next()的，都是Iterator（迭代器，不确定数据多少）



#### itertools.permutations(iterable, num)

 itertools 是python的迭代器模块 : https://www.cnblogs.com/haiyan123/p/9804091.html 

permutations(iterable, num):

循环 iterable 的所有元素，组合为 num 长度的 Iterator[Tuple]



#### sum()

```python
sum(iterable[, start])
```

 `sum()` 方法对 `iterable` 进行求和计算。 

 	start -- 指定相加的参数，如果没有设置这个值，默认为0。 

```python
sum([0,1,2])  			 # 3  
sum((2, 3, 4), 1)        # 元组计算总和后再加 1
						 # 10
sum([0,1,2,3,4], 2)      # 列表计算总和后再加 2
						 # 12
```



#### 同一行代码如何换行？

行末尾加上 `\`

 https://blog.csdn.net/qq_40229981/article/details/83587503 



#### set() 和 集合

 https://www.runoob.com/python3/python3-set.html 

 https://www.runoob.com/python/python-func-set.html 

 `set()` 函数创建一个无序不重复元素集合。

```python
set([iterable])
```

 可以使用大括号 `{ }` 或者 `set()` 函数创建集合，注意：创建一个空集合必须用 `set()` 而不是 `{ }`，因为

 `{ }` 是用来创建一个空字典。 



```python
a = set('abracadabra') 		# {'a', 'r', 'b', 'c', 'd'}
b = set('alacazam') 		# {'a', 'l', 'z', 'c', 'm'}
a | b                       # 并集：集合a或b中包含的所有元素
							# {'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}

a & b                       # 交集：集合a和b中都包含了的元素
							# {'a', 'c'}

a - b                       # 差集：集合a中包含而集合b中不包含的元素
							# {'r', 'd', 'b'}

a ^ b                       # 和交集互补：不同时包含于a和b的元素
							# {'r', 'd', 'b', 'm', 'z', 'l'}
```




