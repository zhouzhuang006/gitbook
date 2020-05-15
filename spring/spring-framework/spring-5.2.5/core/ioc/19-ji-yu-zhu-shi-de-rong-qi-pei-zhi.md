### 1.9 基于注释的容器配置 {#beans-annotation-config}

在配置Spring时，注释是否比XML更好？

基于注释的配置的引入提出了一个问题，即这种方法是否比XML“更好”。简短的答案是“取决于情况”。长的答案是每种方法都有其优缺点，通常，由开发人员决定哪种策略更适合他们。由于定义方式的不同，注释在声明中提供了很多上下文，从而使配置更短，更简洁。但是，XML擅长连接组件而不接触其源代码或重新编译它们。一些开发人员更喜欢将布线放置在靠近源的位置，而另一些开发人员则认为带注释的类不再是POJO，而且，该配置变得分散且难以控制。

无论选择如何，Spring都可以容纳两种样式，甚至可以将它们混合在一起。值得指出的是，通过其[JavaConfig](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java)选项，Spring允许以非侵入方式使用批注，而无需接触目标组件的源代码，并且就工具而言，[Spring Tools for Eclipse](https://spring.io/tools)支持所有配置样式 。

基于注释的配置提供了XML设置的替代方法，该配置依赖字节码元数据来连接组件，而不是尖括号声明。通过使用相关类，方法或字段声明上的注释，开发人员无需使用XML来描述bean的连接，而是将配置移入组件类本身。如[示例中所述：将`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp-examples-rabpp)，`BeanPostProcessor`与注释结合使用是扩展Spring IoC容器的常用方法。例如，Spring 2.0引入了通过[`@Required`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-required-annotation)注释强制执行必需属性的可能性。Spring 2.5使遵循相同的通用方法来驱动Spring的依赖注入成为可能。本质上，`@Autowired`注解提供的功能与[自动装配协作器中](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-autowire)所述的功能相同，但具有更细粒度的控制和更广泛的适用性。Spring 2.5还添加了对JSR-250批注（例如 `@PostConstruct`和）的支持`@PreDestroy`。弹簧3.0 JSR-330（Java依赖注入）加入支持注释包含在`javax.inject`包如`@Inject` 和`@Named`。有关这些注释的详细信息，请参见 [相关章节](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-standard-annotations)。

> 注释注入在XML注入之前执行。因此，XML配置将覆盖通过两种方法连接的属性的注释。

与往常一样，您可以将它们注册为单独的bean定义，但也可以通过在基于XML的Spring配置中包括以下标记来隐式注册它们（注意，包括`context`名称空间）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

（隐式注册的后处理器包括 [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)， [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html)和 [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html)，以及上述 [`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html)。）

> ``只在定义它的相同应用程序上下文中查找关于bean的注释。这意味着，如果 ``在中输入`WebApplicationContext`for `DispatcherServlet`，则仅检查`@Autowired`控制器中的bean，而不检查服务中的bean。有关更多信息，请参见 [DispatcherServlet](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/web.html#mvc-servlet)。

#### 1.9.1 @Required

该`@Required`注释适用于bean属性setter方法，如下面的例子：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

此注释指示必须在配置时通过bean定义中的显式属性值或通过自动装配来填充受影响的bean属性。如果尚未填充受影响的bean属性，则容器将引发异常。这允许急切和显式的故障，避免`NullPointerException` 以后再发生实例等。我们仍然建议您将断言放入bean类本身中（例如，放入init方法中）。这样做会强制执行那些必需的引用和值，即使您在容器外部使用该类也是如此。

> 从`@Required`Spring Framework 5.1开始，正式弃用了该批注，以支持对所需的设置（或`InitializingBean.afterPropertiesSet()`Bean属性setter方法的自定义实现）使用构造函数注入 。

#### 1.9.2 使用`@Autowired`

> 在本节中的示例中，`@Inject`可以使用JSR 330的注释代替Spring的`@Autowired`注释。有关更多详细信息，请参见[此处](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-standard-annotations)。

您可以将`@Autowired`注释应用于构造函数，如以下示例所示：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

> 从Spring Framework 4.3开始，`@Autowired`如果目标bean仅定义一个构造函数作为开始，则不再需要在此类构造函数上添加注释。但是，如果有几个构造函数可用，并且没有主/默认构造函数，则必须至少注释一个构造函数，`@Autowired`以指示容器使用哪个构造函数。有关详细信息，请参见有关[构造函数解析](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation-constructor-resolution)的讨论 。

您还可以将`@Autowired`注释应用于*传统的* setter方法，如以下示例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

您还可以将注释应用于具有任意名称和多个参数的方法，如以下示例所示：

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

您还可以将其应用于`@Autowired`字段，甚至将其与构造函数混合使用，如以下示例所示：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

> 确保目标组件（例如`MovieCatalog`或`CustomerPreferenceDao`）由用于带`@Autowired`注释的注入点的类型一致地声明。否则，注入可能会由于运行时出现“找不到类型匹配”错误而失败。对于通过类路径扫描找到的XML定义的bean或组件类，容器通常预先知道具体的类型。但是，对于`@Bean`工厂方法，您需要确保声明的返回类型具有足够的表现力。对于实现多个接口的组件或可能由其实现类型引用的组件，请考虑在工厂方法中声明最具体的返回类型（至少根据引用您的bean的注入点的要求具体声明）。

您还可以`ApplicationContext`通过将`@Autowired`注释添加到需要该类型数组的字段或方法中，指示Spring提供特定类型的所有bean ，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

如下例所示，这同样适用于类型化集合：

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

> 如果要使数组或列表中的项目以特定顺序排序，则目标bean可以实现`org.springframework.core.Ordered`接口或使用`@Order`或标准`@Priority`注释。否则，它们的顺序将遵循容器中相应目标bean定义的注册顺序。您可以`@Order`在目标类级别和`@Bean`方法上声明注释，可能是针对单个bean定义（在使用同一bean类的多个定义的情况下）。`@Order`值可能会影响注入点的优先级，但请注意它们不会影响单例启动顺序，这是由依赖关系和`@DependsOn`声明确定的正交关注点。请注意，标准`javax.annotation.Priority`注释在该`@Bean`级别不可用 ，因为无法在方法上声明它。可以通过将每种类型的`@Order`值与`@Primary`单个bean 结合使用来对其语义进行建模。

`Map`只要预期的密钥类型为，即使是键入的实例也可以自动装配`String`。映射值包含所有预期类型的bean，并且键包含相应的bean名称，如以下示例所示：

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认情况下，当给定注入点没有匹配的候选bean可用时，自动装配将失败。对于声明的数组，集合或映射，至少应有一个匹配元素。

默认行为是将带注释的方法和字段视为指示所需的依赖项。您可以更改此行为，如以下示例所示，使框架可以通过将其标记为不需要来跳过不满意的注入点（即，将`required`属性设置`@Autowired`为`false`）：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

如果不需要的方法（或在多个参数的情况下，其中一个依赖项）不可用，则根本不会调用该方法。在这种情况下，完全不需要填充非必需字段，而将其默认值保留在适当的位置。

注入的构造函数和工厂方法参数是一种特殊情况，因为由于Spring的构造函数解析算法可能会处理多个构造函数，所以`required` in `@Autowired`中的属性含义有所不同。默认情况下，有效地需要构造函数和工厂方法参数，但是在单构造函数场景中有一些特殊规则，例如，如果没有可用的匹配bean，则多元素注入点（数组，集合，映射）解析为空实例。这允许一种通用的实现模式，其中所有依赖项都可以在唯一的多参数构造函数中声明，例如，声明为不带`@Autowired`注释的单个公共构造函数。

> 只有任何给定的bean类的一个构造可以宣告`@Autowired`与`required` 属性设置为`true`，表明*该*构造函数自动装配作为一个Spring bean使用时。结果，如果将`required`属性保留为其默认值`true`，则只能使用注释单个构造函数`@Autowired`。如果多个构造函数声明注解，则都必须声明它们`required=false`才能被视为自动装配的候选对象（类似于`autowire=constructor`XML）。将选择通过匹配Spring容器中的bean可以满足的依赖关系数量最多的构造函数。如果没有一个候选者满意，则将使用主/默认构造函数（如果存在）。类似地，如果一个类声明了多个构造函数，但都没有用注释`@Autowired`，则将使用主/默认构造函数（如果存在）。如果一个类仅声明一个单一的构造函数开始，即使没有注释，也将始终使用它。请注意，带注释的构造函数不必是公共的。建议 在setter方法的不建议使用的批注上使用的`required`属性。将属性设置为表示该属性对于自动装配不是必需的，并且如果不能自动装配该属性，则将忽略该属性。另一方面，它更强大，因为它强制通过容器支持的任何方式来设置属性，并且如果未定义任何值，则会引发相应的异常。`@Autowired``@Required``required``false``@Required`

另外，您可以通过Java 8来表达特定依赖项的非必需性质`java.util.Optional`，如以下示例所示：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

从Spring Framework 5.0开始，您还可以使用`@Nullable`注释（任何包中的任何注释，例如，`javax.annotation.Nullable`来自JSR-305 的注释），或仅利用Kotlin内置的null安全支持：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

您还可以使用`@Autowired`对于那些众所周知的解析依赖接口：`BeanFactory`，`ApplicationContext`，`Environment`，`ResourceLoader`， `ApplicationEventPublisher`，和`MessageSource`。这些接口及其扩展接口（例如`ConfigurableApplicationContext`或`ResourcePatternResolver`）将自动解析，而无需进行特殊设置。以下示例自动装配`ApplicationContext`对象：

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

> 在`@Autowired`，`@Inject`，`@Value`，和`@Resource`注释由Spring处理 `BeanPostProcessor`实现。这意味着您不能在自己的类型`BeanPostProcessor`或`BeanFactoryPostProcessor`类型（如果有）中应用这些注释。这些类型必须使用XML或Spring `@Bean`方法显式地“连接” 。

#### 1.9.3 通过微调基于注释的自动装配`@Primary`

由于按类型自动布线可能会导致多个候选对象，因此通常有必要对选择过程进行更多控制。实现此目的的一种方法是使用Spring的 `@Primary`注释。`@Primary`指示当多个bean是要自动装配到单值依赖项的候选对象时，应给予特定bean优先权。如果候选中恰好存在一个主bean，它将成为自动装配的值。

考虑以下定义`firstMovieCatalog`为主要配置的配置`MovieCatalog`：

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

使用前面的配置，以下内容`MovieRecommender`将自动连接到 `firstMovieCatalog`：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

相应的bean定义如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

#### 1.9.4 使用限定符对基于注释的自动装配进行微调

`@Primary`当可以确定一个主要候选对象时，它是在几种情况下按类型使用自动装配的有效方法。当您需要更好地控制选择过程时，可以使用Spring的`@Qualifier`注释。您可以将限定符值与特定的参数相关联，从而缩小类型匹配的范围，以便为每个参数选择特定的bean。在最简单的情况下，这可以是简单的描述性值，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

您还可以`@Qualifier`在各个构造函数参数或方法参数上指定注释，如以下示例所示：

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

以下示例显示了相应的bean定义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

> 具有`main`限定符值的Bean与限定有相同值的构造函数参数连接。
>
> 具有`action`限定符值的Bean与限定有相同值的构造函数参数连接。

对于后备匹配，bean名称被视为默认的限定符值。因此，可以使用`id`of `main`而不是嵌套的限定符元素来定义bean ，从而得到相同的匹配结果。但是，尽管可以使用此约定按名称引用特定的bean，但从`@Autowired`根本上讲，它是带有可选语义限定符的类型驱动的注入。这意味着限定符值，即使具有bean名称回退，也总是在类型匹配集中具有狭窄的语义。它们没有在语义上表示对唯一bean的引用`id`。好的限定符值是`main` 或`EMEA`或`persistent`，表示独立于Bean的特定组件的特征`id`，如果是匿名Bean定义（例如上例中的定义），则可以自动生成。

限定符还适用于类型化的集合，如前所述（例如，应用于） `Set`。在这种情况下，根据声明的限定词，所有匹配的bean都作为一个集合注入。这意味着限定词不必是唯一的。相反，它们构成了过滤标准。例如，您可以`MovieCatalog`使用相同的限定符值“ action” 来定义多个bean，所有这些都将注入到带有的`Set`注释中`@Qualifier("action")`。

> 在类型匹配的候选对象中，让限定符值针对目标bean名称进行选择，在`@Qualifier`注入点不需要注释。如果没有其他解析度指示符（例如限定词或主标记），则对于非唯一依赖性情况，Spring将注入点名称（即字段名称或参数名称）与目标Bean名称进行匹配，然后选择同名候选人（如果有）。

就是说，如果您打算按名称表示注释驱动的注入，则不要主要使用`@Autowired`，即使它能够在类型匹配的候选对象中按bean名称进行选择。而是使用JSR-250 `@Resource`批注，该批注的语义定义是通过其唯一名称来标识特定目标组件，而声明的类型与匹配过程无关。`@Autowired`具有不同的语义：按类型选择候选bean之后，`String` 仅在那些类型选择的候选对象中考虑指定的限定符值（例如，将`account`限定符与标记有相同限定符标签的bean进行匹配）。

对于本身定义为collection `Map`或array类型的`@Resource` bean是一个很好的解决方案，可以通过唯一的名称引用特定的collection或array bean。也就是说，从4.3版本开始，只要元素类型信息保留在返回类型签名或集合继承层次结构中，就可以`Map`通过Spring的`@Autowired`类型匹配算法来匹配和数组类型 `@Bean`。在这种情况下，您可以使用限定符值在同类型的集合中进行选择，如上一段所述。

从4.3开始，`@Autowired`还考虑了自我注入的引用（即，对当前注入的Bean的引用）。请注意，自我注入是一个后备。对其他组件的常规依赖始终优先。从这个意义上说，自我推荐不参与常规的候选人选择，因此尤其是绝不是主要的。相反，它们总是以最低优先级结束。实际上，您应该仅将自引用用作最后的手段（例如，通过bean的事务代理在同一实例上调用其他方法）。考虑在这种情况下将受影响的方法分解为单独的委托bean。或者，您可以使用`@Resource`，它可以通过其唯一名称获取返回到当前bean的代理。

> 尝试从`@Bean`相同配置类上的方法中注入结果也是有效的自引用方案。要么在实际需要的方法签名中延迟解析这些引用（与配置类中的自动装配字段相对），要么将受影响的`@Bean`方法声明为`static`，将它们与包含的配置类实例及其生命周期脱钩。否则，仅在回退阶段考虑此类Bean，而将其他配置类上的匹配Bean选作主要候选对象（如果可用）。

`@Autowired`适用于字段，构造函数和多参数方法，从而允许在参数级别缩小限定符注释的范围。相反，`@Resource` 仅支持具有单个参数的字段和bean属性设置器方法。因此，如果注入目标是构造函数或多参数方法，则应坚持使用限定符。

您可以创建自己的自定义限定符注释。为此，请定义一个注释并`@Qualifier`在您的定义中提供该注释，如以下示例所示：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

然后，您可以在自动连接的字段和参数上提供自定义限定符，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

接下来，您可以提供有关候选bean定义的信息。您可以将``标签添加为 标签的子元素，``然后指定`type`和 `value`以匹配您的自定义限定符注释。该类型与注释的完全限定的类名匹配。另外，为方便起见，如果不存在名称冲突的风险，则可以使用简短的类名。下面的示例演示了两种方法：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在“ [类路径扫描和托管组件”中](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-classpath-scanning)，您可以看到基于注释的替代方法，以XML提供限定符元数据。具体来说，请参阅[为Qualifier元数据提供注释](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-scanning-qualifiers)。

在某些情况下，使用没有值的注释就足够了。当注释具有更一般的用途并且可以应用于几种不同类型的依赖项时，这将很有用。例如，您可以提供一个脱机目录，当没有Internet连接可用时，可以对其进行搜索。首先，定义简单注释，如以下示例所示：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```

然后将注释添加到要自动装配的字段或属性，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @Offline 
    private MovieCatalog offlineCatalog;

    // ...
}
```

> 这行添加`@Offline`注释。

现在，bean定义只需要一个限定符`type`，如以下示例所示：

```xml
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/> 
    <!-- inject any dependencies required by this bean -->
</bean>
```

> 该元素指定限定词。

您还可以定义自定义限定符批注，该批注除了简单`value`属性之外或代替简单属性，还接受命名属性。如果随后在要自动装配的字段或参数上指定了多个属性值，则Bean定义必须与所有此类属性值匹配才能被视为自动装配候选。例如，请考虑以下注释定义：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

在这种情况下`Format`是一个枚举，定义如下：

```java
public enum Format {
    VHS, DVD, BLURAY
}
```

要自动装配的字段将用定制限定符进行注释，并包括这两个属性的值：`genre`和`format`，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

最后，bean定义应包含匹配的限定符值。此示例还演示了可以使用bean元属性代替 ``元素。如果可用，则``元素及其属性优先，但是``如果不存在此类限定符，则自动装配机制将根据标签内提供的值退回 ，如以下示例中的最后两个bean定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

#### 1.9.5 将泛型用作自动装配限定符

除了`@Qualifier`注释之外，您还可以将Java泛型类型用作资格的隐式形式。例如，假设您具有以下配置：

```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

假设前面的bean实现了一个通用接口（即`Store`和 `Store`），则可以`@Autowire`将该`Store`接口和通用用作限定符，如以下示例所示：

```java
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

当自动装配列表，`Map`实例和数组时，通用限定符也适用。下面的示例自动连接泛型`List`：

```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

#### 1.9.6 使用`CustomAutowireConfigurer`

[`CustomAutowireConfigurer`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/CustomAutowireConfigurer.html) 是一个`BeanFactoryPostProcessor`，即使您没有使用Spring的`@Qualifier`注释来注释自己的自定义限定符注释类型，也可以使用它。以下示例显示如何使用`CustomAutowireConfigurer`：

```xml
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

通过以下方式`AutowireCandidateResolver`确定自动装配候选对象：

- `autowire-candidate`每个bean定义的值
- 元素`default-autowire-candidates`上可用的任何模式``
- `@Qualifier`注释的存在以及向NET注册的任何自定义注释`CustomAutowireConfigurer`

当多个bean符合自动装配候选条件时，确定“主要”的步骤如下：如果候选中恰好有一个bean定义将`primary` 属性设置为`true`，则将其选中。

#### 1.9.7 注射用`@Resource`

Spring还通过在字段或bean属性设置器方法上使用JSR-250 `@Resource`批注（`javax.annotation.Resource`）支持注入。这是Java EE中的一种常见模式：例如，在JSF管理的Bean和JAX-WS端点中。Spring也为Spring管理的对象支持此模式。

`@Resource`具有名称属性。默认情况下，Spring将该值解释为要注入的Bean名称。换句话说，它遵循名称语义，如以下示例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

> 这行注入`@Resource`。

如果未明确指定名称，则默认名称是从字段名称或setter方法派生的。如果是字段，则采用字段名称。在使用setter方法的情况下，它采用bean属性名称。以下示例将名为bean `movieFinder`的setter方法注入：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

> 提供注解的名称解析由一个bean的名称 `ApplicationContext`，其中的`CommonAnnotationBeanPostProcessor`知道。如果您[`SimpleJndiBeanFactory`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jndi/support/SimpleJndiBeanFactory.html) 显式配置Spring的名称，则可以通过JNDI解析名称 。但是，我们建议您依赖默认行为并使用Spring的JNDI查找功能来保留间接级别。

在专属情况下，`@Resource`不指定明确的名称，以及类似的使用`@Autowired`，`@Resource`发现的主要类型匹配，而不是一个具体的bean并解决众所周知的解析依存关系：`BeanFactory`， `ApplicationContext`，`ResourceLoader`，`ApplicationEventPublisher`，和`MessageSource` 接口。

因此，在以下示例中，该`customerPreferenceDao`字段首先查找名为“ customerPreferenceDao”的bean，然后回退到该类型的主类型匹配项 `CustomerPreferenceDao`：

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

> 该`context`字段是根据已知的可解析依赖类型注入的 `ApplicationContext`。

#### 1.9.8 使用`@Value`

`@Value` 通常用于注入外部属性：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

使用以下配置：

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

和以下`application.properties`文件：

```java
catalog.name=MovieCatalog
```

在这种情况下，`catalog`参数和字段将等于`MovieCatalog`值。

Spring提供了一个默认的宽松内嵌值解析器。它将尝试解析属性值，如果无法解析，`${catalog.name}`则将注入属性名称（例如）作为值。如果要严格控制不存在的值，则应声明一个`PropertySourcesPlaceholderConfigurer`bean，如以下示例所示：

```java
@Configuration
public class AppConfig {

     @Bean
     public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
           return new PropertySourcesPlaceholderConfigurer();
     }
}
```

> 当配置`PropertySourcesPlaceholderConfigurer`使用JavaConfig，该 `@Bean`方法必须是`static`。

如果`${}` 无法解析任何占位符，则使用上述配置可确保Spring初始化失败。也可以使用`setPlaceholderPrefix`，，之类的方法 `setPlaceholderSuffix`或`setValueSeparator`自定义占位符。

> 默认情况下，Spring Boot配置一个`PropertySourcesPlaceholderConfigurer`将从`application.properties`和获取属性的bean `application.yml`。

Spring提供的内置转换器支持允许自动处理简单的类型转换（例如转换为`Integer` 或`int`）。多个逗号分隔的值可以自动转换为String数组，而无需付出额外的努力。

可以提供如下默认值：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
        this.catalog = catalog;
    }
}
```

Spring `BeanPostProcessor`使用`ConversionService`幕后处理将String值转换`@Value`为目标类型的过程。如果要为自己的自定义类型提供转换支持，则可以提供自己的 `ConversionService`bean实例，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public ConversionService conversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        conversionService.addConverter(new MyCustomConverter());
        return conversionService;
    }
}
```

