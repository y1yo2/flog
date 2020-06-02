# 创建容器的最后步骤：刷新容器上下文

参考资料：https://juejin.im/post/5d69e26a6fb9a06aef090111

​					https://blog.csdn.net/ztchun/article/details/90814135

继续分析上下文容器构造函数。

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



本节分析 `refresh()` 的刷新逻辑，下面是 `refresh()` 方法的源码。

```java
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
    	// Prepare this context for refreshing.
    	prepareRefresh();
    
    	// Tell the subclass to refresh the internal bean factory.
    	ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
    
    	// Prepare the bean factory for use in this context.
    	prepareBeanFactory(beanFactory);
    
    	try {
    		// Allows post-processing of the bean factory in context subclasses.
    		postProcessBeanFactory(beanFactory);
    
    		// Invoke factory processors registered as beans in the context.
    		invokeBeanFactoryPostProcessors(beanFactory);
    
    		// Register bean processors that intercept bean creation.
    		registerBeanPostProcessors(beanFactory);
    
    		// Initialize message source for this context.
    		initMessageSource();
    
    		// Initialize event multicaster for this context.
    		initApplicationEventMulticaster();
    
    		// Initialize other special beans in specific context subclasses.
    		onRefresh();
    
    		// Check for listener beans and register them.
    		registerListeners();
    
    		// Instantiate all remaining (non-lazy-init) singletons.
    		finishBeanFactoryInitialization(beanFactory);
    
    		// Last step: publish corresponding event.
    		finishRefresh();
    	} catch (BeansException ex) {
    		if (logger.isWarnEnabled()) {
    			logger.warn("Exception encountered during context initialization - " +
    					"cancelling refresh attempt: " + ex);
    		}
    
    		// Destroy already created singletons to avoid dangling resources.
    		destroyBeans();
    
    		// Reset 'active' flag.
    		cancelRefresh(ex);
    
    		// Propagate exception to caller.
    		throw ex;
    	} finally {
    		// Reset common introspection caches in Spring's core, since we
    		// might not ever need metadata for singleton beans anymore...
    		resetCommonCaches();
    	}
    }
}
```



逐行代码进行分析：

## synchronized (this.startupShutdownMonitor)

对象锁，synchronized 一个 final Object ，变量名叫 startupShutdownMonitor。

使容器的 "refresh" and "destroy" 互斥，即 容器的启动和关闭互斥。



## prepareRefresh();

做刷新前的准备，初始化变量和标志位。主要是启动日期、活动标志和属性源。



## obtainFreshBeanFactory();

通知父类 `GenericApplicationContext` 刷新 BeanFactory，然后返回 `ConfigurableListableBeanFactory `。

`GenericApplicationContext`  使用的工厂是 `DefaultListableBeanFactory`



##  prepareBeanFactory(beanFactory);

为 `DefaultListableBeanFactory` 配置属性。

