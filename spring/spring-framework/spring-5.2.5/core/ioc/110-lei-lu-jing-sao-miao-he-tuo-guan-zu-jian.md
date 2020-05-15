### 1.10 类路径扫描和托管组件 {#beans-classpath-scanning}

本章中的大多数示例都使用XML来指定`BeanDefinition`在Spring容器中生成每个配置的配置元数据。上一节（[基于注释的容器配置](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-annotation-config)）演示了如何通过源级注释提供大量配置元数据。但是，即使在这些示例中，“基本” bean定义也已在XML文件中明确定义，而注释仅驱动依赖项注入。本节介绍了通过扫描类路径来隐式检测候选组件的选项。候选组件是与过滤条件匹配的类，并具有在容器中注册的相应bean定义。这消除了使用XML进行bean注册的需要。相反，您可以使用注释（例如`@Component`），AspectJ类型表达式或您自己的自定义过滤条件来选择哪些类已向容器注册了bean定义。

> 从Spring 3.0开始，Spring JavaConfig项目提供的许多功能是核心Spring Framework的一部分。这使您可以使用Java而不是使用传统的XML文件来定义bean。看看的`@Configuration`，`@Bean`， `@Import`，和`@DependsOn`注释有关如何使用这些新功能的例子。

#### 1.10.1 @Component`和更多的刻板印象注释

的`@Repository`注释是针对满足的存储库（也被称为数据访问对象或DAO）的作用或者固定型的任何类的标记。如[Exception Translation中](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/data-access.html#orm-exception-translation)所述，此标记的用途是自动翻译 [异常](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/data-access.html#orm-exception-translation)。

Spring提供进一步典型化注解：`@Component`，`@Service`，和 `@Controller`。`@Component`是任何Spring托管组件的通用构造型。 `@Repository`，`@Service`和`@Controller`是`@Component`针对更特定用例的专业化（分别在持久性，服务和表示层）。因此，您可以来注解你的组件类有 `@Component`，但是，通过与注解它们`@Repository`，`@Service`或者`@Controller` ，你的类能更好地适合于通过工具处理，或与切面进行关联。例如，这些构造型注释成为切入点的理想目标。`@Repository`，`@Service`和，并且`@Controller`在Spring框架的将来版本中还可以包含其他语义。因此，如果您选择使用`@Component`或`@Service`对于您的服务层，`@Service`显然是更好的选择。同样，如前所述，`@Repository`在持久层中已经支持作为自动异常转换的标记。

#### 1.10.2。使用元注释和组合注释

Spring提供的许多注释都可以在您自己的代码中用作元注释。元注释是可以应用于另一个注释的注释。例如，`@Service`注释提及[早期](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-stereotype-annotations) 为间注释有`@Component`，如下面的示例所示：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

    // ...
}
```

> 将`Component`导致`@Service`以同样的方式来对待`@Component`。

您还可以结合使用元注释来创建“组合注释”。例如，`@RestController`Spring MVC中的注释由`@Controller`和 组成`@ResponseBody`。

此外，组合注释可以选择从元注释中重新声明属性，以允许自定义。当您只希望公开元注释属性的子集时，这特别有用。例如，Spring的 `@SessionScope`注释将作用域名称硬编码为，`session`但仍允许自定义`proxyMode`。以下清单显示了`SessionScope`注释的定义 ：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```

然后，您`@SessionScope`无需声明`proxyMode`以下即可使用：

```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

您还可以覆盖的值`proxyMode`，如以下示例所示：

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

有关更多详细信息，请参见 [Spring Annotation编程模型](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model) Wiki页面。

#### 1.10.3 自动检测类并注册Bean定义

Spring可以自动检测构造型类，并使用来注册相应的 `BeanDefinition`实例`ApplicationContext`。例如，以下两个类别可进行这种自动检测：

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```



