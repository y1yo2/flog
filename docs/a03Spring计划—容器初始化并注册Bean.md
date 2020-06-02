# 容器初始化并注册Bean

参考资料：https://juejin.im/post/5d69e26c5188251ecc40c231

​					https://juejin.im/post/5d69e26af265da03cd0a9608

了解了Spring中的Bean 并搭好Spring FrameWork的本地运行环境之后，

我们新建测试项目并尝试运行一下代码：

```java
@Description("测试一下")
@ComponentScan
@Configuration
public class AppConfig {
}
```

创建 `AnnotationConfigApplicationContext` 容器，并注册 `AppConfig` 类。

```java
public class Main {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext context =
				new AnnotationConfigApplicationContext(AppConfig.class);
		context.close();
	}

}
```



查看 `AnnotationConfigApplicationContext(Class<?>... annotatedClasses)` 构造方法的源码：

```java
public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
	//调用默认无参构造器,里面是一堆初始化逻辑
	this();

	// 注册传入的Class,Class既可以有@Configuration注解,也可以没有@Configuration注解
	// 怎么注册? 
    // 委托给 AnnotatedBeanDefinitionReader.register 方法进行注册
    // org.springframework.context.annotation.AnnotatedBeanDefinitionReader
	// 传入Class生成 BeanDefinition , 然后注册到 BeanDefinitionRegistry
	register(annotatedClasses);

	//刷新容器上下文
	refresh();
}

```



本节先分析 `this()` 的初始化逻辑和 `register(annotatedClasses)` 的注册逻辑。



## this()

首先查看 `AnnotationConfigApplicationContext()` 的源码

```java
public AnnotationConfigApplicationContext() {
    // super();在源码中是隐式调用的，这里显示出来方便查看。
	// 隐式调用父类构造器 GenericApplicationContext
	super(); 

	// 创建 AnnotatedBeanDefinitionReader
	// 创建时会向传入的 BeanDefinitionRegistry/this 注册 注解配置相关的 processors 的 BeanDefinition
	this.reader = new AnnotatedBeanDefinitionReader(this);

	this.scanner = new ClassPathBeanDefinitionScanner(this);
}

public GenericApplicationContext() {
	this.beanFactory = new DefaultListableBeanFactory();
}
```

AnnotationConfigApplicationContext —▷ 继承 GenericApplicationContext ----> BeanFactory

AnnotationConfigApplicationContext ---▷ 实现 BeanDefinitionRegistry

**创建 Context** 时，会**创建 BeanFactory**（DefaultListableBeanFactory） ，并**实现  BeanDefinitionRegistry**（this）



### Context, BeanFactory, Registry



### Reader

然后 传入 BeanDefinitionRegistry/this，创建 AnnotatedBeanDefinitionReader。

**创建 Reader** 主要有两步，首先**从 Registry 获取并设置 Environment**，

然后**向 BeanFactory 注册 注解配置相关的 processors 的 BeanDefinition**。

创建Reader的源码：

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
	Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
	Assert.notNull(environment, "Environment must not be null");
	this.registry = registry;
	this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
	AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

重要的是 `AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);`

1. 从 Registry 获取 BeanFactory

2. 为 BeanFactory 设置属性

3. 向 BeanFactory 注册 Spring内置的 PostProcessor 的BeanDefinition（还未实例化Bean）

   其中的 **ConfigurationAnnotationProcessor** ，实现了 BeanDefinitionRegistryPostProcessor 接口。

   BeanDefinitionRegistryPostProcessor接口是为了通过Registry操作容器中的BeanDefinition，动态注册Bean到spring容器中。
   
   拓展知识：BeanFactoryPostProcessor接口，BeanDefinitionRegistryPostProcessor接口。
   
   ```java
   // 通过beanFactory获取容器中的bean
   postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory);
   
   // 通过registry操作容器中的BeanDefinition
   postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry);
   ```



### Scanner

创建 Scanner。通过查看源码可以发现，这个`scanner`只有在手动调用`AnnotationConfigApplicationContext`的一些方法的时候才会被使用（通过后面的源码探究也可以发现，spring并不是使用这个`scanner`来扫描包获取Bean的）。





## register(annotatedClasses)

初始化好 BeanFactory, Registry, Context, Reader, Scanner之后。（完成this();）

开始注册目标.class，调用 Reader 的 `doRegisterBean(Class<T> beanClass)` 

```java
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
			@Nullable BeanDefinitionCustomizer[] customizers) {

		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
			return;
		}

		abd.setInstanceSupplier(supplier);
		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
		abd.setScope(scopeMetadata.getScopeName());
		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
		if (qualifiers != null) {
			for (Class<? extends Annotation> qualifier : qualifiers) {
				if (Primary.class == qualifier) {
					abd.setPrimary(true);
				}
				else if (Lazy.class == qualifier) {
					abd.setLazyInit(true);
				}
				else {
					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
				}
			}
		}
		if (customizers != null) {
			for (BeanDefinitionCustomizer customizer : customizers) {
				customizer.customize(abd);
			}
		}

		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
	}
```



1. 根据 Class 建 BeanDef：`new AnnotatedGenericBeanDefinition(beanClass);`  

2. 解析 **Bean.Metadata == Class**

   **Class类实现 AnnotedElement接口**，`getDeclaredAnnotations()` 方法：返回该元素（类/方法/属性）上的所有注解，但忽略继承的注解。拓展：getAnnotations() 比较 getDeclaredAnnotations() 

   **获取类上的注解后：**  判断是否有通用注解@Scope，@Lazy，@Primary，@DependsOn，@Role，@Description。并将注解的内容设置进 `BeanDef` 中

3.  `definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);` 

   通过 `@Scope(value=Single/.../..., proxyMode=Interfaces/target_class/...)` ，

   判断 Singleton/Proxy：单例是初始化时创建；代理是生成代理类；

   作用域：  

   1. Singleton：初始化建一个实例
   2. Prototype：每次注入或从上下文获取时建一个新的实例
   3. Session：每个会话建一个实例
   4. Request：每个请求建一个实例

   关于非单例Bean 

   当单例Bean 开始创建时，要注入非单例的Bean，但此时Bean未生成。则通过代理模式，创建代理类的Bean然后注入。以后使用代理Bean时，会使用**懒解析并委托真正的Bean**（待拓展）。

   创建代理类的方式（proxyMode）：

   1. INTERFACES：代理的类，是实现接口的。

      ​							创建一个JDK动态代理，实现目标类公开的所有接口

   2. TARGET_CLASS：用CGLIB基于类的代理

   3. DEFAULT/NO

   然后生成 `BeanDefinitionHolder` 

4. 最后注册器注册 BDHolder

   ```java
   registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
   ```

   核心逻辑是，DefaultListableBeanFactory 里的 beanDefinitionMap，

   ConcurrentHashMap(beanName, BD)。

5. 完成注册后，容器还未实例化bean，注册了BD（Spring内置的postProcessor和传入的类）。





拓展：BDHolder 和 代理类

BDHolder 通过装饰器模式，处理功能的增强。

BDDecorator 的 decorate 方法，处理增强 BDHolder。（而不需改变BDHolder类，装饰器模式的作用）

例如： `ScopedProxyBDDecorator` 的 decorate 方法就是调用 `ScopedProxyUtils.createScopedProxy` 进行生成代理类的增强。

上面第三点 `AnnotationConfigUtils.applyScopedProxyMode` ，判断scopeMetadata需要创建代理类时，也是调用 `ScopedProxyUtils.createScopedProxy` 









