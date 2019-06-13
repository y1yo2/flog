# java泛型



为什么？

基础用法

原理

带来的问题和注意点



## 为什么使用泛型？

假设一个场景：

如果想开发一个在应用中传递对象的容器，但对象的类型不是相同的。

如何开发一个能存储各种类型对象的容器？



没有泛型之前，最好的方案是开发一个能存储和检索 Object 类型的容器。

```java
public class ObjectContainer {
    private Object obj;
    
    public Object getObject() {
        return obj;
    }
    
    public void setObject(Object obj) {
        this.obj = obj;
    }
}
```

但是该方案不是类型安全，在检索封装对象时使用显式的类型转换。



使用泛型后，使用泛型创建一个能存储和检索对象的容器。

```java
public class ObjectContainer<T> {
    private T obj;
    
    public T getObject() {
        return obj;
    }
    
    public void setObject(T obj) {
        this.obj = obj;
    }
}
```

在实例化时分配一个类型，将分配的具体类型替换泛型类型，用于限制容器内使用的值。



比较一下，使用泛型后：

1. 消除了显示的类型转换，代码量更少，也避开了可能引起的 `ClassCastException`  。
2. 编译器提供更强的类型检查，避开了可能引起的 `ClassCastException`  。





## 基础用法：

### 1. 泛型类型

```java
public class GenericContainer<T> {
}
```

**类型参数**（又称类型变量） 放在类名后，用尖括号 `<>` 包括。作用：在运行时为类分配类型。

标准的类型参数用例如下：

- T：类型
- K：键
- V：值
- E：元素
- N：数字
- S、U、V等：扩展第2、3、4个类型



类型推断，自动装箱拆箱



有界类型

在 类型参数部分 使用 extends 或 super 关键字，分别用上限或下限限制类型。

例如，如果希望将某类型限制为特定类型或特定类型的子类型，请使用以下表示法：

```java
<T extends UpperBoundType>
```

同样，如果希望将某个类型限制为特定类型或特定类型的超类型，请使用以下表示法：

```java
<T super LowerBoundType>
```

 

### 2. 泛型方法

泛型类型可用于方法参数和返回类型。

```java
public static <N extends Number> double add(N a, N b){
    double sum = 0;
    sum = a.doubleValue() + b.doubleValue();
    return sum;
}   
```



### 3. 通配符

问号 (`?`) 通配符可用于使用泛型代码表示未知类型。通配符可用于参数、字段、局部变量和返回类型。但最好不要在返回类型中使用通配符，因为确切知道方法返回的类型更安全。

```java
public static <T> void checkList(List<?> myList, T obj){
        if(myList.contains(obj)){
            System.out.println("The list contains the element: " + obj);
        } else {
            System.out.println("The list does not contain the element: " + obj);
        }
    }
```



### 4. Lambda 表达式

lambda 表达式的用例:

Lambda 表达式表示一个匿名函数，它实现函数接口的单一抽象方法。

```java
public static void compareStrings(List<String> list, Predicate<String> predicate) {
    list.stream().filter((n) -> (predicate.test(n))).forEach((n) -> {
        System.out.println(n + " ");
    });
}    

List<String> bookList = new ArrayList<>();
bookList.add("Introducing Java EE 7");
bookList.add("JavaFX 8:  Introduction By Example");
compareStrings(bookList, (n)->n.contains("Java EE"));
```





## 泛型的原理：

### 1. Java泛型的实现方法：类型擦除

Java中的泛型，是在**编译器**这个层次来实现的。在生成的**Java字节码中不包含泛型的类型信息**的。

使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为**类型擦除** (type erasure)。





### 2. 类型擦除后保留的原始类型

原始类型（raw type）就是擦除去了泛型信息，最后在字节码中的类型变量的真正类型。无论何时定义一个泛型类型，相应的原始类型都会被自动地提供。类型变量被擦除（crased），并使用其限定类型（无限定的变量用Object）替换。





## 带来的问题和注意点

### 1. 先检查，再编译

java编译器是通过先检查代码中泛型的类型，然后再进行类型擦除，在进行编译的。

