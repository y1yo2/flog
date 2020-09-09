# Spring boot 源码分析



### 核心注解：@SpringBootApplication

里面包含三个注解：

@SpringBootConfiguration：就是一个@Configuration，配置类 解析引入 Bean到IOC

@ComponentScan：包扫描 其他组件

@EnableAutoConfiguration：开启自动配置功能





### 启动过程

1.分析 SpringApplication的 run方法

```java
public ConfigurableApplicationContext run(String... args) {
    // stopWatch 用于统计run启动过程的花费时间
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    
    // 定义应用上下文 ConfigurableApplicationContext
    // 该上下文加载应用所需的类和各种环境配置
    ConfigurableApplicationContext context = null;
    
    // 异常报告集合，报告 springboot启动时异常
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    // “java.awt.headless”属性，默认为ture，设置没有显示器也能启动。
    configureHeadlessProperty();
    
    // SpringApplicationRunListeners监听器
    // 【1】.从 spring.factories加载 EventPublishingRunListener的实现类，放到 listeners
    // EventPublishingRunListener作用是发送 springboot启动过程的一些事件，标志每个不同启动阶段
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 启动SpringApplicationRunListeners监听器，表示开始启动
    // 》》》发送 【ApplicationStartingEvent】 事件
    listeners.starting();
    
    
    try {
        // applicationArguments封装args
       ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        //【2】.准备环境变量：系统变量，环境变量，命令行参数，默认变量，servlet相关配置变量，JNDI属性值
        // 配置文件（比如application.properties）等，存在优先级问题
        // 》》》发送【ApplicationEnvironmentPreparedEvent】事件
       ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        
        // 配置spring.beaninfo.ignore属性，默认为true，跳过搜索BeanInfo classes
        configureIgnoreBeanInfo(environment);
        // 【3】控制台打印SpringBoot的bannner标志
        Banner printedBanner = printBanner(environment);
        
        // 【4】根据环境创建对应类型的 spring applicationcontext 应用上下文
        // servlet环境，创建 AnnotationConfigServletWebServerApplicationContext 容器对象
        // ConfigurableApplicationContext是AnnotationConfigServletWebServerAC的父接口
        // createApplicationContext可拓展
        context = createApplicationContext();
        
        //【5】从spring.factories配置文件中加载异常报告期实例：FailureAnalyzers
        // 传入context是因为FailureAnalyzers从context获取beanFactory和environment
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);
        
        //【6】初始化 context
        //1.属性AnnotatedBeanDefinitionReader和ClassPathBeanDefinitionScanner设置environmen
        //2.对ApplicationContext应用一些后置处理，比如设置resourceLoader属性等
        //3.容器刷新前，调各个ApplicationContextInitializer的初始方法
        //4.》》》发送【ApplicationContextInitializedEvent】事件，标志context初始化完成
        //5.从context获取beanFactory，并向beanFactory注册一些单例bean，比如applicationArguments，printedBanner
        //6.》》》发送【ApplicationPreparedEvent】，标志context准备完成
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        
        //【7】刷新 context容器，spring的refresh逻辑
        refreshContext(context);
        
        //【8】执行刷新容器后的后置处理逻辑，空方法待拓展
        afterRefresh(context, applicationArguments);
        //停止stopWatch计时
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        //》》》发送【ApplicationStartedEvent】事件，标志context刷新完毕，已加载所有bean
        listeners.started(context);
        //【9】调用ApplicationRunner和CommandLineRunner的run方法，实现spring容器启动后需要做的一些东西比如加载一些业务数据等
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        //【10】出现异常，使用FailureAnalyzers报告
        //》》》并发送【ApplicationFailedEvent】事件，标志springboot启动失败
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        //》》》发送【ApplicationReadyEvent】事件，标志springboot启动成功，可以接收服务请求
        listeners.running(context);
    }
    catch (Throwable ex) {
        //出现异常，使用FailureAnalyzers报告
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

2.精简总结：

1. 从 `spring.factories` 加载 `EventPublishingRunListener` ，该listener拥有 `SimpleApplicationEventMulticaster` ，在启动过程的不同阶段发送**事件**。

2. **环境变量** environment

3. 根据环境创建 applicationcontext 。servlet环境创建`AnnotationConfigServletWebServerApplicationContext` **容器**

4. SpringApplication类初始化时会从 `spring.factories` 加载 `ApplicationContextInitializer` 的实现。

   创建容器后会调用各个 `ApplicationContextInitializer` 执行**初始化**方法

5. 从 `spring.factories` 加载 `FailureAnalyzers`对象，**报告**SpringBoot启动过程的**异常**

6. **刷新容器**，与 spring的 refresh类似

   1. 调用 beanFactory 后置处理器（BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor）
   2. ConfigurationClassPostProcessor 加载所有 BeanDefinition
   3. 注册 BeanPostProcessors
   4. 初始化事件广播器且广播事件
   5. 初始化所有未被初始化的单例 bean
   6. SpringBoot创建内嵌的`Tomcat`服务器（待拓展）

7. 完成刷新容器的后置处理，空方法提供用户扩展

8. `ApplicationRunner`和`CommandLineRunner`的run方法，实现这两个接口可以在spring容器启动后，加载一些业务数据等;

9. 若启动过程中抛出异常，用`FailureAnalyzers`来报告异常;



发送的事件分别是：

1. ApplicationStartingEvent：发送事件的listener启动了
2. ApplicationEnvironmentPreparedEvent：环境environment准备好了
3. ApplicationContextInitializedEvent：context使用 Initialized 初始化完成
4. ApplicationPreparedEvent：context准备的bean和后置处理器，准备完成
5. ApplicationStartedEvent：context刷新完成，已加载所有bean
6. ApplicationReadyEvent：springboot启动成功，开始接收请求
7. ApplicationFailedEvent：springboot启动失败







### 详解自动配置功能：

@EnableAutoConfiguration 开启自动配置功能

注解源码如下：

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

关键代码：`@Import(AutoConfigurationImportSelector.class)` 

加载 MainClass时，会加载 AutoConfigurationImportSelector。

解读 `AutoConfigurationImportSelector` 的关键源码

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
   if (!isEnabled(annotationMetadata)) {
      return NO_IMPORTS;
   }
   AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
   return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}

/**
 * Return the {@link AutoConfigurationEntry} based on the {@link AnnotationMetadata}
 * of the importing {@link Configuration @Configuration} class.
 * @param annotationMetadata the annotation metadata of the configuration class
 * @return the auto-configurations that should be imported
 */
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
   if (!isEnabled(annotationMetadata)) {
      return EMPTY_ENTRY;
   }
   AnnotationAttributes attributes = getAttributes(annotationMetadata);
   List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
   configurations = removeDuplicates(configurations);
   Set<String> exclusions = getExclusions(annotationMetadata, attributes);
   checkExcludedClasses(configurations, exclusions);
   configurations.removeAll(exclusions);
   configurations = getConfigurationClassFilter().filter(configurations);
   fireAutoConfigurationImportEvents(configurations, exclusions);
   return new AutoConfigurationEntry(configurations, exclusions);
}
```



