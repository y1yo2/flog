```java
This class consists exclusively of static methods that operate on or return
* collections.  It contains polymorphic algorithms that operate on
* collections, "wrappers", which return a new collection backed by a
* specified collection, and a few other odds and ends.
```



# Collections

Collections 是操作 Collection 的工具类，例如 swap、sort。



其中的 

```java
<T> void sort(List<T> list, Comparator<? super T> c)
```



# Comparator 接口如何使用？

实现 `int compare(T o1, T o2);` 方法。该方法的参数与返回值的作用。

```java
o1 the first object to be compared.
o2 the second object to be compared.
a negative integer, zero, or a positive integer as the first argument is less than, equal to, or greater than the second.
```



返回值是：负数/零/正数，表示第一个参数比第二个参数 小/相等/大。



例子：

```java
Integer[] intArray = {0,5,1,7,2,3};               
List<Integer> intList = Arrays.asList(intArray);

Collections.sort(intList, (Integer first, Integer second) -> {first-second}); 
// 由小到大排序，满足第一个参数比第二个参数大时，返回正数
// [0,1,2,3,4,5,7]

Collections.sort(intList, (Integer first, Integer second) -> {second-first}); 
// 由大到小排序，第一个参数比第二个参数大时，返回了负数。
// [7, 5, 3, 2, 1, 0]
```



# HashMap

 https://juejin.im/post/5aa5d8d26fb9a028d2079264#heading-21 

HashMap(1.7):

​		`Entry<K, V>` 接口;

​		`Node(key, value, next)` 实现 Entry;

​		`Node[] table` : Entry 实现类的数组;



1. newHashMap(): 初始化 容量大小、阈值的变量。

2. put(): 第一次 put 时，初始化存储数组 table

   ​			key 为 null 时，放在 table[0]

   

   ​			生成 Hash 值：hashcode 加扰动处理。

   ​			生成数组位置（i）：Hash 值 与运算（&） 数组长度-1

   ​			

   ​			获得 table[i] 的对象 Entry，遍历 next，key 存在则替换；不存在，则插入。

   

   ​			插入：先判断容量，是否扩容。

   ​						插入操作：

   ```java
   Entry newEntry = new Entry(hash, key, value, next=table[i]);
   table[i] = newEntry;
   ```

   将该元素插入 table[i]，next为之前的 table[i]。链表的头部插入。

   ​			



HashMap(1.8):

​		`Entry<K, V>` 接口;

​		`Node(key, value, next)` 实现 Entry;

​		`TreeNode<K, V>` 实现 `LinkedHashMap.Entry<K, V>`

​		`Node[] table` : Entry 实现类的数组;



`capacity` (容量) * `loadfactor` (加载因子) —> `threshold` (扩容阈值) 



1. newHashMap()，初始化：初始化容量（16） 和 加载因子（0.75）

2. put(): 第一次 put 时，初始化：扩容阈值 = 容量*加载因子 = 12 

   ​			初始化 table = Node[16] 

   ​			插入 Node：

      1. tab[i] 为 null，插入

      2. tab[i]：key 相等时，插入；不是链表是红黑树，树形插入；

         ​				遍历链表，是否有 key 相等，插入；

         ​				没有 key 相等，最后新增 Node；

         ​				判断链表长度大于 树化阈值，链表转红黑树。

3. 链表转红黑树：

   ​		如果 table 小于 最小树化容量，resize

   ​		如果 table.size 大于 阈值=容量*加载因子，resize

   ​		否则，红黑树化











