

原命题 <=> 逆否命题，同真假。

​                         原命题                                                        逆否命题

真命题:   <1>. 若 equals ，则 hashcode 相等。 <=> 若 hashcode 不相等，则不 equals。

​               <2>. 若不 equals ，则不一定 hashcode 不相等。<=> 若 hashcode 相等，则不一定 equals。





为什么会这样呢？

来看看 equals 和 hashcode 的关系：

假如有两个 hashcode：hashcode1 和 hashcode2

有八个对象：A1 equals A2，B1 equals B2，C1 equals C2，D1 equals D2

可能出现的情况是：

| hashcode1 | hashcode2 |
| :-------: | :-------: |
|  A1对象   |  C1对象   |
|  A2对象   |  C2对象   |
|  B1对象   |  D1对象   |
|  B2对象   |  D2对象   |

对于对象间的 equals：

A1 和 A2 equals，A1 对象的 hashcode 一定和 A2 对象的 hashcode相等。

A1，B1，C1 对象不 equals，但是 不能判断 hashcode 是否相等。



对于 hashcode 的相等：

A1，A2，B1的 hashcode 相等，但是 不能判断对象是否 equals。

A1 和 C1 的 hashcode 不相等，A1 和 C1 一定不 equals。