`selectImports`：判断 enableautoconfiguration 配置，是否有开启自动装配功能



`getAutoConfigurationEntry`：

获取 @EnableAutoConfiguration注解的 Class，

调 SpringFactoriesLoader.loadFactoryNames(Class, ClassLoader) ，

获得 EnableAutoConfiguration的ClassName在 META-INF/spring.factories 的 List<String> 



SpringFactoriesLoader：

​	loadSpringFactories 方法：

​		classLoader.getResources(META-INF/spring.factories)

​		通过 resource 加载 Map<String, List<String>>

​	loadFactoryNames

​		Class.getName

​		在 Map中找到 List<String>



1.容器刷新时，ConfigurationClassParser 的 processImports方法，会调用 selector.selectImports



2.SpringApplication的 getSpringFactoriesInstances 方法

​	调 LoadFactoryNames 通过Class 拿到 List<String> 

​	通过 String，Class.forName 拿到 Class

​		Class.getDeclaredConstructor

​		constructor.newInstance 获得对象

​	最后通过 @Order排序获取 第一个对象





### spring boot 与 spring mvc

重点代码还是在 SpringApplication 的 run方法。

主要是 `context = createApplicationContext();` 和 `refreshContext(context)` 



创建上下文时，判断 webApplicationType：

SERVLET：AnnotationConfigServletWebServerApplicationContext

REACTIVE：AnnotationConfigReactiveWebServerApplicationContext

default：AnnotationConfigApplicationContext



AnnotationConfigServletWebServerApplicationContext 

继承 ServletWebServerApplicationContext 

继承 AbstractApplicationContext



刷新上下文是 AbstractApplicationContext 的 refresh方法

内部会调用 ServletWebServerApplicationContext 的 onRefresh方法

```java
// ServletWebServerApplicationContext.onRefresh
protected void onRefresh() {
	super.onRefresh();
	try {
		this.createWebServer();
	} catch (Throwable var2) {}
}
```

 关键方法 ServletWebServerApplicationContext 的 createWebServer 方法