当`@Value`包含[`SpEL`表达式时，](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#expressions)该值将在运行时动态计算，如以下示例所示：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {
        this.catalog = catalog;
    }
}
```

SpEL还支持使用更复杂的数据结构：

```java
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(
            @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

#### 1.9.9 使用`@PostConstruct`和`@PreDestroy`

将`CommonAnnotationBeanPostProcessor`不仅承认了`@Resource`注解也是JSR-250的生命周期注解：`javax.annotation.PostConstruct`和 `javax.annotation.PreDestroy`。在Spring 2.5中引入了对这些注释的支持，为[初始化回调](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)和 [销毁回调中](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)描述的生命周期回调机制提供了一种替代 [方法](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)。假设 `CommonAnnotationBeanPostProcessor`在Spring内注册`ApplicationContext`，则在生命周期的同一点将调用带有这些注释之一的方法作为相应的Spring生命周期接口方法或显式声明的回调方法。在以下示例中，缓存在初始化时预先填充，并在销毁时清除：

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

有关组合各种生命周期机制的效果的详细信息，请参见 [组合生命周期机制](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-combined-effects)。

> 像和一样`@Resource`，`@PostConstruct`和`@PreDestroy`注释类型是JDK 6到8的标准Java库的一部分。但是，整个`javax.annotation` 包都与JDK 9中的核心Java模块分开，并最终在JDK 11中删除了。如果需要，需要对`javax.annotation-api`工件进行处理。现在可以通过Maven Central获得，只需像其他任何库一样将其添加到应用程序的类路径中即可。

