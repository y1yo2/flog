> 学习资料：
>
> [java 栈和栈帧](https://www.cnblogs.com/minisculestep/articles/4934947.html "With a Title") 
> [Java的堆，栈，方法区你都搞清楚了吗？](https://juejin.im/entry/5a4ece456fb9a01ca6029670#%E6%96%B9%E6%B3%95%E5%8C%BA "With a Title")
> [java基础：String — 字符串常量池与intern(二）](<https://juejin.im/post/5c160420518825235a05301e> "With a Title")
> [Java8内存模型—永久代(PermGen)和元空间(Metaspace)](https://www.cnblogs.com/paddix/p/5309550.html "With a Title")
> [Java 反射修改 final 属性值](https://blog.csdn.net/tabactivity/article/details/50726353 "With a Title")
> [JVM JIT编译能改变某些反射的执行结果](https://zhuanlan.zhihu.com/p/29763246 "With a Title")
> [深入理解 Java 基本数据类型](https://juejin.im/post/5c851ab2e51d4520d33183c3 "With a Title")
> [享元设计模式分析以及构建简单对象缓存池](https://juejin.im/post/5a340ea7f265da432d282a34 "With a Title")




# JVM 基础一

jvm 经常被提及的内存区域是：栈、堆、方法区

假设对这几个区域都有大概的认识，我从一个方法的执行入手分析



## 1. JVM的栈、堆、方法区

**栈内存**是线程私有的，当线程执行一个方法，这个方法作为栈帧入栈。

方法内部的局部变量在栈帧内分配内存（局部变量表）。

原理：

jvm 为每个线程分配一个栈，以帧为单位 保存线程的状态。

jvm 对栈的操作只有两种：以帧为单位的 **压栈** 和 **出栈** 。

栈帧的组成：

- 局部变量表
- 操作数栈
- 动态链接
- 方法返回地址



对象初始化（创建对象），在**堆内存**开辟空间，堆内存的地址给引用类型变量。

堆内存非线程私有。

堆内存的对象可同时被多个线程访问，并非线程安全。

对象内的基本类型的成员变量也保存在堆。

> 特殊情况：JVM 对方法做逃逸分析优化
>
> **如果** 对象在方法内创建，它的引用不离开方法，对象的生命周期随着栈帧出栈结束，
>
> JVM **可能** 把该对象分配在栈内，则该对象的基本类型成员变量保存在栈上。



**方法区**

方法区和堆一样，是各个线程共享的内存区域，不需要连续的内存，可选择 固定/扩展大小

用于存储：

1. 虚拟机已加载的类信息。

   类的版本、字段、方法、接口等描述信息

2. 常量池（Constant Pool Table）

   编译器生成的字面量和符号引用：

   - 字面量是：final 修饰的成员变量和类变量（static），也就是除final修饰的局部变量。

   - 符号引用：属于编译原理方面的概念，包括 
     1.类和接口的全限定名(即路径，包名+类名)。 
     2.字段的名称和描述符。 
     3.方法的名称和描述符。  

     当虚拟机运行时，需要从常量池获得对应的符号引用。再在类创建或运行时解析、翻译到具体的内存地址之中（直接引用）。

   常量池除了这些内容，还有：

   - 编译期间，类的直接引用，存储到运行时常量池
   - 运行期间，通过代码将新的内容放入运行时常量池



> 补充关于栈的知识：
>
> 上述的栈内存是用于执行 JVM 方法，又称 JVM 栈
>
> 还有复杂执行 JVM 使用的 Native 方法的栈，称为 Native 栈
>
> **JVM 栈、Native 栈**
>
> JVM 栈执行 Java 方法（字节码）
>
> JVM 使用的 Native 方法



## 2. 方法中定义不同类型的变量

**方法中定义的变量都存储在栈内存中**，但是不同类型的变量存储的内容不一样。

基本数据类型：

  根据类型分配内存，初始化值

引用类型：

  存储对象在堆中的地址



## 3. 字符串也是引用类型

#### 字符串和其他普通对象有什么区别呢？

源码分析：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
/** The value is used for character storage. */
private final char value[];
```

内部存储一个 `private final` 的 `char[]`，用于存储 String 的内容。

String 类用 final 修饰。而且 String 类没有提供方法修改 char[]，即 String 对象的值。

（一般情况下无法修改 String 对象的 char[]，String **不可变性的基础**）



#### 那 String 的 `trim` 和 `replace` 等方法呢？

这些方法都是返回一个 `new String`



#### 字符串常量池（也称为：全局字符串常量池）是什么呢？

Java 防止创建过多 String 对象的一种优化机制。

原理是：

使用双引号 `""` （字面量）创建的字符串，在堆中创建对象后，对象的引用插入到字符串常量池中。

当遇到用这种方式（字面量）创建的字符串，如果内容相同，则不需要再次创建。



```java
String str1 = “hello”;
String str2 = “hello”;
  
System.out.printl(str1 == str2) // true，str1和str2指向同一个对象 
```



#### 字符串常量池的原理是什么？

**jdk1.6** 中，常量池在永久代（方法区），存储的是**对象**。

**jdk1.7** 中，常量池在堆中，存储的是**引用**（堆中的对象的地址）。



例子分析：

```java
String str1 = new String("a") + new String("a");
str1.intern();
String str2 = "aa";
System.out.println(str1 == str2);
```

该代码在 j6 输出 `false`，在 j7 输出 `true`



j6：

1. `str1;` 创建常量池的对象 "a"，然后在堆中创建两个 String 对象（内部字符数组内容是 "a"），然后通过StringBuilder连接，在堆中创建一个 String 对象（内容是 "aa"）
2. `str1.intern();` 常量池中寻找内容为 "aa" 的对象，如果没有，在常量池创建内容为 "aa" 的对象，返回该对象的地址。
3. `str2;` "aa"（字面量）创建字符串，先在常量池中寻找内容为 "aa" 的对象，如果有，返回该对象地址。
4. `str1 == str2` str1 指向堆的一个对象（内容是 "aa"），str2 指向常量池的一个对象（内容是 "aa"）。



j7：

1. `str1;` 堆中创建对象 "a"，然后在常量池保存 "a" 对象的引用。在堆中创建两个 String 对象（内容是 "a"），通过StringBuilder连接，在堆中创建一个 String 对象（内容是 "aa"）
2. `str1.intern();` 常量池中寻找内容为 "aa" 的对象，如果没有，将 str1 指向的 String 对象的地址保存在常量池中，返回该对象的地址。
3. `str2;` "aa"（字面量）创建字符串，先在常量池中寻找内容为 "aa" 的对象，如果有，返回该对象地址（str1 指向的 String 对象）。
4. `str1 == str2` str1 指向堆的一个对象（内容是 "aa"），str2 指向堆的**同一个对象**（内容是 "aa"）。





#### 关于永久代

通俗的说，永久代 是以前实现 方法区（字符串常量池就是方法区的一部分） 的一种方式。

从 jdk1.7 开始，逐渐移除永久代。例如：符号引用(Symbols)转移到了native heap；字面量(interned strings)转移到了java heap；类的静态变量(class statics)转移到了java heap。

从 jdk1.8 开始，永久代被移除，取而代之的是 **元空间**（**Metaspace**）。



##### 元空间 是什么呢？

元空间的本质和永久代类似，都是对 JVM 规范中方法区的实现。

不过元空间与永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存**。

因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过参数来指定元空间的大小。



**为什么要取代 永久代 呢？**

1. 字符串（字符串常量池）存在永久代中，容易出现性能问题和内存溢出。

2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。（元空间使用本地内存更有弹性）

3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

4. Oracle 可能会将 HotSpot 与 JRockit 合二为一。



#### 字面量 和 字符串常量池

最后说一下 字面量 和 字符串常量池 的关系

对 **HotSpot VM** 的实现来说，**加载类**的时候，那些**字符串字面量**会进入到**当前类的运行时常量池**，不会进入**全局的字符串常量池** ;

在字面量赋值的时候，会翻译成字节码 ldc 指令，ldc 指令触发 **lazy resolution** 动作

> - 到当前类的运行时常量池（runtime constant pool，HotSpot VM里是ConstantPool + ConstantPoolCache）去查找该index对应的项
> - 如果该项尚未resolve则resolve之，并返回resolve后的内容。
> - 在遇到String类型常量时，resolve的过程如果发现StringTable已经有了内容匹配的java.lang.String的引用，则直接返回这个引用;
> - 如果StringTable里尚未有内容匹配的String实例的引用，则会在Java堆里创建一个对应内容的String对象，然后在StringTable记录下这个引用，并返回这个引用出去。



#### 字面量 和 `+` 

如果是 `+` 含有对象，则底部是使用StringBuilder实现的拼接。（01文档有说明）

```java
String str1 ="str1";
String str2 ="str2";
String str3 = str1 + str2; // StringBuilder
```

如果相加的参数只有字面量或者常量或基础类型变量，则会直接编译为拼接后的字符串。

```java
String str1 =1+"str2"+"str3";
```


 如果使用字面量拼接的话，java常量池里是不会保存拼接的参数的，而是直接编译成拼接后的字符串保存。

```java
String str1 = new String("aa"+"bb");
String str2 = new StringBuilder("a").append("a").toString();
System.out.println(str2==str2.intern());
String str3 = new StringBuilder("aab").append("b").toString();
System.out.println(str3==str3.intern());
```

输出 `true` 和 `false` 。

创建 str1 时，虽然用了字面量 “aa” 和 "bb"，但是字符串常量池中没有 "aa" 和 "bb"，有 "aabb" 。

`str2==str2.intern()` 返回 `true` 。因为常量池没有 "aa" ，intern 后常量池指向的对象就是 str2 指向的对象。

`str3==str3.intern()` 返回 `false`。因为常量池有 "aabb"，常量池指向的对象和 str3 指向的对象不是同一个。





## 4. 修改String对象的内容

String 对象的内容是：

```java
private final char value[];
```

其实就是如何修改 `private final` 修饰的属性值。



```java
Test test = new Test();
// 获取Test类的属性
Field proFinField = Test.class.getDeclaredField("privateFinal");
// 因为该属性被private修饰，需要设置为可被访问
proFinField.setAccessible(true);
// Field类中的 modifiers 属性记录着该属性的修饰符，我们需要修改proFinField的modifiers属性
// 获取Field类的modifiers属性
Field field = Field.class.getDeclaredField("modifiers");
// 因为该属性被private修饰，需要设置为可被访问
field.setAccessible(true);
// 修改proFinField的modifiers属性，将final修饰符去掉
field.setInt(proFinField, proFinField.getModifiers() & ~Modifier.FINAL);

// 设置为test对象修改privateFinal属性的属性值
proFinField.set(test, "change!");

// 上述操作虽然成功修改，但是这里输出的值可能还是属性原本的值
System.out.println("改变后：" + test.getPrivateFinal());
```



在修改 final 修饰的属性值时，要留意它的常量值本身是否被编译器内联优化。

否则会出现虽然没有异常，但取出的还是原来的值。



像字符串字面量，基本数据类型等在编译时能确定的值，用 final 修饰后，编译器会执行内联优化。

```java
System.out.println(test.getPrivateFinal()); 
// 编译后其实是：System.out.println("initValue"); 
```



JIT 编译也会出现内联优化。

JIT 编译：JVM的运行时优化——当代码频繁执行时，会触发JIT编译。



## 5. 常量池总结

1. **Class常量池**：主要存放两大类常量：字面量和符号引用。

2. **运行时常量池**：编译期生成的各种字面量和符号引用，将在类加载后进入方法区的运行时常量池。

3. **字符串常量池**：本质是一个 `HashSet<String>` ，运行时惰性维护。只存储String对象的引用，并根据引用在堆中获得 String 对象。

4. **6种基本类型的包装类的常量池**：分别是 Byte,Short,Integer,Long,Character,Boolean



#### 基本类型的装箱和拆箱

基本数据类型与包装类的转换被称为`装箱`和`拆箱`。

- `装箱`（boxing）是将值类型转换为引用类型。例如：`int` 转  `Integer`
- 装箱过程是通过调用包装类的 `valueOf` 方法实现的。
  
- `拆箱`（unboxing）是将引用类型转换为值类型。例如：`Integer` 转  `int`
- 拆箱过程是通过调用包装类的 `xxxValue` 方法实现的。（xxx 代表对应的基本数据类型）。



以 Integer 类为例：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];
    static {
        // 完成缓存初始化
    }
}
```



5种包装类型（Boolean只有 True 和 False）默认创建了[-128，127]的相应类型的缓存数据，当调用 valueof 装箱时，缓存范围内的数据（常量池的数据）会直接返回，超出此范围才会去创建新的对象。



只有 Float 和 Double 没有使用缓存数据。



#### 构建对象缓存池和享元模式

享元模式：复用存在的细粒度对象，并且在对象不需要的时候回收，方便再次使用，减少系统的消耗。



具体例子说明：

```java
// 享元接口或者（抽象享元类），定义共享接口
public interface FlyWeight {
 void cell();
}

// 其中一个实现类
public class Food implements FlyWeight{
 private String name;
    
 Food(String name){
  this.name = name;
 }
    
 @Override
 public void cell() {
  System.out.println("卖的食品是："+this.name);
 }
}
```



```java
// 享元工厂类，控制实例的创建和共享
public class FlyWeightFactory {
    
    private Map<String, FlyWeight> foodPools = new HashMap<String, FlyWeight>();
    private static FlyWeightFactory factory = new FlyWeightFactory();
    
    public static FlyWeightFactory getInstance(){
        return factory;
    }
    
    // 通过名字获取对象，使用了常量池
    public FlyWeight getFood(String foodName){
        FlyWeight food = null;
        if (foodPools.containsKey(foodName)) {
            food = foodPools.get(foodName);
        }else{
            food = new Food(foodName);
            foodPools.put(foodName, food);
        }
        return food;
    }
    
    public int getTotal(){
        return foodPools.size();
    }

}
```



所以 享元模式 的核心就是创建一个常量池保存常用的对象，减少创建重复对象。

主要有两种使用方式：

1. 创建一个工厂类管理这个常量池。
2. 在实现类的内部，创建内部类保存常量池，实现类可以直接使用。



## 6. 调用方法时值传递的解析

回顾一开始提到的，执行方法时，线程使用栈空间。

那方法执行的时候，形参的改变会对实参造成什么影响呢？



**基本类型参数：**

  形参在栈中保存一个值。当你进行修改时，只是修改形参的值，对实参没有影响。



**引用类型参数：**

  形参在栈中保存一个引用，引用指向堆的一个对象，而实参也是引用这个对象。

  当你修改该对象时，实参也会受到影响。



**字符串类型参数：**

   形参在栈中保存一个引用，引用指向堆的一个 String 对象，而实参也是引用这个 String 对象。

   但是因为 String 的不可变性，你无法修改 String 对象的内容，你只是修改了形参保存的引用地址

  （指向堆中另一个 String 对象）。