内部调用 ServletWebServerFactory 的 getWebServer 方法



ServletWebServerFactory 有4个实现类

```java
//实现
AbstractServletWebServerFactory
JettyServletWebServerFactory
TomcatServletWebServerFactory
UndertowServletWebServerFactory
```



一个tomcat只包含一个Server，一个Server可以包含多个Service，一个Service只有一个Container，但有多个Connector，这样一个服务可以处理多个连接。 多个Connector和一个Container就形成了一个Service，有了Service就可以对外提供服务了，但是Service要提供服务又必须提供一个宿主环境，那就非Server莫属了，所以整个tomcat的声明周期都由Server控制。



一个tomcat实例就是一个Server，一个Server包含多个Service，也就是多个应用程序，每个Service包含多个Connector和一个Container，Container用于封装和管理Servlet，以及处理具体的Request请求。



### 配置文件的功能 与 加载源码详解：

##### bootstrap 与 application

Spring Boot 中有两种上下文，一种是 bootstrap, 另外一种是 application， bootstrap 是application的父上下文，也就是说 bootstrap 加载优先于 applicaton。

- boostrap 由父 ApplicationContext 加载，比 applicaton 优先加载
- boostrap 里面的属性不能被覆盖
- 从额外的资源来加载配置信息，使用 Spring Cloud Config 时，需要在 bootstrap 配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；

- 一些固定的不能被覆盖的属性
- 一些加密/解密的场景；



##### 参数加载顺序

1. 当使用了`Devtools`时，`$HOME/.config/spring-boot`文件夹中的Devtools全局设置属性

2. `@TestPropertySource`针对对测试的注解
3. 测试的`properties`。在`@SpringBootTest`和测试注释中提供，用于测试应用程序的特定部分
4. 命令行参数
5. 来自`SPRING_APPLICATION_JSON`（内嵌在环境变量或系统属性中的JSON）的属性
6. `ServletConfig`初始化参数
7. `ServletContext`初始化参数
8. JNDI属性：`java:comp/env`
9. Java系统属性: `System.getProperties()`
10. 操作系统环境变量
11. `RandomValuePropertySource`，其属性仅在`random.*`中
12. 打包`jar之外`的特定于概要文件的应用程序属性（如`application-{profile}.properties`和对应的YAML变量）

打包`在jar中`的特定于概要文件的应用程序属性（如`application-{profile}.properties`和YAML变量）

打包`jar之外`的应用程序属性（`application.properties`和YAML变量）

打包`在jar中`的应用程序属性（`application.properties`和YAML变量）

`@Configuration`类上的`@PropertySource`注解

默认属性（通过设置`SpringApplication.setDefaultProperties`指定）



我的回答：

1. 命令行参数
2. 系统环境变量
3. jar外的 application-{profile}.properties
4. jar中的 application-{profile}.properties
5. jar外的 application
6. jar中的 application
7. `@Configuration` 类的 `@PropertySource` 
8. `SpringApplication.setDefaultProperties` 设置默认值



比如在应用程序类路径（例如，打包`在jar内`）上，可以有一个`application.properties`文件，该文件为`name`属性设置了默认属性值。在新环境中运行时，可以`在jar外部`提供`application.properties`文件，该文件覆盖会覆盖`在jar内`的`application.properties`。又如对于一次性测试，可以使用特定的命令行开关启动（例如，`java -jar app.jar --name="Spring"`）也可以覆盖`name`属性值。又如可以JSON格式环境变量`$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar`来覆盖。其他方式就不一一举例了。



总结：

越外层的 配置优先级越高，读外层的配置



参考资料：

Spring Boot 核心配置文件 bootstrap & application 详解

https://zhuanlan.zhihu.com/p/63326667

Spring Boot从零入门7_最新配置文件配置及优先级详细介绍

https://juejin.im/post/6844904002707013645





其他：

常用的starter有什么呢？















参考资料：

Spring Boot自动配置实现原理、Spring Boot数据访问实现原理、微服务核心技术实现原理：

https://juejin.im/post/6844903565547290632

SpringBoot的启动流程是怎样的？SpringBoot源码（七）

https://juejin.im/post/6844904099897409550#heading-4

自动配置解析：https://juejin.im/post/6844903828819558413

SpringBoot是如何实现自动配置的？ SpringBoot源码（四）

https://juejin.im/post/6844904080872046599



SpringBoot内置tomcat启动原理：https://zhuanlan.zhihu.com/p/98562027









