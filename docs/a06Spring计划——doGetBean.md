# doGetBean

`AbstractApplicationContext` 的 `finishBeanFactoryInitialization`，初始化所有还未被初始化的单例bean。

用到了 `getBean(BeanName)` 方法创建或获取对应的bean，调用 `AbstractBeanFactory#doGetBean` 。

`doGetBean` 源码如下：

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
						  @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

	// 如果这个 name 是 FactoryBean 的beanName (&+beanName),就删除& , 返回beanName ,传入的name也可以是别名,也需要做转换
	// 注意 beanName 和 name 变量的区别,beanName是经过处理的,经过处理的beanName就直接对应singletonObjects中的key
	final String beanName = transformedBeanName(name);
	Object bean;

	// Eagerly check singleton cache for manually registered singletons.
	// 根据beanName尝试从singletonObjects获取Bean
	// 获取不到则再尝试从earlySingletonObjects,singletonFactories 从获取Bean
	// 这段代码和解决循环依赖有关
	Object sharedInstance = getSingleton(beanName);
	// 第一次进入，sharedInstance肯定为null
	if (sharedInstance != null && args == null) {
		if (logger.isTraceEnabled()) {
			if (isSingletonCurrentlyInCreation(beanName)) {
				logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
						"' that is not fully initialized yet - a consequence of a circular reference");
			} else {
				logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
			}
		}
		// 如果sharedInstance不为null,也就是非第一次进入
		// 为什么要调用 getObjectForBeanInstance 方法,判断当前Bean是不是FactoryBean,如果是,那么要不要调用getObject方法
		// 因为传入的name变量如果是(&+beanName),那么beanName变量就是(beanName),也就是说,程序在这里要返回FactoryBean
		// 如果传入的name变量(beanName),那么beanName变量也是(beanName),但是,之前获取的sharedInstance可能是FactoryBean,需要通过sharedInstance来获取对应的Bean
		// 如果传入的name变量(beanName),那么beanName变量也是(beanName),获取的sharedInstance就是对应的Bean的话,就直接返回Bean
		bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
	} else {
		// Fail if we're already creating this bean instance:
		// We're assumably within a circular reference.
		// 判断是否循环依赖
		if (isPrototypeCurrentlyInCreation(beanName)) {
			throw new BeanCurrentlyInCreationException(beanName);
		}

		// Check if bean definition exists in this factory.
		// 获取父BeanFactory,一般情况下,父BeanFactory为null,如果存在父BeanFactory,就先去父级容器去查找
		BeanFactory parentBeanFactory = getParentBeanFactory();
		if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
			// Not found -> check parent.
			String nameToLookup = originalBeanName(name);
			if (parentBeanFactory instanceof AbstractBeanFactory) {
				return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
						nameToLookup, requiredType, args, typeCheckOnly);
			} else if (args != null) {
				// Delegation to parent with explicit args.
				return (T) parentBeanFactory.getBean(nameToLookup, args);
			} else if (requiredType != null) {
				// No args -> delegate to standard getBean method.
				return parentBeanFactory.getBean(nameToLookup, requiredType);
			} else {
				return (T) parentBeanFactory.getBean(nameToLookup);
			}
		}

		// 创建的Bean是否需要进行类型验证,一般情况下都不需要
		if (!typeCheckOnly) {
			// 标记 bean 已经被创建
			markBeanAsCreated(beanName);
		}

		try {
			// 获取其父类Bean定义,子类合并父类公共属性
			final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
			checkMergedBeanDefinition(mbd, beanName, args);

			// Guarantee initialization of beans that the current bean depends on.
			// 获取当前Bean依赖的Bean的名称 ,@DependsOn
			String[] dependsOn = mbd.getDependsOn();
			if (dependsOn != null) {
				for (String dep : dependsOn) {
					if (isDependent(beanName, dep)) {
						throw new BeanCreationException(mbd.getResourceDescription(), beanName,
								"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
					}
					// 如果当前Bean依赖其他Bean,把被依赖Bean注册给当前Bean
					registerDependentBean(dep, beanName);
					try {
						// 先去创建所依赖的Bean
						getBean(dep);
					} catch (NoSuchBeanDefinitionException ex) {
						throw new BeanCreationException(mbd.getResourceDescription(), beanName,
								"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
					}
				}
			}

			// Create bean instance.
			if (mbd.isSingleton()) {
				// 创建单例Bean
				sharedInstance = getSingleton(beanName, () -> {
					try {
						return createBean(beanName, mbd, args);
					} catch (BeansException ex) {
						// Explicitly remove instance from singleton cache: It might have been put there
						// eagerly by the creation process, to allow for circular reference resolution.
						// Also remove any beans that received a temporary reference to the bean.
						destroySingleton(beanName);
						throw ex;
					}
				});
				bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
			} else if (mbd.isPrototype()) {
				// It's a prototype -> create a new instance.
				// 创建prototype Bean,每次都会创建一个新的对象
				Object prototypeInstance = null;
				try {
					// 回调beforePrototypeCreation方法，注册当前创建的原型对象
					beforePrototypeCreation(beanName);
					// 创建对象
					prototypeInstance = createBean(beanName, mbd, args);
				} finally {
					// 回调 afterPrototypeCreation 方法，告诉容器该Bean的原型对象不再创建
					afterPrototypeCreation(beanName);
				}
				bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
			} else {
				// 如果既不是单例Bean,也不是prototype,则获取其Scope
				String scopeName = mbd.getScope();
				final Scope scope = this.scopes.get(scopeName);
				if (scope == null) {
					throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
				}
				try {
					// 创建对象
					Object scopedInstance = scope.get(beanName, () -> {
						beforePrototypeCreation(beanName);
						try {
							return createBean(beanName, mbd, args);
						} finally {
							afterPrototypeCreation(beanName);
						}
					});
					bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
				} catch (IllegalStateException ex) {
					throw new BeanCreationException(beanName,
							"Scope '" + scopeName + "' is not active for the current thread; consider " +
									"defining a scoped proxy for this bean if you intend to refer to it from a singleton",
							ex);
				}
			}
		} catch (BeansException ex) {
			cleanupAfterBeanCreationFailure(beanName);
			throw ex;
		}
	}

	// Check if required type matches the type of the actual bean instance.
	// 对创建的Bean进行类型检查
	if (requiredType != null && !requiredType.isInstance(bean)) {
		try {
			T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
			if (convertedBean == null) {
				throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
			}
			return convertedBean;
		} catch (TypeMismatchException ex) {
			if (logger.isTraceEnabled()) {
				logger.trace("Failed to convert bean '" + name + "' to required type '" +
						ClassUtils.getQualifiedName(requiredType) + "'", ex);
			}
			throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
		}
	}
	return (T) bean;
}