源码如下：

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
	// Tell the internal bean factory to use the context's class loader etc.
	beanFactory.setBeanClassLoader(getClassLoader());
	beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
	beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

	// Configure the bean factory with context callbacks.
	//增加 BeanPostProcessor 实例 ApplicationContextAwareProcessor
	//ApplicationContextAwareProcessor 主要作用是对  Aware接口的支持,如果实现了相应的 Aware接口，则注入对应的资源
	beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));

	//默认情况下，只忽略BeanFactoryAware接口,现在新增忽略如下类型的自动装配
	beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
	beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
	beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
	beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
	beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
	beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

	// BeanFactory interface not registered as resolvable type in a plain factory.
	// MessageSource registered (and found for autowiring) as a bean.

	//注册自动装配规则,如果发现依赖特殊类型，就使用该指定值注入
	beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
	beanFactory.registerResolvableDependency(ResourceLoader.class, this);
	beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
	beanFactory.registerResolvableDependency(ApplicationContext.class, this);

	// Register early post-processor for detecting inner beans as ApplicationListeners.
	beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

	// Detect a LoadTimeWeaver and prepare for weaving, if found.
	if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
		beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
		// Set a temporary ClassLoader for type matching.
		beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
	}

	// Register default environment beans.
	//注册默认的environment beans.
	if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
		// Environment
		beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
	}
	if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
		// System Properties
		beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
	}
	if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
		// System Environment
		beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
	}
}
```



### beanPostProcessors，bean后置处理器

添加两个 `BeanPostProcessor` 到 `beanPostProcessors` 中。

`AbstractBeanFactory` 的一个 `CopyOnWriteArrayList`。

分别添加了：

#### ApplicationContextAwareProcessor

当 Bean 实现 Aware接口后，AwareProcessor 会通过 Aware接口的方法，为 Bean 注入Spring内部的环境变量。

Bean 实现 Spring Aware 就是为了获取 Spring内部信息。

实现不同的 Aware 接口可以获取不同的内部变量。



#### ApplicationListenerDetector

当 Bean 实现 ApplicationListener 后，将 Bean 添加到 ApplicationEventMulticaster 中。

 

### beanFactory.registerSingleton

为 beanFactory 添加好 bean后置处理器后，使用 beanFactory 注册三个 Singleton环境变量。

1. environment
2. systemProperties
3. systemEnvironment

registerSingleton 方法注册单例Bean，传入 beanName 和 singleObject。

`DefaultListableBeanFactory` 继承 `DefaultSingletonBeanRegistry` （单例Bean的注册器和管理器）

在 `SingletonBeanRegistry`  中，有个叫 `singletonObjects`的`concurrentHashMap` 存单例Bean 的 beanname 和 instance。



### prepareBeanFactory总结

prepareBeanFactory 完成后：

`DefaultListableBeanFactory` 的 `BeanDefinitionMap` 存了6个 BeanDefinition。

工厂继承的 `DefaultSingletonBeanRegistry` 的 `singletonObjects` 存了3个单例Bean

工厂继承的 `AbstractBeanFactory` 的 `beanPostProcessors` 存了2个 BeanPostProcessor



## postProcessBeanFactory(beanFactory);

这个方法是 `AbstractApplicationContext` 的一个 protected 空方法。

提供给子类重写。子类可以利用这个方法，在 refresh方法执行的时候，操作 beanFactory，

例如 注册一些自定义的 `BeanPostProcessor` 。（拓展 BeanPostProcessor 的使用和执行时机）



## invokeBeanFactoryPostProcessors(beanFactory);

这个方法作用是 实例化 和 调用 所有注册的 `BeanFactoryPostProcessor` Beans。

该方法的参数主要是 beanFactory 和 **beanFactoryPostProcessors**。

beanFactoryPostProcessors：

AbstractApplicationContext 的一个 ArrayList，只能通过 ApplicationContext的addProcessor方法增加。



具体逻辑：

beanFactory 如果实现了 BeanDefinitionRegistry（beanFactory也是一个 BDRegistry）

**优先轮询 `beanFactoryPostProcessors`** ，检查哪个 BeanFactoryPostProcessor 实现了 BeanDefinitionRegistryPostProcessor（BeanDefinitionRegistryPostProcessor继承了BeanFactoryPostProcessor），就**执行它的 postProcessBeanDefinitionRegistry 方法**（需要传入 BDRegistry）

通过 beanFactory.getBeanNamesForType，获取实现 BeanDefinitionRegistryPostProcessor 的 bean 的 name。beanFactory.isTypeMatch 判断**实现 PriorityOrdered接口**，如果实现了则执行该类的BDRpostProcessor逻辑。

同样方式，判断**实现了 Ordered接口**，则执行该类的 BDRpostProcessor逻辑。

执行**剩下的实现了 BeanDefinitionRegistryPostProcessor接口**的BDRpostProcessor逻辑。



执行刚刚所有实现 BeanDefinitionRegistryPostProcessor接口的 BeanFactoryPostProcessor逻辑。



执行 `beanFactoryPostProcessors` 中所有类的 BeanFactoryPostProcessor逻辑。

如果**实现 PriorityOrdered接口**，执行 BeanFactoryPostProcessor逻辑。

如果**实现 Ordered接口**，执行 BeanFactoryPostProcessor逻辑。

最后执行所有剩下的实现了BeanFactoryPostProcessor接口的类的 BeanFactoryPostProcessor逻辑。





## 总结 BeanPostProcessor、BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor

初步总结：

BeanPostProcessor：在Bean实例化前、后执行操作，能获取到bean的实例和名字进行操作。

BeanFactoryPostProcessor：通过 beanFactory，getBeansWithAnnotation或getBeanNamesForType，获取到某些特别的 bean，进行执行或修改操作。

BeanDefinitionRegistryPostProcessor：通过 Registry，操作BeanDefinition，注册、移除等。





































































