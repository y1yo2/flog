spring-framework

参考资料：https://juejin.im/post/5d69e2696fb9a06aff5e8854



从 git 克隆 5.1x 版本到本地。

克隆过慢：先克隆到码云，再从码云克隆到本地。



项目导入 idea

问题一：Gradle项目 build 时报错，plugin with id 'java-test-fixtures' not found；

　　　　原因：构建的 gradle 版本不能低于5.6 也不能高于6.0。

问题二：build 过慢，需要配置镜像。

​				修改项目文件中的 `build.gradle` ，添加 `buildscript` 和 `allprojects` ：

```json
buildscript {
    repositories {
        maven { url 'https://plugins.gradle.org/m2/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/google' }
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
    }
}

allprojects {
    repositories {
        maven { url 'https://plugins.gradle.org/m2/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/google' }
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
}
```



问题三：报错：CoroutinesUtils cannot be resolved

1. `spring-core` 工程下，找到 `kotlin-coroutines` 工程
2. 在 `kotlin-coroutines` 的 `src/main/kotlin/org/springframeword/core` 根位置，点右键 `Build Module` 
3. 在  `kotlin-coroutines` 的 `build` 文件夹下成功生成 `CoroutinesUtils.class` 













