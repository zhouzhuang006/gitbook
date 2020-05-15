### 1.13 环境抽象 {#beans-environment}

该[`Environment`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/Environment.html)接口是集成在容器中的抽象，可以对应用程序环境的两个关键方面进行建模：[profile](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-definition-profiles) 和[properties](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-property-source-abstraction)。

概要文件是仅在给定概要文件处于活动状态时才向容器注册的Bean定义的命名逻辑组。可以将Bean分配给概要文件，无论是以XML定义还是带有注释。`Environment`对象与配置文件相关的作用是确定当前哪些配置文件（如果有）处于活动状态，以及默认情况下哪些配置文件（如果有）应处于活动状态。

属性在几乎所有应用程序中都扮演着重要的角色，并且可能源自各种来源：属性文件，JVM系统属性，系统环境变量，JNDI，Servlet上下文参数，即席`Properties`对象，`Map`对象等。`Environment`对象与属性有关的角色是为用户提供方便的服务界面，用于配置属性源并从中解析属性。

#### 1.13.1。Bean定义配置文件

Bean定义配置文件在核心容器中提供了一种机制，该机制允许在不同环境中注册不同的Bean。“环境”一词对不同的用户可能具有不同的含义，并且此功能可以帮助解决许多用例，包括：

- 在开发中针对内存中的数据源进行工作，而不是在进行QA或生产时从JNDI查找相同的数据源。
- 仅在将应用程序部署到性能环境中时注册监视基础结构。
- 为客户A和客户B部署注册bean的自定义实现。

考虑实际应用中第一个用例的需求 `DataSource`。在测试环境中，配置可能类似于以下内容：

爪哇

科特林

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在，假设该应用程序的数据源已在生产应用程序服务器的JNDI目录中注册，请考虑如何将该应用程序部署到QA或生产环境中。`dataSource`现在，我们的bean看起来像下面的清单：

爪哇

科特林

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是如何根据当前环境在使用这两种变体之间进行切换。随着时间的流逝，Spring用户已经设计出许多方法来完成此任务，通常依赖于系统环境变量和XML ``语句的组合，XML 语句包含`${placeholder}`根据环境变量的值解析为正确的配置文件路径的令牌。Bean定义配置文件是一个核心容器功能，可提供此问题的解决方案。

如果我们概括前面特定于环境的Bean定义示例中所示的用例，那么最终需要在某些上下文中而不是在其他上下文中注册某些Bean定义。您可能会说您要在情况A中注册一个特定的bean定义配置文件，在情况B中注册一个不同的配置文件。我们首先更新配置以反映这种需求。

##### 使用 `@Profile`

该[`@Profile`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Profile.html) 注解让你指示组件有资格登记在一个或多个指定的配置文件是活动的。使用前面的示例，我们可以`dataSource`如下重写配置：

爪哇

科特林

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

爪哇

科特林

