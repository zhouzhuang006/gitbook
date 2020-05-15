### 1.12 基于Java的容器配置 {#beans-java}

本节介绍如何在Java代码中使用注释来配置Spring容器。它包括以下主题：

- [基本概念：`@Bean`和`@Configuration`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts)
- [使用实例化Spring容器 `AnnotationConfigApplicationContext`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-instantiating-container)
- [使用`@Bean`注释](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-bean-annotation)
- [使用`@Configuration`注释](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-configuration-annotation)
- [组成基于Java的配置](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-composing-configuration-classes)
- [Bean定义配置文件](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-definition-profiles)
- [`PropertySource` 抽象化](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-property-source-abstraction)
- [使用 `@PropertySource`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-using-propertysource)
- [声明中的占位符解析](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-placeholder-resolution-in-statements)

#### 1.12.1 基本概念：`@Bean`和`@Configuration`

Spring的新Java配置支持中的主要工件是-带 `@Configuration`注释的类和-带`@Bean`注释的方法。

该`@Bean`注释被用于指示一个方法实例，可以配置，并初始化到由Spring IoC容器进行管理的新对象。对于那些熟悉Spring的``XML配置的人来说，`@Bean`注释的作用与``元素相同。您可以`@Bean`对任何Spring 使用带注释的方法 `@Component`。但是，它们最常与`@Configuration`bean一起使用。

用注释类`@Configuration`表示其主要目的是作为Bean定义的来源。此外，`@Configuration`类允许通过调用`@Bean`同一类中的其他方法来定义Bean之间的依赖关系。最简单的`@Configuration`类如下：

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

上一`AppConfig`类等效于以下Spring ``XML：

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

完整的@Configuration与“精简” @Bean模式？

如果`@Bean`在未使用注释的类中声明方法，则 `@Configuration`它们被称为以“精简”模式进行处理。在一个`@Component`或什至在一个普通的旧类中声明的Bean方法被认为是“精简版”，其包含类的主要目的不同，而`@Bean`方法在那儿是一种奖励。例如，服务组件可以通过`@Bean`每个适用组件类上的其他方法向容器公开管理视图。在这种情况下，`@Bean`方法是通用的工厂方法机制。

与full不同`@Configuration`，lite `@Bean`方法无法声明Bean间的依赖关系。相反，它们在其包含组件的内部状态上进行操作，并且还可以根据可能声明的自变量进行操作。`@Bean`因此，此类方法不应调用其他 `@Bean`方法。每个此类方法实际上只是针对特定bean引用的工厂方法，而没有任何特殊的运行时语义。这里的积极副作用是，不必在运行时应用CGLIB子类，因此在类设计方面没有任何限制（也就是说，包含类可以是`final`依此类推）。

在常见情况下，`@Bean`方法将在`@Configuration`类中声明，以确保始终使用“完全”模式，并且因此将跨方法引用重定向到容器的生命周期管理。这样可以防止`@Bean`通过常规Java调用意外地调用同一 方法，从而有助于减少在“精简”模式下运行时难以追查的细微错误。

的`@Bean`和`@Configuration`注解的深度在以下章节中讨论。但是，首先，我们介绍了通过基于Java的配置使用创建spring容器的各种方法。

#### 1.12.2 使用实例化Spring容器`AnnotationConfigApplicationContext`

以下各节`AnnotationConfigApplicationContext`介绍了Spring 3.0中引入的Spring。这种通用的`ApplicationContext`实现方式不仅可以接受`@Configuration`类作为输入，还可以接受 普通`@Component`类和使用JSR-330元数据注释的类。

当`@Configuration`提供类作为输入时，`@Configuration`该类本身将注册为Bean定义，并且`@Bean`该类中所有已声明的方法也将注册为Bean定义。

当提供`@Component`和JSR-330类时，它们被注册为bean定义，并且假定在必要时在这些类中使用DI元数据，例如`@Autowired`或`@Inject`。

##### 施工简单

