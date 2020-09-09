## ConfigurationClassPostProcessor 中碰到的注解



1. postProcessBeanDefinitionRegistry

加载所有 `@Configuration`，ConfigurationClassParser解析 配置类



如果类有继承@Component 的注解（@Configuration, @Controller,@Repository, @Service），则递归解析内部



解析：

`@PropertySource`

读取yml 配置放入spring上下文



`@ComponentScan`

循环所有的ComponentScan，立即执行扫描。检验扫描获得的BeanDefinition中是否有配置类，如果有配置类则递归解析。



`@Import `

加载指定的类到 IOC容器



`@ImportResource`

加载XML配置文件，加载xml的bean



`@Bean`

将Bean 放入IOC容器





2.postProcessBeanFactory

是否 @Configuration 注解，且proxyBeanMethods=true(默认)，则为Full配置类；否则，为LITE配置类。

收集所有**Full配置类**，加强beanDef中的Class。

如何**加强**？用enhancer生成cglib的代理子类，设置 Interceptor。

BeanMethodInterceptor：拦截方法调用获取Bean，改为从上下文中获取。

BeanFactoryAwareMethodInterceptor：支持BeanFactoryAware，为继承BeanFactoryAware的

​																		@Configuration类传入BeanFactory实例



2.给 BeanFactory 添加 ImportAwareBeanPostProcessor

该BeanPostProcess支持：BeanFactoryAware、ImportAware（传入AnnotationMetadata）





## 自动装配遇到的注解

META/INF 的 spring.factories 的 EnableAutoConfiguration配置的类



`@Configuration` 



`@ConditionalOnClass({A.class, B.class}) ` 

必须有以下（A，B）的类，才会创建该类



使用 @ConditionalOnClass 和 @ConditionalOnMissingBean ，确保仅当找到相关的类且未声明自己的@Configuration时，才应用该类进行自动配置。



`@AutoConfigureAfter` 和 `@AutoConfigureBefore`

按照特定顺序加载 config类



`@ConfigurationProperties(prefix="??")` 

读取配置文件中，以 ??? 开头的属性并和 bean的属性匹配赋值。

通常用于 @Configuration 类或者 **@EnableConfigurationProperties**({???.class}) 指定的读取配置文件的类。



对比

`@Value `

注解取配置文件中的值 到属性



 `@PropertySource` 

加载yml 配置 到spring上下文



`@ImportSource`

指定 加载xml配置文件 到 spring上下文中。



`@Import` 

导入配置类：@Configuration注解 或 实现了**ImportSelector**/ImportBeanDefinitionRegistrar。



参考资料：

https://juejin.im/post/6844904174660894727#heading-25