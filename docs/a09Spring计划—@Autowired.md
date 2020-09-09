@Autowired的使用和原理



@Autowired、@Resource和@Inject的区别

https://juejin.im/post/6844904158252761102

Spring注解@Autowired源码分析

https://juejin.im/post/6844903960491327501#heading-3

Spring @Autowired 注入小技巧

https://juejin.im/post/6844903644819619853



简单总结：

#### 注解处理器

​    在Spring框架内部实现当中，注解实现注入主要是通过bean后置处理器BeanPostPocessor接口的实现类来生效的。BeanPostProcessor后置处理器是在spring容器启动时，创建bean对象实例后，马上执行的，对bean对象实例进行加工处理。     

- @Autowired是通过BeanPostProcessor接口的实现类**AutowiredAnnotationBeanPostProcessor**来实现对bean对象对其他bean对象的依赖注入的；
- @Resource和@Inject是通过BeanPostProcessor接口的实现类CommonAnnotationBeanPostProcessor来实现的，顾名思义即公共注解CommonAnotation，CommonAnnotationBeanPostProcessor是Spring中统一处理JDK中定义的注解的一个BeanPostProcessor。该类会处理的注解还包括@PostConstruct，@PreDestroy等。