```

doGetBean 逻辑分析：

1. transformedBeanName(name); 通过 name 获取 beanName。

2. `sharedInstance = getSingleton(beanName);` 通过 beanName **从 singletonObjects 获取 Bean**。

3. 如果 sharedInstance 不为 null，`getObjectForBeanInstance(sharedInstance, name, beanName);`
   通过 getObjectForBeanInstance 方法获取 bean。

4. 如果 sharedInstance 为 null，执行如下逻辑：

5. `isPrototypeCurrentlyInCreation(beanName)` **判断循环依赖**。

6. 获取 父BeanFactory，如果不为null，先在 父BeanFactory 中寻找

7. 获取 bean 的父类定义，**子类合并父类公共属性**
   `RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);`

   `checkMergedBeanDefinition(mbd, beanName, args);` 

8. 获取是否有依赖的其他 bean，**@DependsOn**。如果依赖其他 bean，先将 依赖beans 注册给bean，优先创建 依赖bean。
   `deps = mbd.getDependsOn();` 
   `registerDependentBean(dep, beanName);` 
   `getBean(dep);` 

9. mbd.isSingleton() 判断是否单例。
   `createBean(beanName, mbd, args);` 
   `getObjectForBeanInstance(sharedInstance, name, beanName, mbd);` 
    生成单例 bean。

10. mbd.isPrototype() 判断是否为 原型。
    `createBean(beanName, mbd, args);`
    `getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);` 
    prototype bean，每次都创建一个新的对象

11. 其他 Scope（Session, Request）。

    `createBean(beanName, mbd, args);`
    `getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);` 



具体方法分析：

## Object sharedInstance = getSingleton(beanName);

解决循环依赖的关键，与 `isPrototypeCurrentlyInCreation` 方法配合。

源码如下：

```java
public Object getSingleton(String beanName) {
	return getSingleton(beanName, true);
}