```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

|      | 如前所述，使用`@Bean`方法时，通常选择使用程序化JNDI查找，方法是使用Spring的`JndiTemplate`/ `JndiLocatorDelegate`helpers或`InitialContext`前面显示的直接JNDI 用法，而不使用`JndiObjectFactoryBean` 变体，这将迫使您将返回类型声明为该`FactoryBean`类型。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

配置文件字符串可以包含简单的配置文件名称（例如`production`）或配置文件表达式。配置文件表达式允许表达更复杂的配置文件逻辑（例如`production & us-east`）。概要文件表达式中支持以下运算符：

- `!`：配置文件的逻辑“不”
- `&`：配置文件的逻辑“与”
- `|`：配置文件的逻辑“或”

|      | 您不能在不使用括号的情况下混合使用`&`and `|`运算符。例如， `production & us-east | eu-central`不是有效的表达式。它必须表示为 `production & (us-east | eu-central)`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您可以将其`@Profile`用作[元注释](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-meta-annotations)，以创建自定义的组合注释。以下示例定义了一个自定义 `@Production`批注，您可以将其用作替代品 `@Profile("production")`：

爪哇

科特林

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

|      | 如果用`@Configuration`标记了一个类，则除非一个或多个指定的配置文件处于活动状态，否则将忽略与该类关联的`@Profile`所有`@Bean`方法和 `@Import`注释。如果使用标记了`@Component`或`@Configuration`类`@Profile({"p1", "p2"})`，除非激活了配置文件“ p1”或“ p2”，否则该类不会注册或处理。如果给定的配置文件以NOT运算符（`!`）为前缀，则仅在该配置文件不活动时才注册带注释的元素。例如，给定`@Profile({"p1", "!p2"})`，如果配置文件“ p1”处于活动状态或配置文件“ p2”未处于活动状态，则会进行注册。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`@Profile` 也可以在方法级别声明为仅包含配置类的一个特定bean（例如，特定bean的替代变体），如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

|      | 该`standaloneDataSource`方法仅在`development`配置文件中可用。 |
| ---- | ------------------------------------------------------------ |
|      | 该`jndiDataSource`方法仅在`production`配置文件中可用。       |

|      | 使用`@Profile`on `@Bean`方法时，可能会出现特殊情况：对于具有`@Bean`相同Java方法名称的重载方法（类似于构造函数重载），`@Profile`必须在所有重载方法上一致声明条件。如果条件不一致，则仅重载方法中第一个声明的条件很重要。因此，`@Profile`不能用于选择具有特定自变量签名的重载方法。在创建时，同一bean的所有工厂方法之间的解析都遵循Spring的构造函数解析算法。如果要定义具有不同概要文件条件的备用Bean，请通过使用`@Bean`name属性使用指向相同Bean名称的不同Java方法名称，如前面的示例所示。如果参数签名都相同（例如，所有变体都具有no-arg工厂方法），则这是首先在有效Java类中表示这种排列的唯一方法（因为只能有一个特定名称和参数签名的方法）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### XML Bean定义配置文件

XML对应项是元素的`profile`属性``。我们前面的示例配置可以重写为两个XML文件，如下所示：

```xml
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

也可以避免``在同一文件中拆分和嵌套元素，如以下示例所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

在`spring-bean.xsd`受到了制约，使这些元素只能作为文件中的最后一个人。这应该有助于提供灵活性，而不会引起XML文件混乱。

|      | XML对应项不支持前面描述的配置文件表达式。但是，可以通过使用`!`运算符来取消配置文件。也可以通过嵌套配置文件来应用逻辑“和”，如以下示例所示：`                                           `在前面的示例中，`dataSource`如果`production`和 `us-east`配置文件都处于活动状态，则该bean被公开。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 激活个人资料

现在我们已经更新了配置，我们仍然需要指示Spring哪个配置文件处于活动状态。如果我们现在启动示例应用程序，则会看到`NoSuchBeanDefinitionException`抛出的错误，因为容器找不到名为的Spring bean `dataSource`。

可以通过多种方式来激活配置文件，但是最直接的方法是针对`Environment`通过可以使用的API以 编程方式进行配置`ApplicationContext`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

此外，您还可以通过`spring.profiles.active`属性声明性地激活配置文件，可以通过系统环境变量，JVM系统属性，中的servlet上下文参数`web.xml`或什至可以作为JNDI中的条目来指定概要文件 （请参见[`PropertySource`Abstraction](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-property-source-abstraction)）。在集成测试中，可以通过使用 模块中的`@ActiveProfiles`注释来声明活动配置文件`spring-test`（请参阅[环境配置文件的上下文配置](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/testing.html#testcontext-ctx-management-env-profiles)）。

请注意，配置文件不是“非此即彼”的命题。您可以一次激活多个配置文件。通过编程，您可以为该`setActiveProfiles()`方法提供多个配置文件名称，该名称 接受`String…`varargs。以下示例激活多个配置文件：

爪哇

科特林

```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

以声明的方式，`spring.profiles.active`可以接受以逗号分隔的配置文件名称列表，如以下示例所示：

```
    -Dspring.profiles.active =“ profile1，profile2”
```

##### 默认配置文件

默认配置文件表示默认情况下启用的配置文件。考虑以下示例：

爪哇

科特林

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有任何配置文件处于活动状态，则将`dataSource`创建。您可以看到这是为一个或多个bean提供默认定义的一种方法。如果启用了任何配置文件，则默认配置文件将不适用。

您可以通过更改默认的配置文件的名称`setDefaultProfiles()`上`Environment`，或者声明，通过使用`spring.profiles.default`属性。

#### 1.13.2。`PropertySource`抽象化

Spring的`Environment`抽象提供了可配置属性源层次结构上的搜索操作。考虑以下清单：

爪哇

科特林

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

在前面的代码片段中，我们看到了一种询问Spring是否`my-property`为当前环境定义了该属性的高级方法。为了回答这个问题，`Environment`对象在一组对象上执行搜索[`PropertySource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html) 。A `PropertySource`是对任何键-值对源的简单抽象，Spring的[`StandardEnvironment`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html) 配置有两个PropertySource对象-一个代表JVM系统属性集（`System.getProperties()`）和一个代表系统环境变量集（`System.getenv()`）。