与实例化a时将Spring XML文件用作输入的方式几乎相同，实例化a时 `ClassPathXmlApplicationContext`可以将`@Configuration`类用作输入`AnnotationConfigApplicationContext`。如下面的示例所示，这允许Spring容器的使用完全不依赖XML：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如前所述，`AnnotationConfigApplicationContext`不仅限于仅使用`@Configuration`类。`@Component`可以将任何或带有JSR-330注释的类作为输入提供给构造函数，如以下示例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

前面的例子中假定`MyServiceImpl`，`Dependency1`以及`Dependency2`使用Spring依赖注入注解，例如`@Autowired`。

##### 通过使用编程方式构建容器 `register(Class…)`

您可以`AnnotationConfigApplicationContext`使用no-arg构造函数实例化一个，然后使用`register()`方法进行配置。以编程方式构建.NET时，此方法特别有用`AnnotationConfigApplicationContext`。以下示例显示了如何执行此操作：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

##### 使用启用组件扫描 `scan(String…)`

要启用组件扫描，您可以`@Configuration`按如下方式注释您的类：

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

> 此注释启用组件扫描。

> 有经验的Spring用户可能熟悉Spring `context:`命名空间中的XML声明，如以下示例所示：

在前面的示例中，将`com.acme`扫描软件包以查找任何带 `@Component`注释的类，并将这些类注册为容器内的Spring bean定义。`AnnotationConfigApplicationContext`公开此 `scan(String…)`方法以允许相同的组件扫描功能，如以下示例所示：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

> 请记住，`@Configuration`类是[元注释](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-meta-annotations) 用`@Component`，所以他们是组件扫描候选人。在前面的示例中，假设`AppConfig`在`com.acme`包（或下面的任何包）中声明，则在调用期间将其拾取`scan()`。基于`refresh()`，其所有`@Bean` 方法都将在容器内进行处理并注册为Bean定义。

##### 支持Web应用程序 `AnnotationConfigWebApplicationContext`

一个`WebApplicationContext`变种`AnnotationConfigApplicationContext`是可用的`AnnotationConfigWebApplicationContext`。在配置Spring `ContextLoaderListener`servlet侦听器，Spring MVC等时 `DispatcherServlet`，可以使用此实现。以下`web.xml`代码片段配置了典型的Spring MVC Web应用程序（请注意使用`contextClass`context-param和init-param）：