//getSingleton(beanName, true);源码
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	//singletonObjects 就是Spring内部用来存放单例Bean的对象池,key为beanName，value为Bean
	Object singletonObject = this.singletonObjects.get(beanName);
	// singletonsCurrentlyInCreation 存放了当前正在创建的bean的BeanName
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		synchronized (this.singletonObjects) {
			// earlySingletonObjects 是早期单例Bean的缓存池,此时Bean已经被创建(newInstance),但是还没有完成初始化
			// key为beanName，value为Bean
			singletonObject = this.earlySingletonObjects.get(beanName);
			//是否允许早期依赖
			if (singletonObject == null && allowEarlyReference) {
				//singletonFactories 单例工厂的缓存,key为beanName,value 为ObjectFactory
				ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
				if (singletonFactory != null) {
					//获取早期Bean
					singletonObject = singletonFactory.getObject();
					//将早期Bean放到earlySingletonObjects中
					this.earlySingletonObjects.put(beanName, singletonObject);
					this.singletonFactories.remove(beanName);
				}
			}
		}
	}
	return singletonObject;
}
```



## getObjectForBeanInstance

`transformedBeanName`  和 `getObjectForBeanInstance` 主要是处理，bean 有可能是 **BeanFactory** ，则需要从 BeanFactory 中继续获取 Bean。

主要情况如下：

#### 第一种，直接往容器中注册 Bean。

```java
@Bean
public UserBean userBean() {
	return new UserBean("shen", 111);
}
```

name 为 “userBean”

调 `transformedBeanName(name)` 获得 "userBean"。

从容器获得 `sharedInstance` 为 UserBean。

调 `getObjectForBeanInstance(sharedInstance, name, beanName, null)` 获得 UserBean。



#### 第二种情况，通过`FactoryBean`往容器中注册 Bean

```java
@Bean
public FactoryBean userBean() {
	return new FactoryBean<UserBean>() {
		@Override
		public UserBean getObject() throws Exception {
			return new UserBean("shen", 111);
		}

		@Override
		public Class<?> getObjectType() {
			return UserBean.class;
		}
	};
}
```

这里分为两种情况：

##### 第一种情况：

name 为 “userBean”

调 `transformedBeanName(name)` 获得 "userBean"。

从容器获得 `sharedInstance` 为一个 FactoryBean。

调 `getObjectForBeanInstance(sharedInstance, name, beanName, null)` 获得 UserBean。

（从FactoryBean中调getObject获得 UserBean）



##### 第二种情况：

name 为 “&userBean”

调 `transformedBeanName(name)` 获得 "userBean"。

从容器获得 `sharedInstance` 为一个 FactoryBean。

调 `getObjectForBeanInstance(sharedInstance, name, beanName, null)` 获得 FactoryBean。

（判断 name 为 "&userBean"，返回 FactoryBean）



## @DependsOn

Spring 在创建 Bean 之前，首先会创建当前 Bean 所有依赖的 Bean。

关键源码如下：

```java
String[] dependsOn = mbd.getDependsOn();

for{
    // 如果当前Bean依赖其他Bean,把被依赖Bean注册给当前Bean
	registerDependentBean(dep, beanName);
    
    // 先去创建所依赖的Bean
    getBean(dep);
}
```



关于 `@DependsOn` ：

当前 bean 依赖其他 bean，保证依赖的bean在此 bean 之前由容器创建。
当 依赖的bean 不是通过属性或构造函数参数显式注入。



例子：

```java
@Service
public class OrderService {

	public OrderService() {
		System.out.println("OrderService create");
	}

	@PreDestroy
	public void destroy() {
		System.out.println("OrderService destroy");
	}
}
```

```java
@DependsOn("orderService")
@Service
public class UserService {

	public UserService() {
		System.out.println("UserService create");
	}