|      | 这些默认属性源存在于中`StandardEnvironment`，供在独立应用程序中使用。[`StandardServletEnvironment`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/context/support/StandardServletEnvironment.html) 用其他默认属性源（包括servlet配置和servlet上下文参数）填充。它可以选择启用[`JndiPropertySource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jndi/JndiPropertySource.html)。有关详细信息，请参见javadoc。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

具体来说，当您使用时`StandardEnvironment`，`env.containsProperty("my-property")` 如果在运行时存在`my-property`系统属性或`my-property`环境变量，则对的调用将返回true 。

|      | 执行的搜索是分层的。默认情况下，系统属性优先于环境变量。因此，如果`my-property`在调用时在两个地方同时设置了`env.getProperty("my-property")`该属性，则系统属性值将“获胜”并返回。请注意，属性值不会合并，而是会被前面的条目完全覆盖。对于common `StandardServletEnvironment`，完整层次结构如下，最高优先级条目位于顶部：ServletConfig参数（如果适用，例如在`DispatcherServlet`上下文的情况下）ServletContext参数（web.xml上下文参数条目）JNDI环境变量（`java:comp/env/`条目）JVM系统属性（`-D`命令行参数）JVM系统环境（操作系统环境变量） |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

最重要的是，整个机制是可配置的。也许您具有要集成到此搜索中的自定义属性源。为此，请实现并实例化自己的实例`PropertySource`并将其添加到`PropertySources`current 的集合中`Environment`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

在前面的代码中，`MyPropertySource`已在搜索中添加了最高优先级。如果它包含一个`my-property`属性，则会检测到并返回该属性，从而支持`my-property`任何其他属性`PropertySource`。该 [`MutablePropertySources`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/MutablePropertySources.html) API公开了许多方法，这些方法可以精确地控制属性源集。

#### 1.13.3。使用`@PropertySource`

该[`@PropertySource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/PropertySource.html) 注解提供便利和声明的机制添加`PropertySource` 到Spring的`Environment`。

给定一个`app.properties`包含键-值对的名为的文件`testbean.name=myTestBean`，以下`@Configuration`类以`@PropertySource`一种调用`testBean.getName()`return 的方式使用`myTestBean`：

爪哇

科特林

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

资源位置中`${…}`存在的所有占位符`@PropertySource`都是根据已经针对环境注册的一组属性源来解析的，如以下示例所示：

爪哇

科特林

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

假设`my.placeholder`存在于已注册的属性源之一（例如，系统属性或环境变量）中，则占位符将解析为相应的值。如果不是，则`default/path`用作默认值。如果未指定默认值并且无法解析属性， `IllegalArgumentException`则抛出。

|      | 该`@PropertySource`注释是可重复的，根据Java的8约定。但是，所有此类`@PropertySource`批注都需要在同一级别上声明，可以直接在配置类上声明，也可以在同一自定义批注中声明为元批注。不建议将直接注释和元注释混合使用，因为直接注释会有效地覆盖元注释。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.13.4。声明中的占位符解析

从历史上看，元素中占位符的值只能根据JVM系统属性或环境变量来解析。这已不再是这种情况。由于`Environment`抽象是在整个容器中集成的，因此很容易通过它路由占位符的解析。这意味着您可以按照自己喜欢的任何方式配置解析过程。您可以更改搜索系统属性和环境变量的优先级，也可以完全删除它们。您还可以根据需要将自己的属性源添加到组合中。

具体而言，无论该`customer` 属性在何处定义，以下语句均有效，只要该属性在以下位置可用`Environment`：

```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```