```xml
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 1.12.3 使用`@Bean`注释

`@Bean`是方法级别的注释，是XML ``元素的直接模拟。注释支持提供的某些属性``，例如：* [init-method](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) * [destroy-method](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) * [autowiring](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-autowire) * `name`。

你可以使用`@Bean`一个注释`@Configuration`-annotated或在 `@Component`-annotated类。

##### 声明一个Bean

要声明一个bean，可以用注解对方法进行`@Bean`注解。您可以使用此方法在`ApplicationContext`指定为该方法的返回值的类型内注册Bean定义。缺省情况下，bean名称与方法名称相同。以下示例显示了`@Bean`方法声明：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

前面的配置与下面的Spring XML完全等效：

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

双方的声明作出的bean `transferService`中可用 `ApplicationContext`，势必类的对象实例`TransferServiceImpl`，如下面的文字图像显示：

```
transferService-> com.acme.TransferServiceImpl
```

您还可以`@Bean`使用接口（或基类）返回类型声明您的方法，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

但是，这将高级类型预测的可见性限制为指定的接口类型（`TransferService`）。然后，使用`TransferServiceImpl`仅一次容器已知的完整类型（），实例化受影响的单例bean。非惰性单例bean根据其声明顺序实例化，因此您可能会看到不同的类型匹配结果，具体取决于另一个组件何时尝试通过未声明的类型进行匹配（例如`@Autowired TransferServiceImpl`，，仅`transferService`在实例化bean 后才解析）。

>   如果您通过声明的服务接口一致地引用类型，则 `@Bean`返回类型可以安全地加入该设计决策。但是，对于实现多个接口的组件或由其实现类型潜在引用的组件，声明可能的最具体的返回类型（至少与引用您的bean的注入点所要求的具体类型一样）更为安全。

##### Bean依赖

带`@Bean`注释的方法可以具有任意数量的参数，这些参数描述构建该bean所需的依赖关系。例如，如果我们`TransferService` 需要一个`AccountRepository`，我们可以使用方法参数来实现该依赖关系，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

解析机制与基于构造函数的依赖注入几乎相同。[有关](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-constructor-injection)更多详细信息，请参见[相关部分](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-constructor-injection)。

##### 接收生命周期回调

用`@Bean`注释定义的任何类都支持常规的生命周期回调，并且可以使用JSR-250中的`@PostConstruct`和`@PreDestroy`注释。有关更多详细信息，请参见 [JSR-250注释](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)。

常规Spring [生命周期](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-nature)回调也得到完全支持。如果bean实现`InitializingBean`，`DisposableBean`或`Lifecycle`，则容器将调用它们各自的方法。

也完全支持标准`*Aware`接口集（例如[BeanFactoryAware](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-beanfactory)， [BeanNameAware](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware)， [MessageSourceAware](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-functionality-messagesource)， [ApplicationContextAware](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware)等）。

该`@Bean`注释支持指定任意初始化和销毁回调方法，就像Spring XML中的`init-method`和`destroy-method`属性的`bean`元素，如下面的示例所示：

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

> 默认情况下，用Java配置定义的具有公共`close`或`shutdown` 方法的bean 会自动通过销毁回调进行登记。如果您有公共 `close`或`shutdown`方法，并且不希望在容器关闭时调用它，则可以添加`@Bean(destroyMethod="")`到bean定义中以禁用默认`(inferred)`模式。默认情况下，您可能要对通过JNDI获取的资源执行此操作，因为其生命周期是在应用程序外部进行管理的。特别是，请确保始终对进行操作`DataSource`，因为这在Java EE应用程序服务器上是有问题的。以下示例显示了如何防止对的自动销毁回调 `DataSource`：爪哇科特林`@Bean(destroyMethod="") public DataSource dataSource() throws NamingException {    return (DataSource) jndiTemplate.lookup("MyDS"); }`而且，对于`@Bean`方法，通常使用程序化JNDI查找，方法是使用Spring `JndiTemplate`或`JndiLocatorDelegate`辅助方法，或者直接`InitialContext`使用JNDI 用法，但不使用`JndiObjectFactoryBean`变体（这将迫使您将返回类型声明为`FactoryBean`类型，而不是实际的目标类型，这使其很难在`@Bean`打算引用此处提供的资源的其他方法中用于交叉引用调用）。

对于`BeanOne`前面注释中的示例，`init()` 在构造期间直接调用该方法同样有效，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // ...
}
```

> 当您直接使用Java工作时，您可以对对象执行任何操作，而不必总是依赖于容器的生命周期。

##### 指定Bean范围

Spring包含`@Scope`注释，以便您可以指定bean的范围。

###### 使用`@Scope`注释

您可以指定使用`@Bean`批注定义的bean 应该具有特定范围。您可以使用[Bean Scopes](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes)部分中指定的任何标准范围 。

默认范围是`singleton`，但是您可以使用`@Scope`注释覆盖它，如以下示例所示：

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

###### `@Scope` 和 `scoped-proxy`

Spring提供了一种通过[作用域代理](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)处理作用域依赖关系的便捷方法 。使用XML配置时创建此类代理的最简单方法是``元素。使用`@Scope`注释在Java中配置bean 可以为该`proxyMode`属性提供同等的支持。默认值为no proxy（`ScopedProxyMode.NO`），但您可以指定`ScopedProxyMode.TARGET_CLASS`或`ScopedProxyMode.INTERFACES`。

如果将XML参考文档（请参阅[作用域代理](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)）中的作用域代理示例移植到我们`@Bean`使用Java的示例中 ，则它类似于以下内容：

