该`org.springframework.context.ApplicationContext`接口代表Spring IoC容器，并负责实例化，配置和组装Bean。容器通过读取配置元数据来获取有关要实例化，配置和组装哪些对象的指令。配置元数据以XML，Java注解或Java代码表示。它使您能够表达组成应用程序的对象以及这些对象之间的丰富相互依赖关系。

Spring提供了`ApplicationContext`接口的几种实现。在独立应用程序中，通常创建[`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html)或的实例[`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html)。尽管XML是定义配置元数据的传统格式，但是您可以通过提供少量XML配置来声明性地启用对这些其他元数据格式的支持，从而指示容器将Java注释或代码用作元数据格式。

在大多数应用场景中，不需要显式的用户代码来实例化一个Spring IoC容器的一个或多个实例。例如，在Web应用程序场景中，应用程序文件中的简单八行（约）样板Web描述符XML`web.xml`通常就足够了（请参阅[Web应用程序的便捷ApplicationContext实例化](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#context-create)）。如果您使用[Spring Tools for Eclipse](https://spring.io/tools)（Eclipse支持的开发环境），则只需单击几下鼠标或击键即可轻松创建此样板配置。

下图显示了Spring的工作原理的高级视图。您的应用程序类与配置元数据结合在一起，以便在`ApplicationContext`创建和初始化之后，您便拥有了一个完全配置且可执行的系统或应用程序。

![](/assets/import.png)

图1. Spring IoC容器

#### 配置元数据 {#beans-factory-metadata}

如上图所示，Spring IoC容器使用一种形式的配置元数据。此配置元数据表示您作为应用程序开发人员如何告诉Spring容器实例化，配置和组装应用程序中的对象。

传统上，配置元数据以简单直观的XML格式提供，这是本章大部分内容用来传达Spring IoC容器的关键概念和功能的内容。

> 基于XML的元数据不是配置元数据的唯一允许形式。Spring IoC容器本身与实际写入此配置元数据的格式完全脱钩。如今，许多开发人员为他们的Spring应用程序选择

有关在Spring容器中使用其他形式的元数据的信息，请参见：

* [基于注释的配置](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-annotation-config)：Spring 2.5引入了对基于注释的配置元数据的支持。

* [基于Java的配置](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-java)：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。因此，您可以使用Java而不是XML文件来定义应用程序类外部的bean。要使用这些新功能，请参阅[`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)，[`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)，[`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html)，和[`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html)注释。

Spring配置由容器必须管理的至少一个（通常是一个以上）bean定义组成。基于XML的配置元数据将这些bean配置为`<bean/>`顶级元素内的`<beans/>`元素。Java配置通常`@Bean`在`@Configuration`类中使用带注释的方法。

这些bean定义对应于组成应用程序的实际对象。通常，您定义服务层对象，数据访问对象（DAO），表示对象（例如Struts`Action`实例），基础结构对象（例如Hibernate`SessionFactories`，JMS`Queues`等）。通常，不会在容器中配置细粒度的域对象，因为创建和加载域对象通常是DAO和业务逻辑的职责。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器的控制范围之外创建的对象。请参阅[使用AspectJ通过Spring依赖注入域对象](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-atconfigurable)。

以下示例显示了基于XML的配置元数据的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

* 该`id`属性是标识单个bean定义的字符串。

* 该\`class\` 属性定义bean的类型， 并使用完全限定的类名。

该`id`属性的值是指协作对象。在此示例中未显示用于引用协作对象的XML。有关更多信息，请参见[依赖项](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-dependencies)。

#### 实例化容器 {#beans-factory-instantiation}

提供给`ApplicationContext`构造函数的位置路径是资源字符串，这些资源字符串使容器可以从各种外部资源（例如本地文件系统，Java等）加载配置元数据`CLASSPATH`等等。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

> 在了解了Spring的IoC容器之后，您可能想了解更多有关Spring的Resource抽象（如参考资料中所述），它提供了一种方便的机制，用于从URI语法中定义的位置读取InputStream。特别Resource是，如应用程序上下文和资源路径中所述，路径用于构造应用程序上下文。

以下示例显示了服务层对象`(services.xml)`配置文件：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

以下示例显示了数据访问对象`daos.xml`文件：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由的PetStoreServiceImpl类和类型的两个数据访问对象JpaAccountDao和JpaItemDao（基于JPA对象关系映射标准）。该property name元素是指JavaBean属性的名称，以及ref元素指的是另一个bean定义的名称。id和ref元素之间的这种联系表达了协作对象之间的依赖性。有关配置对象的依赖关系的详细信息，请参见 依赖关系。

##### 组成基于XML的配置元数据 {#beans-factory-xml-import}

使bean定义跨越多个XML文件可能很有用。通常，每个单独的XML配置文件都代表体系结构中的逻辑层或模块。

您可以使用应用程序上下文构造函数从所有这些XML片段中加载bean定义。Resource如上一节中所示，该构造函数具有多个位置 。或者，使用一个或多个出现的&lt;import/&gt;元素从另一个文件中加载bean定义。以下示例显示了如何执行操作：

```XML
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的例子中，外部豆定义是从三个文件加载： services.xml，messageSource.xml，和themeSource.xml。所有位置路径都相对于进行导入的定义文件，因此services.xml必须与进行导入的文件位于同一目录或类路径位置， messageSource.xml并且themeSource.xml必须位于resources导入文件位置下方的位置。如您所见，斜杠被忽略。但是，鉴于这些路径是相对的，最好不要使用任何斜线。&lt;beans/&gt;根据Spring Schema，导入的文件的内容（包括顶层元素）必须是有效的XML bean定义。

> 可以但不建议使用相对的“ ../”路径引用父目录中的文件。这样做会创建对当前应用程序外部文件的依赖。特别是，不建议将该引用用于classpath:URL（例如classpath:../services.xml），在URL 中，运行时解析过程将选择“最近”的类路径根，然后查看其父目录。类路径配置的更改可能导致选择其他错误的目录。
>
> 您始终可以使用完全限定的资源位置来代替相对路径：例如file:C:/config/services.xml或classpath:/config/services.xml。但是，请注意，您正在将应用程序的配置耦合到特定的绝对位置。通常最好为这样的绝对位置保留一个间接寻址-例如，通过在运行时针对JVM系统属性解析的“ $ {…}”占位符。

命名空间本身提供了导入指令功能。Spring提供的一系列XML名称空间（例如context和util名称空间）中提供了超出普通bean定义的其他配置功能。

##### Groovy Bean定义DSL {#groovy-bean-definition-dsl}

作为外部化配置元数据的另一个示例，Bean定义也可以在Spring的Groovy Bean定义DSL中表达，如Grails框架所知。

通常，这种配置位于“ .groovy”文件中，其结构如以下示例所示：

```GROOVY
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这种配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置名称空间。它还允许通过`importBeans`指令导入XML bean定义文件。

#### 使用容器 {#beans-factory-client}

`ApplicationContext`是一个维护bean定义以及相互依赖的注册表的高级工厂的接口。通过使用方法`T getBean(String name, Class<T> requiredType)`，您可以检索bean的实例。

将`ApplicationContext`让你读bean定义和访问它们，如下例所示：

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用Groovy配置，看起来非常相似。它有一个不同的上下文实现类，该类可识别Groovy（但也识别XML Bean定义）。以下示例显示了Groovy配置：

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是`GenericApplicationContext`与读取器委托结合使用，例如，与`XmlBeanDefinitionReader`XML文件结合使用，如以下示例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

您也可以将`GroovyBeanDefinitionReader`Groovy文件用于，如以下示例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以`ApplicationContext`在不同的配置源中从相同的Bean中混合和匹配此类阅读器委托，以读取Bean定义。

然后，您可以`getBean`用来检索bean的实例。`ApplicationContext`接口还有其他几种检索bean的方法，但是理想情况下，您的应用程序代码永远不要使用它们。实际上，您的应用程序代码应该根本不调用该`getBean()`方法，因此完全不依赖于Spring API。例如，Spring与Web框架的集成为各种Web框架组件（例如控制器和JSF管理的Bean）提供了依赖注入，使您可以通过元数据（例如自动装配注释）声明对特定Bean的依赖。

