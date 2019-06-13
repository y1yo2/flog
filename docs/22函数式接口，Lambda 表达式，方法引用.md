# 函数式接口，Lambda 表达式，方法引用

- 函数式接口

   函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。
```java
public interface Consumer<T> {

    void accept(T t);

}
```

- Lambda 表达式
   使用Lambda表达式来表示该接口的一个实现(注：JAVA 8 之前一般是用匿名类实现的)。
```java
public class LambdaApplication {

    private static void userConsumer(Consumer c){}

    private static void useLambda(){
        //使用lamda实现接口的实例
        Consumer one = t -> System.out.println(Objects.isNull(t));
        //使用lamda实现接口的实例用于方法参数
        useConsumer(t -> System.out.println(Objects.isNull(t)));
    }
}
```
_lambda 表达式只能引用标记了 final 的外层局部变量_


- 方法引用
   方法引用通过方法的名字来指向一个方法（实际上可认为是一种Lambda表达式）。
```java
public  class Application {
    
    public static Application create(final Supplier<Application> supplier) {
        return supplier.get();
    }
 
    public static void staticMethod(final Application app) {
        System.out.println("staticMethod" + app.toString());
    }
 
    public void commonMethod(final Application another) {
        System.out.println("commonMethod" + another.toString());
    }
 
    public void repair() {
        System.out.println("Repaired " + this.toString());
    }

    public static void methodRefer(){
        //构造方法引用：Class::new，或Class< T >::new
        final Application app = Application .create( Application ::new );
        final List< Application > apps= Arrays.asList( app);

        //静态方法引用：Class::static_method
        apps.forEach( Application ::staticMethod);

        //类型的普通方法引用：Class::methodName
        apps.forEach( Application ::commonMethod);

        //实例的方法引用：instanceReference::methodName
        apps.forEach( app::repair );
    }
    /**
     * 超类上的实例方法引用：super::methodName
     * 数组构造方法引用：TypeName[]::new
     */
}
```