```java
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

##### 自定义Bean命名

默认情况下，配置类使用`@Bean`方法的名称作为结果bean的名称。但是，可以使用`name`属性覆盖此功能，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean(name = "myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

##### Bean别名

如[Naming Beans中](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-beanname)所讨论的，有时希望为单个Bean提供多个名称，否则称为Bean别名。 为此`name`，`@Bean`注释的属性接受String数组。以下示例显示了如何为bean设置多个别名：

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

##### Bean描述

有时，提供有关bean的更详细的文本描述会很有帮助。当出于监视目的而暴露（可能通过JMX）bean时，这特别有用。

要将说明添加到`@Bean`，可以使用 [`@Description`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Description.html) 批注，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

#### 1.12.4 使用`@Configuration`注释

`@Configuration`是类级别的注释，指示对象是Bean定义的源。`@Configuration`类通过公共`@Bean`注释方法声明bean 。`@Bean`对`@Configuration`类的方法的调用也可以用于定义Bean之间的依赖关系。请参阅[基本概念：`@Bean`以及`@Configuration`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts)一般性介绍。

##### 注入bean间的依赖关系

当bean彼此依赖时，表达这种依赖就像让一个bean方法调用另一个依赖一样简单，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

在前面的示例中，通过构造函数注入`beanOne`接收对的引用`beanTwo`。

> 仅当`@Bean`在`@Configuration`类中声明该方法时，此声明bean间依赖关系的方法才有效。您不能使用普通`@Component`类声明bean间的依赖关系。

##### 查找方法注入

如前所述，[查找方法注入](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection)是一项高级功能，您应该很少使用。在单例作用域的bean依赖于原型作用域的bean的情况下，这很有用。将Java用于这种类型的配置为实现这种模式提供了自然的方法。以下示例显示如何使用查找方法注入：

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

通过使用Java配置，可以创建一个覆盖`CommandManager`抽象`createCommand()`方法的子类，该方法将以某种方式查找新的（原型）命令对象。以下示例显示了如何执行此操作：

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

##### 有关基于Java的配置如何在内部工作的更多信息

考虑以下示例，该示例显示了一个带`@Bean`注释的方法被调用两次：

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

`clientDao()`被称为一次`clientService1()`和一次`clientService2()`。由于此方法创建的新实例`ClientDaoImpl`并返回它，因此通常希望有两个实例（每个服务一个）。那肯定是有问题的：在Spring中，实例化的bean在`singleton`默认情况下具有作用域。这就是神奇的地方：所有`@Configuration`类在启动时都使用子类化`CGLIB`。在子类中，子方法在调用父方法并创建新实例之前，首先检查容器中是否有任何缓存（作用域）的bean。

> 根据bean的范围，行为可能有所不同。我们在这里谈论单例。

> 从Spring 3.2开始，不再需要将CGLIB添加到您的类路径中，因为CGLIB类已经被重新打包`org.springframework.cglib`并直接包含在spring-core JAR中。

> 由于CGLIB在启动时会动态添加功能，因此存在一些限制。特别是，配置类不能是最终的。但是，从4.3版本开始，配置类中允许使用任何构造函数，包括`@Autowired`对默认注入使用 或单个非默认构造函数声明。如果您希望避免任何CGLIB施加的限制，请考虑`@Bean` 在非`@Configuration`类上声明您的方法（例如，在普通`@Component`类上声明）。`@Bean`然后不会截获方法之间的跨方法调用，因此您必须专门依赖那里的构造函数或方法级别的依赖项注入。

#### 1.12.5。组成基于Java的配置

Spring的基于Java的配置功能使您可以编写批注，从而可以降低配置的复杂性。

##### 使用`@Import`注释

就像``在Spring XML文件中使用元素来帮助模块化配置一样，`@Import`注释允许`@Bean`从另一个配置类加载定义，如以下示例所示：

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

现在，无需同时指定两者`ConfigA.class`和`ConfigB.class`实例化上下文，只需`ConfigB`显式提供，如以下示例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方法简化了容器的实例化，因为只需要处理一个类，而不是要求您`@Configuration`在构造过程中记住大量潜在的 类。

>   从Spring Framework 4.2开始，`@Import`还支持对常规组件类的引用，类似于该`AnnotationConfigApplicationContext.register`方法。如果要通过使用一些配置类作为入口点来显式定义所有组件，从而避免组件扫描，则此功能特别有用。

###### 注入对导入`@Bean`定义的依赖

前面的示例有效，但过于简单。在大多数实际情况下，Bean在配置类之间相互依赖。使用XML时，这不是问题，因为不涉及任何编译器，并且您可以声明 `ref="someBean"`并信任Spring以便在容器初始化期间对其进行处理。使用`@Configuration`类时，Java编译器会在配置模型上施加约束，因为对其他bean的引用必须是有效的Java语法。

幸运的是，解决这个问题很简单。正如[我们已经讨论的那样](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-dependencies)，一种`@Bean`方法可以具有任意数量的描述Bean依赖关系的参数。考虑以下具有多个`@Configuration` 类的更真实的场景，每个类都取决于其他类中声明的bean：

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

还有另一种方法可以达到相同的结果。请记住，`@Configuration`类是在容器中最终只有另一个bean：这意味着他们可以利用 `@Autowired`与`@Value`注入等功能相同的其它bean。

> 确保以这种方式注入的依赖项只是最简单的一种。`@Configuration` 在上下文的初始化期间很早就处理了类，并且强制以这种方式注入依赖项可能会导致意外的早期初始化。如上例所示，尽可能使用基于参数的注入。另外，还要特别注意`BeanPostProcessor`和的`BeanFactoryPostProcessor`定义`@Bean`。通常应将那些声明为`static @Bean`方法，而不触发其包含的配置类的实例化。否则，`@Autowired`并且`@Value`可能不会对配置类本身的工作，因为可以将其创建为一个bean实例早于 [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)。

以下示例显示如何将一个bean自动连接到另一个bean：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

> `@Configuration`从Spring Framework 4.3开始，仅支持 在类中进行构造函数注入。还要注意，无需指定`@Autowired`目标bean是否仅定义一个构造函数。

完全合格的Bean，易于浏览

在前面的场景中，使用`@Autowired`效果很好，并提供了所需的模块化，但是要确切确定声明自动装配的bean定义的位置仍然有些模棱两可。例如，当开发人员查看时`ServiceConfig`，您如何确切知道该`@Autowired AccountRepository`bean的声明位置？它在代码中不是明确的，这可能很好。请记住， [Spring Tools for Eclipse](https://spring.io/tools)提供了可以渲染图形的工具，这些图形显示了所有接线的方式，这可能就是您所需要的。另外，您的Java IDE可以轻松找到该`AccountRepository`类型的所有声明和使用，并快速向您显示`@Bean`返回该类型的方法的位置。

如果这种歧义是不可接受的，并且您希望从IDE内部直接从一个`@Configuration`类导航到另一个类，请考虑自动装配配置类本身。以下示例显示了如何执行此操作：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

在前面的情况中，哪里`AccountRepository`定义是完全明确的。但是，`ServiceConfig`现在与紧密耦合`RepositoryConfig`。那是权衡。通过使用基于接口的类或基于抽象类的`@Configuration`类，可以在某种程度上缓解这种紧密耦合。考虑以下示例：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

现在`ServiceConfig`就具体而言松散耦合 `DefaultRepositoryConfig`，并且内置的IDE工具仍然有用：您可以轻松地获得实现的类型层次结构`RepositoryConfig`。通过这种方式，导航`@Configuration`类及其依赖项与导航基于接口的代码的通常过程没有什么不同。

> 如果要影响某些Bean的启动创建顺序，请考虑将其中一些声明为`@Lazy`（用于首次访问而不是在启动时创建）或声明为`@DependsOn`其他某些Bean（确保在当前Bean之前创建特定的其他Bean），后者的直接依赖意味着什么）。

##### 有条件地包含`@Configuration`类或`@Bean`方法

基于某些任意系统状态，有条件地启用或禁用完整的`@Configuration`类甚至单个`@Bean`方法通常很有用。一个常见的示例是`@Profile`仅在Spring中启用了特定配置文件时才使用注释激活Bean `Environment`（ 有关详细[信息](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-definition-profiles)，请参见[Bean定义配置文件](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-definition-profiles)）。

该`@Profile`注释是通过使用一种称为更灵活的注释实际执行[`@Conditional`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html)。该`@Conditional`注释指示特定 `org.springframework.context.annotation.Condition`前应谘询的实施`@Bean`是注册。

`Condition`接口的实现提供了`matches(…)` 返回`true`或的方法`false`。例如，以下清单显示了`Condition`用于的实际 实现`@Profile`：

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // Read the @Profile annotation attributes
    MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    if (attrs != null) {
        for (Object value : attrs.get("value")) {
            if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                return true;
            }
        }
        return false;
    }
    return true;
}
```

