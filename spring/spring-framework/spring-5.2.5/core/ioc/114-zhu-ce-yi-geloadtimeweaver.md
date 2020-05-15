### 1.14 注册一个`LoadTimeWeaver` {#context-load-time-weaver}


当`LoadTimeWeaver`类加载到Java虚拟机（JVM）中时，Spring会使用来动态转换类。

要启用加载时编织，可以将`@EnableLoadTimeWeaving`_ 添加到您的一个 `@Configuration`类，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

另外，对于XML配置，可以使用`context:load-time-weaver`元素：

```xml
<beans>
    <context:load-time-weaver/>
</beans>
```

一旦针对进行了配置，则其中的`ApplicationContext`任何bean都`ApplicationContext` 可以实现`LoadTimeWeaverAware`，从而接收到对加载时韦弗实例的引用。与[Spring的JPA支持](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/data-access.html#orm-jpa)结合使用时，该功能特别有用， 因为对于JPA类转换，可能需要加载时间编织。有关[`LocalContainerEntityManagerFactoryBean`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/orm/jpa/LocalContainerEntityManagerFactoryBean.html) 更多详细信息，请查阅 javadoc。有关AspectJ加载时编织的更多信息，请参见[Spring Framework中的AspectJ加载时编织](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aop-aj-ltw)。