```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的bean，您需要添加 `@ComponentScan`到`@Configuration`类中，其中`basePackages`属性是两个类的公共父包。（或者，您可以指定一个逗号分隔，分号分隔或空格分隔的列表，其中包括每个类的父包。）

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

> 为简洁起见，前面的示例可能使用`value`了注释的属性（即`@ComponentScan("org.example")`）。

以下替代方法使用XML：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

> 使用``隐式启用的功能 ``。``使用时，通常无需包含 元素``。

>   扫描类路径包需要在类路径中存在相应的目录条目。使用Ant构建JAR时，请确保不要激活JAR任务的仅文件开关。此外，在某些环境中，可能不会基于安全策略公开类路径目录，例如，在JDK 1.7.0_45及更高版本上的独立应用程序（这需要在清单中设置“受信任的库”，请参见 [https://stackoverflow.com/ Questions / 19394570 / java-jre-7u45-breaks-classloader-getresources](https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources)）。在JDK 9的模块路径（Jigsaw）上，Spring的类路径扫描通常可以按预期进行。但是，请确保在您的`module-info` 描述符中导出了组件类。如果您期望Spring调用您的类的非公共成员，请确保它们是“打开的”（也就是说，它们使用`opens`声明而不是描述符中的 `exports`声明`module-info`）。

此外，当您使用component-scan元素时，`AutowiredAnnotationBeanPostProcessor`和 `CommonAnnotationBeanPostProcessor`都隐式包括在内。这意味着将自动检测这两个组件并将它们连接在一起，而这一切都不需要XML中提供的任何bean配置元数据。

> 您可以禁用注册`AutowiredAnnotationBeanPostProcessor`并 `CommonAnnotationBeanPostProcessor`通过包括`annotation-config`与属性的值`false`。

#### 1.10.4 使用过滤器自定义扫描

默认情况下，类注有`@Component`，`@Repository`，`@Service`，`@Controller`， `@Configuration`，或自定义的注释，与自身的注解`@Component`是唯一检测到的候选组件。但是，您可以通过应用自定义过滤器来修改和扩展此行为。将它们添加为`includeFilters`或注释的`excludeFilters`属性`@ComponentScan`（或XML配置中元素的``或 ``子元素``）。每个过滤器元素都需要`type`和`expression`属性。下表描述了过滤选项：

| 过滤器类型   | 范例表达                     | 描述                                                         |
| :----------- | :--------------------------- | :----------------------------------------------------------- |
| 注释（默认） | `org.example.SomeAnnotation` | 在目标组件中的类型级别上*存在*或*元存在*的注释。             |
| 可分配的     | `org.example.SomeClass`      | 目标组件可分配给（扩展或实现）的类（或接口）。               |
| 方面         | `org.example..*Service+`     | 目标组件要匹配的AspectJ类型表达式。                          |
| 正则表达式   | `org\.example\.Default.*`    | 要与目标组件的类名匹配的正则表达式。                         |
| 习俗         | `org.example.MyTypeFilter`   | `org.springframework.core.type.TypeFilter`接口的自定义实现。 |

以下示例显示了忽略所有`@Repository`注释并改为使用“存根”存储库的配置：

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

以下清单显示了等效的XML：

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

> 您也可以通过设置`useDefaultFilters=false`注释或通过提供元素`use-default-filters="false"`的属性来 禁用默认过滤器``。这样有效地禁止的注释的或与间注解的类自动检测`@Component`，`@Repository`，`@Service`，`@Controller`， `@RestController`，或`@Configuration`。

#### 1.10.5。在组件中定义Bean元数据

Spring组件还可以将bean定义元数据贡献给容器。您可以`@Bean`使用与在带`@Configuration` 注释的类中定义Bean元数据相同的注释来执行此操作。以下示例显示了如何执行此操作：

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

上一类是Spring组件，其`doWork()`方法中包含特定于应用程序的代码 。但是，它也提供了具有工厂方法的bean定义，该工厂方法引用了method `publicInstance()`。该`@Bean`注释标识工厂方法和其它bean定义特性，如通过一个限定值`@Qualifier`注释。可以指定其他方法级别的注解是 `@Scope`，`@Lazy`和自定义限定器注解。

> 除了用于组件初始化的角色外，您还可以将`@Lazy`注释放置在标有`@Autowired`或的注入点上`@Inject`。在这种情况下，它导致注入了惰性解析代理。

如前所述，支持自动连线的字段和方法，并自动支持`@Bean`方法的附加支持。以下示例显示了如何执行此操作：

```java
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

该示例将`String`方法参数自动连接`country`到`age` 另一个名为的bean上的属性值`privateInstance`。Spring Expression Language元素通过符号定义属性的值`#{  }`。对于`@Value` 注释，表达式解析器已预先配置为在解析表达式文本时查找bean名称。