	@PreDestroy
	public void destroy() {
		System.out.println("UserService destroy");
	}
}
```

```
//运行结果
OrderService create
UserService create
UserService destroy
OrderService destroy
```

总结：可以控制 创建顺序，也可控制 销毁顺序。



## getSingleton(beanName, () -> {createBean(beanName, mbd, args);})

```java
// 创建单例Bean
sharedInstance = getSingleton(beanName, () -> {
    try {
        return createBean(beanName, mbd, args);
    } catch (BeansException ex) {
        // Explicitly remove instance from singleton cache: It might have been put there
        // eagerly by the creation process, to allow for circular reference resolution.
        // Also remove any beans that received a temporary reference to the bean.
        destroySingleton(beanName);
        throw ex;
    }
});
```



`getSingleton(String beanName, ObjectFactory<?> singletonFactory)` 方法的源码如下：

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
	Assert.notNull(beanName, "Bean name must not be null");
	synchronized (this.singletonObjects) {
		//判断单例Bean是否已经存在,如果存在,则直接返回
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null) {
			if (this.singletonsCurrentlyInDestruction) {
				throw new BeanCreationNotAllowedException(beanName,
						"Singleton bean creation not allowed while singletons of this factory are in destruction " +
								"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
			}
			if (logger.isDebugEnabled()) {
				logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
			}
			// 创建单例之前调用该方法,将此Bean标记为正在创建中,用来检测循环依赖
			beforeSingletonCreation(beanName);
			boolean newSingleton = false;
			boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
			if (recordSuppressedExceptions) {
				this.suppressedExceptions = new LinkedHashSet<>();
			}
			try {
				// 通过方法传入的ObjectFactory<?> singletonFactory来创建Bean
				singletonObject = singletonFactory.getObject();
				newSingleton = true;
			} catch (IllegalStateException ex) {
				// Has the singleton object implicitly appeared in the meantime ->
				// if yes, proceed with it since the exception indicates that state.
				singletonObject = this.singletonObjects.get(beanName);
				if (singletonObject == null) {
					throw ex;
				}
			} catch (BeanCreationException ex) {
				if (recordSuppressedExceptions) {
					for (Exception suppressedException : this.suppressedExceptions) {
						ex.addRelatedCause(suppressedException);
					}
				}
				throw ex;
			} finally {
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = null;
				}
				// 创建单例之后调用该方法,将单例标记为不在创建中
				afterSingletonCreation(beanName);
			}
			if (newSingleton) {
				// 加入到单例池容器中
				addSingleton(beanName, singletonObject);
			}
		}
		return singletonObject;
	}
}
```

通过 beforeSingletonCreation(beanName); 标记 bean 为创建中。afterSingletonCreation(beanName); 标记 bean 为不在创建中。用来检测循环依赖。

`singletonObject = singletonFactory.getObject();` ，调用 singletonFactory 的 getObject 方法创建 bean。此处使用 `AbstractAutowireCapableBeanFactory#createBean` 方法实现 getObject逻辑。 



## createBean(beanName, mbd, args);

可以发现，Singleton、Prototype、其他Scope（Session，Request），创建 Bean时都是调用这个方法。

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)		throws BeanCreationException {
	if (logger.isTraceEnabled()) {
		logger.trace("Creating instance of bean '" + beanName + "'");
	}
	RootBeanDefinition mbdToUse = mbd;

	// Make sure bean class is actually resolved at this point, and
	// clone the bean definition in case of a dynamically resolved Class
	// which cannot be stored in the shared merged bean definition.
	// 为指定的bean定义解析bean类，将bean类名称解析为Class引用（如果需要,并将解析后的Class存储在bean定义中以备将来使用),也就是通过类加载去加载这个Class
	Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
	if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
		mbdToUse = new RootBeanDefinition(mbd);
		mbdToUse.setBeanClass(resolvedClass);
	}

	// Prepare method overrides.
	try {
		// 校验和准备 Bean 中的方法覆盖
		mbdToUse.prepareMethodOverrides();
	} catch (BeanDefinitionValidationException ex) {
		throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
				beanName, "Validation of method overrides failed", ex);
	}

	try {
		// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
		// 执行BeanPostProcessors , 给 BeanPostProcessors 一个机会直接返回代理对象来代替Bean实例
		// 在Bean还没有开始实例化之前执行 InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation方法,这个方法可能会直接返回Bean
		// 如果这里直接返回了Bean,那么这里返回的Bean可能是被经过处理的Bean(可能是代理对象)
		// 在这里需要注意Spring方法中的两个单词: Instantiation 和 Initialization , 实例化和初始化, 先实例化,然后初始化
		Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
		if (bean != null) {
			// 如果bean不为null,则直接返回,不在做后续处理
			return bean;
		}
	} catch (Throwable ex) {
		throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
				"BeanPostProcessor before instantiation of bean failed", ex);
	}

	try {
		// 创建Bean
		Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		if (logger.isTraceEnabled()) {
			logger.trace("Finished creating instance of bean '" + beanName + "'");
		}
		return beanInstance;
	} catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
		// A previously detected exception with proper bean creation context already,
		// or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
		throw ex;
	} catch (Throwable ex) {
		throw new BeanCreationException(
				mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
	}
}
```

1. 调用`resolveBeanClass(mbd, beanName);`方法，为指定的bean定义解析bean类
2. 调用`mbdToUse.prepareMethodOverrides();`方法，校验和准备 Bean中的方法覆盖
3. 调用`resolveBeforeInstantiation(beanName, mbdToUse);`方法，给`BeanPostProcessors`一个机会直接返回代理对象来代替Bean实例
4. 调用`doCreateBean(beanName, mbdToUse, args);`方法，真正执行Bean的创建



















































参考文章：https://juejin.im/post/5d7cd448f265da03c9272882