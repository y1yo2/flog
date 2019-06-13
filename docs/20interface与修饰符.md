interface 与 修饰符



## 1. 接口

`interface` 可以使用 **public** 或 **默认** 或 **abstract**（is redundant）进行修饰。

```java
public interface TestInterface1 {}

interface TestInterface2 {}
```

使用其他修饰符会编译错误：private、protected、static、final



## 2. 内部接口

类的内部的 `interface` 可以使用 public、默认、protected、private、abstract、static 进行修饰

```java
public class AClass {
    private static abstarct interface TestInterface1 {}
}
```

使用其他修饰符会编译错误：final



## 3. 接口中的方法

接口中的方法如果不加修饰符，则默认是 public

如果加范围限制的修饰符，只能够加 public

```java
public interface TestInterface1 {
    void test1();
    public void test2();
    // 范围都是 public
}
```



implements，extends 后，重写方法修饰符范围只能大于或等于。

```java
public abstract class AbsClass {
    abstract void Function();
}

public class Test extends AbsClass {
    
    // 只能是 public 或者 默认
    @Override
    public void Function(){
        
    }
    
}
```



default 修饰符

可以修饰接口中已实现的方法。

default 修饰的方法通过实例调用。





static 修饰符

可以修饰接口中已实现的方法。

static 修饰的方法通过接口名调用。



## 4. 接口中的变量

不填修饰符默认是 public static final

> 好像你主动用 public，static，final 单独修饰变量，变量最后的修饰符都是 public static final

```java
System.out.println(Modifier.toString(TestInterface.class.getDeclaredField("a").getModifiers()));
```









