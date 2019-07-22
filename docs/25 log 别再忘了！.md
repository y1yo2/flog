<https://juejin.im/post/5a1f86f0f265da4326529c61>

混乱的 spring、logback、slf4j、log4j



思路整理：log4j ——> logback ——> slf4j ——> spring



log4j（log for java）

日志实现框架，`Ceki Gülcü` 创造，给了 `Apache` 



logback（log back）

日志实现框架， `Ceki Gülcü` 又创造了一个 //滑稽



slf4j（The Simple Logging Facade for Java）

日志门面抽象框架，提供 门面 API 和 一个简单日志类实现，配合 `Log4j，LogBack，java.util.logging` 使用。



LogBack被分为3个组件，`logback-core, logback-classic` 和 `logback-access`。

**logback-core**：提供了LogBack的核心功能，是另外两个组件的基础。

**logback-classic**：则实现了 `Slf4j` 的 `API` ，所以当想配合Slf4j使用时，需要将 `logback-classic` 加入 `classpath` 。

**logback-access**：是为了集成`Servlet`环境而准备的，可提供`HTTP-access`的日志接口。





spring 与 logback 的关系：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

spring boot 将自动使用 `logback` 作为应用日志框架。

spring boot 启动时，由 `org.springframework.boot.logging.Logging-Application-Listener` 根据情况初始化并使用。

而且 `spring-boot-starter` 包含了 `spring-boot-starter-logging` 。

所以 spring boot 的默认日志框架为 logback



配置文件 **logback-spring.xml**

**application.properties**