```java
List<String> arrayList1=new ArrayList(); //第一种 情况
List arrayList2=new ArrayList<String>();//第二种 情况
```

第一种情况，可以实现与 完全使用泛型参数一样的效果，第二种则完全没效果。

因为，本来类型检查就是编译时完成的。new ArrayList()只是在内存中开辟一个存储空间，可以存储任何的类型对象。而**真正涉及类型检查的是它的引用**。

因为我们是使用它引用arrayList1 来调用它的方法，比如调用add()方法。所以arrayList1引用能完成泛型类型的检查。

而引用arrayList2没有使用泛型，所以不行。

通过上面的例子，我们可以明白，类型检查就是针对引用的，谁是一个引用，用这个引用调用泛型方法，就会对这个引用调用的方法进行类型检测，而无关它真正引用的对象。



### 2. 泛型类型变量不能是基本数据类型

比如，没有`ArrayList<double>` ，只有 `ArrayList<Double>` 。因为当类型擦除后，ArrayList的原始类型变为Object，但是Object类型不能存储double值，只能引用Double的值。



### 3. 运行时类型查询

运行时进行类型查询的时候使用下面的方法是错误的：

```
if( arrayList instanceof ArrayList<String>) 
```

java限定了这种类型查询的方式，下面是正确的写法：

```
if( arrayList instanceof ArrayList<?>)  
```



### 4. 泛型数组

查看sun的说明文档，在java中是**”不能创建一个确切的泛型类型的数组”**的。

采用通配符的方式是被允许的:**数组的类型不可以是类型变量，除非是采用通配符的方式**，因为对于通配符的方式，最后取出数据是要做显式的类型转换的。



### 5. 获得 T.class （获取泛型参数的Class）

对于泛型类来说，想要获取 T.class，有两种方式。

##### 1. 通过 泛型的实例

```java
t.getClass();
```



##### 2. 创建子类，继承泛型父类并实现泛型

```java
public class GenericClass<T>{
        private Class<T> entityClass;
        public GenericClass() {
            entityClass =(Class<T>) ((ParameterizedType) getClass()
                    .getGenericSuperclass()).getActualTypeArguments()[0];
        }

        public Class<T> getEntityClass() {
            return this.entityClass;
        }
}

public class GenericClassImpl extends GenericClass<String> {

}
```



方法分析：

Object类的 getClass 方法

```java
public final native Class<?> getClass()
```

Returns the runtime class of this. 返回 this 的运行时 Class。



Class类的 getGenericSuperclass 方法

```java
public Type getGenericSuperclass()
```

用来返回表示当前`Class` 所表示的实体（类、接口、基本类型或 void）的直接超类的`Type`。如果这个直接超类是参数化类型的，则返回的Type对象必须明确反映在源代码中声明时使用的类型。



如果 超类是泛型类，则`Type` 是 

```java
public class ParameterizedTypeImpl implements ParameterizedType

public interface ParameterizedType extends Type
```



如果 超类不是泛型类，则 `Type` 是

```java
public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement
```




ParameterizedTypeImpl 类的 getActualTypeArguments 方法

```java
public Type[] getActualTypeArguments()
```

Returns an array of {@code Type} objects representing the actual type arguments to this type.

返回 一个数组，里面是表示此类型的实际类型参数。





关键语句

Class < T >  entityClass  =  (Class < T > ) ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[ 0 ];

放在了超类，也就是声明泛型的那个类的构造方法中。这样一来，子类在继承具有泛型的超类时，会自动调用超类的构造方法。在此超类的构造方法中，调用的getClass返回的是子类的Class类型，则在子类中就无需再显式地使用 getGenericInterfaces()和getGenericSuperclass()等方法了。



```java
public static <T> Class<T> getClass(Class cla) {
    Class<T> result = null;
    Type type = ((ParameterizedType) cla.getGenericSuperclass()).getActualTypeArguments()[0];
    if (type instanceof ParameterizedType) {
        result = (Class<T>) ((ParameterizedType) type).getRawType();
    } else {
        result = (Class<T>) type;
    }
    return result;
}
```