有关[`@Conditional`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html) 更多详细信息，请参见javadoc。

##### 结合Java和XML配置

Spring的`@Configuration`类支持并非旨在100％完全替代Spring XML。某些工具（例如Spring XML名称空间）仍然是配置容器的理想方法。在使用XML方便或必要的情况下，您可以选择：通过使用，以“以XML为中心”的方式实例化容器`ClassPathXmlApplicationContext`，或者通过使用`AnnotationConfigApplicationContext`和`@ImportResource`注释以“以Java中心的”方式 实例化容器。 根据需要导入XML。

###### 以XML为中心的`@Configuration`类使用

最好从XML引导Spring容器并`@Configuration`以即席方式包含 类。例如，在使用Spring XML的大型现有代码库中，根据需要创建`@Configuration`类并从现有XML文件中包含类会更容易。在本节的后面，我们将介绍`@Configuration`在这种“以XML为中心”的情况下使用类的选项。

将`@Configuration`类声明为纯Spring ``元素

请记住，`@Configuration`类最终是容器中的bean定义。在本系列示例中，我们创建一个`@Configuration`名为的类，`AppConfig`并将其包含在其中`system-test-config.xml`作为``定义。因为 ``已打开，所以容器会识别 `@Configuration`注释并 正确处理`@Bean`声明的方法`AppConfig`。