从Spring Framework 4.3开始，您还可以声明类型`InjectionPoint`（或其更具体的子类：）的工厂方法参数， `DependencyDescriptor`以访问触发当前bean创建的请求注入点。注意，这仅适用于实际创建bean实例，而不适用于注入现有实例。因此，此功能对原型范围的bean最有意义。对于其他作用域，factory方法仅在给定作用域中看到触发创建新bean实例的注入点（例如，触发创建惰性单例bean的依赖项）。在这种情况下，可以将提供的注入点元数据与语义一起使用。以下示例显示如何使用`InjectionPoint`：

```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```

将`@Bean`在普通的Spring组件方法比春天里的同行处理方式不同`@Configuration`类。不同之处在于`@Component` CGLIB并未增强类来拦截方法和字段的调用。CGLIB代理是一种方法，通过该方法可以调用类中的方法或`@Bean`方法中的字段来`@Configuration`创建Bean元数据引用以协作对象。此类方法不是用普通的Java语义调用的，而是通过容器进行的，以提供Spring Bean的常规生命周期管理和代理，即使通过程序调用`@Bean`方法引用其他Bean时也是如此。相反，`@Bean`在普通区域内调用方法或方法中的字段`@Component` 类具有标准的Java语义，没有特殊的CGLIB处理或其他限制。

> 您可以将`@Bean`方法声明为`static`，从而允许在不将其包含配置类创建为实例的情况下调用它们。在定义后处理器Bean（例如类型`BeanFactoryPostProcessor` 或`BeanPostProcessor`）时，这特别有意义，因为此类Bean在容器生命周期的早期进行了初始化，并且应避免在此时触发配置的其他部分。由于技术限制，对静态`@Bean`方法的调用永远不会被容器拦截，即使在`@Configuration`类内也不会 （如本节前面所述），因为技术限制：CGLIB子类只能覆盖非静态方法。结果，直接调用另一个`@Bean`方法具有标准的Java语义，从而导致直接从工厂方法本身返回一个独立的实例。方法的Java语言可见性`@Bean`不会对Spring容器中的结果bean定义产生直接影响。您可以随意声明自己的工厂方法（如果您认为适合非`@Configuration`类），也可以在任何地方声明静态方法。但是，类中的常规`@Bean`方法`@Configuration`必须是可重写的-也就是说，不得将其声明为`private`或`final`。`@Bean`还可以在给定组件或配置类的基类上以及在由组件或配置类实现的接口中声明的Java 8默认方法上发现方法。这为组合复杂的配置安排提供了很大的灵活性，从Spring 4.2开始，通过Java 8默认方法甚至可以进行多重继承。最后，单个类可以`@Bean`为同一个bean 保留多个方法，这取决于在运行时可用的依赖项，以安排使用多个工厂方法。这与在其他配置方案中选择“最贪婪”的构造函数或工厂方法的算法相同：在构造时选择具有最大可满足依赖关系数量的变量，类似于容器在多个`@Autowired`构造函数之间进行选择的方式。

#### 1.10.6 命名自动检测的组件

当组件被自动检测为扫描过程的一部分时，其bean名称由该`BeanNameGenerator`扫描器已知的策略生成。默认情况下，任何Spring刻板印象注释（`@Component`，`@Repository`，`@Service`和 `@Controller`），其中包含一个名称`value`，从而提供了名称，相应的bean定义。

如果这样的注释不包含名称，`value`或者不包含任何其他检测到的组件（例如，由自定义过滤器发现的组件），则缺省bean名称生成器将返回不使用大写字母的非限定类名称。例如，如果检测到以下组件类，则名称为`myMovieLister`和`movieFinderImpl`：

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```



```java
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

如果不想依赖默认的Bean命名策略，则可以提供自定义Bean命名策略。首先，实现 [`BeanNameGenerator`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/support/BeanNameGenerator.html) 接口，并确保包括默认的无参数构造函数。然后，在配置扫描程序时提供完全限定的类名，如以下示例注释和Bean定义所示。

> 如果由于多个自动检测到的组件具有相同的非限定类名而遇到命名冲突（即，具有相同名称但位于不同包中的类），则可能需要配置一个`BeanNameGenerator`默认生成的完全限定类名豆名称。从Spring Framework 5.2.3开始， `FullyQualifiedAnnotationBeanNameGenerator`位于package中的包 `org.springframework.context.annotation`可用于此类目的。



