# 创建容器的最后步骤：刷新容器上下文（二）



```
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
```



## registerBeanPostProcessors(beanFactory);

注册BeanPostProcessors

从beanDefinitionNames中获取类型为BeanPostProcessor的所有beanName

遍历所有的postProcessorNames，将其分类：

- 实现`PriorityOrdered`接口的
- 实现`MergedBeanDefinitionPostProcessor`接口的
- 实现`Ordered`接的
- 普通的`BeanPostProcessor`

按一定是先后顺序依次执行所有的 `BeanPostProcessor` 。







## initMessageSource();

初始化信息源,作国际化相关

`initMessageSource`方法负责，初始化信息源,是一些国际化相关功能。



## initApplicationEventMulticaster();

初始化容器实现传播器,也就是往容器中添加了一个Bean

```java
protected void initApplicationEventMulticaster() {
	ConfigurableListableBeanFactory beanFactory = getBeanFactory();
	if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
		this.applicationEventMulticaster =
				beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
	} else {
		this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
		beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
	}
}
```





## onRefresh();

在特定 ApplicationContext 的子类中触发某些特殊的Bean初始化

在此处AbstractApplicationContext.onRefresh 是一个空方法

`onRefresh`方法在此处`AbstractApplicationContext.onRefresh`是一个空方法，其作用是在特定`ApplicationContext`的子类中触发某些特殊的Bean初始化。



## registerListeners();

注册`ApplicationListener`，源码如下：

```java
protected void registerListeners() {
	// Register statically specified listeners first.
	// 获取 通过AbstractApplicationContext.addApplicationListener 加入的listener
	for (ApplicationListener<?> listener : getApplicationListeners()) {
		getApplicationEventMulticaster().addApplicationListener(listener);
	}

	// Do not initialize FactoryBeans here: We need to leave all regular beans
	// uninitialized to let post-processors apply to them!
	// 获取ApplicationListener的实现类，默认情况下,这里也是空
	String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
	for (String listenerBeanName : listenerBeanNames) {
		getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
	}

	// Publish early application events now that we finally have a multicaster...
	// 通过传播器发布事件，默认情况下,这里还是空
	Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
	this.earlyApplicationEvents = null;
	if (earlyEventsToProcess != null) {
		for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
			getApplicationEventMulticaster().multicastEvent(earlyEvent);
		}
	}
}
```



## finishBeanFactoryInitialization(beanFactory);

初始化所有还未被初始化的单例bean

初始化所有还未被初始化的单例bean。 `AbstractApplicationContext#finishBeanFactoryInitialization` 调用`DefaultListableBeanFactory#preInstantiateSingletons`：



```java
//DefaultListableBeanFactory#preInstantiateSingletons源码：
public void preInstantiateSingletons() throws BeansException {
	// Iterate over a copy to allow for init methods which in turn register new bean definitions.
	// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
	// 获取所有的beanDefinitionNames
	List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

	// Trigger initialization of all non-lazy singleton beans...
	// 遍历所有的beanDefinitionNames
	for (String beanName : beanNames) {
		// 根据指定的beanName获取其父类的相关公共属性,返回合并的RootBeanDefinition
		RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
		// 如果不是抽象类,而且是单例,又不是懒加载
		if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
			// 判断是不是FactoryBean
			if (isFactoryBean(beanName)) {
				// 如果是FactoryBean,使用 &+beanName ,去获取 FactoryBean
				// 为什么要这样做,因为beanName获取的是FactoryBean生产的Bean,要获取FactoryBean本身,需要通过&+beanName
				// 其实,实例化所有的非懒加载单例Bean的时候,如果是FactoryBean,这里只是创建了FactoryBean
				// 什么时候去创建由FactoryBean产生的Bean呢? 好像也是懒加载的,在使用到这个Bean的时候,才通过FactoryBean去创建Bean
				Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
				if (bean instanceof FactoryBean) {
					final FactoryBean<?> factory = (FactoryBean<?>) bean;
					boolean isEagerInit;
					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
						isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
										((SmartFactoryBean<?>) factory)::isEagerInit,
								getAccessControlContext());
					} else {
						isEagerInit = (factory instanceof SmartFactoryBean &&
								((SmartFactoryBean<?>) factory).isEagerInit());
					}
					if (isEagerInit) {
						getBean(beanName);
					}
				}
			} else {
				// 不是FactoryBean
				getBean(beanName);
			}
		}
	}

	// Trigger post-initialization callback for all applicable beans...
	for (String beanName : beanNames) {
		Object singletonInstance = getSingleton(beanName);
		// Spring容器的一个拓展点SmartInitializingSingleton
		// 在所有非懒加载单例Bean创建完成之后调用该接口 @since 4.1
		if (singletonInstance instanceof SmartInitializingSingleton) {
			final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
			if (System.getSecurityManager() != null) {
				AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
					smartSingleton.afterSingletonsInstantiated();
					return null;
				}, getAccessControlContext());
			} else {
				smartSingleton.afterSingletonsInstantiated();
			}
		}
	}
}
```

1. 获取所有的beanDefinitionNames，遍历：

2. 先合并其父类的相关公共属性,返回合并的 `RootBeanDefinition` 

3. 如果不是抽象类,而且是非懒加载的单例，开始创建Bean：

4. 首先判断是不是 `FactoryBean`，如果是FactoryBean,使用 `&+beanName` ,去获取 FactoryBean。

   如果不是，调用 `getBean(beanName)` **创建或者获取对应的Bean**。

5. `SmartInitializingSingleton` 是`Spring4.1`版本之后的一个新扩展点。在创建完所有的非懒加载单例Bean后，获取 `SmartInitializingSingleton`接口实现类的bean并调用，完成回调。



getBean方法调用 `AbstractBeanFactory#doGetBean` ，下节主要分析doGetBean逻辑。




## finishRefresh();

容器启动完成，清理缓存，发布`ContextRefreshedEvent`事件。

```java
// Publish the final event.
publishEvent(new ContextRefreshedEvent(this));
```



TODO：补充EventMulticaster，Listener，Event相关知识



参考文章：https://juejin.im/post/5d7afbc6518825345a05c549#heading-0