以下示例显示了Java中的普通配置类：

```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

以下示例显示了示例`system-test-config.xml`文件的一部分：

```xml
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

以下示例显示了一个可能的`jdbc.properties`文件：

```
jdbc.url = jdbc：hsqldb：hsql：// localhost / xdb 
jdbc.username = sa 
jdbc.password =
```



```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

> 在`system-test-config.xml`文件中，`AppConfig` ``没有声明`id` 元素。尽管这样做是可以接受的，但由于没有其他bean引用过它，因此这是不必要的，并且不太可能通过名称从容器中显式获取。同样，`DataSource`仅按类型自动对Bean进行接线，因此`id` 严格要求不使用显式Bean 。

使用<context：component-scan />拾取`@Configuration`类

因为`@Configuration`是间注释有`@Component`，`@Configuration`-annotated类自动对于组件扫描的候选者。使用与上一示例相同的方案，我们可以重新定义`system-test-config.xml`以利用组件扫描的优势。请注意，在这种情况下，我们无需显式声明 ``，因为``启用了相同的功能。

以下示例显示了修改后的`system-test-config.xml`文件：

```xml
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

###### `@Configuration` 以类为中心的XML使用 `@ImportResource`

在`@Configuration`类是配置容器的主要机制的应用程序中，仍然有必要至少使用一些XML。在这些情况下，您可以`@ImportResource`根据需要使用和定义尽可能多的XML。这样做实现了“以Java为中心”的方法来配置容器，并使XML保持在最低限度。以下示例（包括配置类，定义Bean的XML文件，属性文件和`main`该类）显示了如何使用`@ImportResource`注释来实现按需使用XML的“以Java为中心”的配置：



```java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```
properties-config.xml
```xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```
```
jdbc.properties 
jdbc.url = jdbc：hsqldb：hsql：// localhost / xdb 
jdbc.username = sa 
jdbc.password =
```



```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