```java
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    // ...
}
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

作为一般规则，每当其他组件可能对其进行显式引用时，请考虑使用注释指定名称。另一方面，只要容器负责接线，自动生成的名称就足够了。

#### 1.10.7 提供自动检测组件的范围

一般而言，与Spring管理的组件一样，自动检测到的组件的默认范围也是最常见的范围是`singleton`。但是，有时您需要`@Scope`注释可以指定的其他范围。您可以在批注中提供范围的名称，如以下示例所示：

爪哇

科特林

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

> `@Scope`注释仅在具体的bean类（对于带注释的组件）或工厂方法（对于`@Bean`方法）上进行内省。与XML bean定义相反，没有bean定义继承的概念，并且在类级别的继承层次结构与元数据目的无关。

有关特定于Web的范围的详细信息，例如Spring上下文中的“ request”或“ session”，请参阅[Request，Session，Application和WebSocket Scope](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other)。与这些作用域的预构建批注一样，您也可以使用Spring的元注释方法来编写自己的作用域注释：例如，用`@Scope("prototype")`，注释自定义注释的自定义注释，也可能会声明自定义作用域代理模式。

> 要提供用于范围解析的自定义策略，而不是依赖于基于注释的方法，可以实现该 [`ScopeMetadataResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/ScopeMetadataResolver.html) 接口。确保包括默认的无参数构造函数。然后，可以在配置扫描器时提供完全限定的类名，如以下注释和Bean定义示例所示：

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    // ...
}
<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```

使用某些非单作用域时，可能有必要为作用域对象生成代理。在[范围Bean中将](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)推理描述[为依赖项](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)。为此，在component-scan元素上可以使用scoped-proxy属性。三个可能的值是：`no`，`interfaces`，和`targetClass`。例如，以下配置生成标准的JDK动态代理：

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    // ...
}
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

#### 1.10.8 提供带有注释的限定符元数据

在`@Qualifier`注释中讨论[，基于注解微调自动装配与预选赛](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers)。该部分中的示例演示了如何使用`@Qualifier`注释和自定义限定符注释在解析自动装配候选时提供细粒度的控制。由于这些示例基于XML Bean定义，因此通过使用XML 中的元素的`qualifier`或`meta`子元素，在候选Bean定义上提供了限定符元数据`bean`。当依靠类路径扫描来自动检测组件时，可以在候选类上为限定符元数据提供类型级别的注释。下面的三个示例演示了此技术：

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

> 与大多数基于注释的替代方法一样，请记住，注释元数据绑定到类定义本身，而XML的使用允许相同类型的多个bean提供其限定符元数据的变体，因为该元数据是按-instance而不是按类。

#### 1.10.9 生成候选组件的索引

尽管类路径扫描非常快，但是可以通过在编译时创建候选静态列表来提高大型应用程序的启动性能。在这种模式下，作为组件扫描目标的所有模块都必须使用此机制。

> 您现有的`@ComponentScan`或`指令必须保持原样，以请求上下文扫描某些软件包中的候选对象。当 `ApplicationContext`检测到这样的索引时，它将自动使用它而不是扫描类路径。

要生成索引，请向每个包含组件的模块添加附加依赖关系，这些组件是组件扫描指令的目标。以下示例显示了如何使用Maven进行操作：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.2.6.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

对于Gradle 4.5及更早版本，应在`compileOnly` 配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    compileOnly "org.springframework:spring-context-indexer:5.2.6.RELEASE"
}
```

对于Gradle 4.6和更高版本，应在`annotationProcessor` 配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    annotationProcessor "org.springframework:spring-context-indexer:{spring-version}"
}
```

该过程将生成一个`META-INF/spring.components`包含在jar文件中的文件。

> 在IDE中使用此模式时，`spring-context-indexer`必须将其注册为注释处理器，以确保在更新候选组件时索引是最新的。

> 当`META-INF/spring.components`在类路径上找到a时，索引将自动启用。如果某个索引对于某些库（或用例）部分可用，但无法为整个应用程序构建，则可以通过将设置`spring.index.ignore`为 `true`，来回退到常规的类路径安排（好像根本没有索引）属性或`spring.properties`类路径根目录下的文件中。

