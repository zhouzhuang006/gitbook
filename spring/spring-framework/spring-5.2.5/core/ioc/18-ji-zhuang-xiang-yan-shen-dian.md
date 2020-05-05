### 1.8 集装箱延伸点 {#beans-factory-extension}



通常，应用程序开发人员不需要对`ApplicationContext`实现类进行子类化。相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。接下来的几节描述了这些集成接口。

#### 1.8.1 使用a定制Bean`BeanPostProcessor`

该`BeanPostProcessor`接口定义了回调方法，您可以实现这些回调方法以提供自己的（或覆盖容器的默认值）实例化逻辑，依赖关系解析逻辑等。如果您想在Spring容器完成实例化，配置和初始化bean之后实现一些自定义逻辑，则可以插入一个或多个自定义`BeanPostProcessor`实现。

您可以配置多个`BeanPostProcessor`实例，并且可以`BeanPostProcessor`通过设置`order`属性来控制这些实例的执行顺序。仅当`BeanPostProcessor`实现`Ordered`接口时才可以设置此属性。如果您编写自己的代码`BeanPostProcessor`，则也应该考虑实现该`Ordered`接口。有关更多详细信息，请参见[`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)和[`Ordered`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/Ordered.html)接口的javadoc 。另请参见有关[实例](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-programmatically-registering-beanpostprocessors)[编程注册`BeanPostProcessor`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-programmatically-registering-beanpostprocessors)的说明。

> `BeanPostProcessor`实例在bean（或对象）实例上运行。也就是说，Spring IoC容器实例化一个bean实例，然后`BeanPostProcessor`实例执行其工作。`BeanPostProcessor`实例是按容器划分作用域的。仅在使用容器层次结构时，这才有意义。如果`BeanPostProcessor`在一个容器中定义一个，它将仅对该容器中的bean进行后处理。换句话说，一个容器中定义的bean不会被`BeanPostProcessor`另一个容器中的定义进行后处理，即使这两个容器是同一层次结构的一部分也是如此。要更改实际的bean定义（即定义bean的蓝图），您需要使用a`BeanFactoryPostProcessor`，如 使用“[定制配置元数据”中所述`BeanFactoryPostProcessor`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension-factory-postprocessors)。

该`org.springframework.beans.factory.config.BeanPostProcessor`接口恰好由两个回调方法组成。当此类被注册为容器的后处理器时，对于容器创建的每个bean实例，后处理器都会在容器初始化方法（例如`InitializingBean.afterPropertiesSet()`或任何声明的`init`方法）被使用之前从容器获得回调。在任何bean初始化回调之后调用。后处理器可以对bean实例执行任何操作，包括完全忽略回调。Bean后处理器通常检查回调接口，或者可以用代理包装Bean。一些Spring AOP基础结构类被实现为bean后处理器，以提供代理包装逻辑。

A`ApplicationContext`自动检测实现该`BeanPostProcessor`接口的配置元数据中定义的所有bean 。将`ApplicationContext`这些Bean注册为后处理器，以便以后在创建Bean时调用它们。Bean后处理器可以与其他Bean相同的方式部署在容器中。

请注意，在配置类上`BeanPostProcessor`通过使用`@Bean`工厂方法声明a时，工厂方法的返回类型应该是实现类本身或至少是`org.springframework.beans.factory.config.BeanPostProcessor`接口，以清楚地表明该bean的后处理器性质。否则，`ApplicationContext`无法在完全创建之前按类型自动检测它。由于a`BeanPostProcessor`需要提前实例化以便应用于上下文中其他bean的初始化，因此这种早期类型检测至关重要。

> 以编程方式注册`BeanPostProcessor`实例虽然建议的`BeanPostProcessor`注册方法是通过`ApplicationContext`自动检测（如前所述），但是您可以`ConfigurableBeanFactory`使用`addBeanPostProcessor`方法以编程方式针对进行注册。当您需要在注册之前评估条件逻辑，甚至需要跨层次结构中的上下文复制bean后处理器时，这将非常有用。但是请注意，以`BeanPostProcessor`编程方式添加的实例不遵守该`Ordered`接口。在这里，注册的顺序决定了执行的顺序。还要注意，以`BeanPostProcessor`编程方式注册的实例总是在通过自动检测注册的实例之前进行处理，而不考虑任何明确的顺序。

> `BeanPostProcessor`实例和AOP自动代理实现该`BeanPostProcessor`接口的类是特殊的，并且容器对它们的处理方式有所不同。`BeanPostProcessor`它们直接引用的所有实例和bean在启动时都会实例化，作为的特殊启动阶段的一部分`ApplicationContext`。接下来，`BeanPostProcessor`以排序的方式注册所有实例，并将其应用于容器中的所有其他bean。因为AOP自动代理`BeanPostProcessor`本身是实现的，所以`BeanPostProcessor`实例或它们直接引用的bean都没有资格进行自动代理，因此没有编织的方面。对于任何此类bean，您应该看到一条参考日志消息：`Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)`。如果您`BeanPostProcessor`使用自动装配或`@Resource`（可能会退回到自动装配）将Bean连接到您的 bean ，则Spring在搜索类型匹配的依赖项候选对象时可能会访问意外的bean，因此使它们不符合自动代理或其他种类的bean的要求。处理。例如，如果您有一个依赖项，`@Resource`其中字段或设置器名称不直接与bean的声明名称相对应，并且不使用name属性，则Spring将访问其他bean以按类型匹配它们。

以下示例显示了如何在中编写，注册和使用`BeanPostProcessor`实例`ApplicationContext`。

##### 示例：Hello World，`BeanPostProcessor`-style

第一个示例说明了基本用法。该示例显示了一个定制`BeanPostProcessor`实现，该实现调用`toString()`容器创建每个bean 的方法并将其输出到系统控制台。

以下清单显示了定制`BeanPostProcessor`实现类定义：

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

以下`beans`元素使用`InstantiationTracingBeanPostProcessor`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

注意`InstantiationTracingBeanPostProcessor`仅如何定义。它甚至没有名称，并且因为它是Bean，所以可以像其他任何Bean一样对其进行依赖注入。（前面的配置还定义了一个由Groovy脚本支持的bean。Spring动态语言支持在标题为[Dynamic Language Support](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/languages.html#dynamic-language)的章节中有详细介绍 。）

以下Java应用程序运行上述代码和配置：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = ctx.getBean("messenger", Messenger.class);
        System.out.println(messenger);
    }

}
```

前面的应用程序的输出类似于以下内容：

```
创建的Bean'messenger '：org.springframework.scripting.groovy.GroovyMessenger@272961 

org.springframework.scripting.groovy.GroovyMessenger@272961
```

##### 示例：`RequiredAnnotationBeanPostProcessor`

将回调接口或注释与自定义`BeanPostProcessor`实现结合使用 是扩展Spring IoC容器的常用方法。一个例子是Spring的`RequiredAnnotationBeanPostProcessor` -一个`BeanPostProcessor`与Spring发行版一起提供的实现，该 实现可确保在Bean上标有（任意）批注的JavaBean属性实际上（配置为）依赖注入了一个值。

#### 1.8.2 使用以下命令自定义配置元数据`BeanFactoryPostProcessor`

我们要看的下一个扩展点是`org.springframework.beans.factory.config.BeanFactoryPostProcessor`。该接口的语义与相似，但`BeanPostProcessor`有一个主要区别：`BeanFactoryPostProcessor`对Bean配置元数据进行操作。也就是说，Spring IoC容器允许`BeanFactoryPostProcessor`读取配置元数据，并有可能在容器实例化实例以外的任何bean_之前_更改它`BeanFactoryPostProcessor`。

您可以配置多个`BeanFactoryPostProcessor`实例，并且可以`BeanFactoryPostProcessor`通过设置`order`属性来控制这些实例的运行顺序。但是，只有在`BeanFactoryPostProcessor`实现`Ordered`接口的情况下 才能设置此属性。如果您编写自己的代码`BeanFactoryPostProcessor`，则也应该考虑实现该`Ordered`接口。有关更多详细信息，请参见[`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)和[`Ordered`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/Ordered.html)接口的javadoc 。

> 如果要更改实际的bean实例（即，从配置元数据创建的对象），则需要使用a`BeanPostProcessor`（前面在[使用a定制Bean中所述`BeanPostProcessor`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp)）。尽管在技术上可以使用`BeanFactoryPostProcessor`（例如，通过使用`BeanFactory.getBean()`）中的bean实例，但这会导致bean实例化过早，从而违反了标准容器的生命周期。这可能会导致负面影响，例如绕过bean后处理。同样，`BeanFactoryPostProcessor`实例是按容器划分作用域的。仅在使用容器层次结构时才有意义。如果`BeanFactoryPostProcessor`在一个容器中定义a ，则仅将其应用于该容器中的bean定义。一个容器中的Bean定义不会被`BeanFactoryPostProcessor`另一个容器中的实例后处理，即使两个容器都属于同一层次结构也是如此。

将Bean工厂后处理器声明为时，它将自动执行`ApplicationContext`，以便将更改应用于定义容器的配置元数据。Spring包含许多预定义的bean工厂后处理器，例如`PropertyOverrideConfigurer`和`PropertySourcesPlaceholderConfigurer`。您还可以使用自定义`BeanFactoryPostProcessor` -例如，注册自定义属性编辑器。

A`ApplicationContext`自动检测实现该`BeanFactoryPostProcessor`接口的部署到其中的所有bean 。它在适当的时候将这些bean用作bean工厂的后处理器。您可以像部署其他任何bean一样部署这些后处理器bean。

> 与`BeanPostProcessor`s一样，您通常不希望将`BeanFactoryPostProcessor`s 配置 为延迟初始化。如果没有其他bean引用a`Bean(Factory)PostProcessor`，则该后处理器将完全不会实例化。因此，将其标记为延迟初始化将被忽略，并且`Bean(Factory)PostProcessor`即使您在元素声明中将`default-lazy-init`属性设置为， 也会立即实例化 。 \`true\`\`\`

##### 示例：类名替换`PropertySourcesPlaceholderConfigurer`

您可以使用`PropertySourcesPlaceholderConfigurer`标准Java`Properties`格式，使用来从单独文件中的bean定义外部化属性值。这样做使部署应用程序的人员可以自定义特定于环境的属性，例如数据库URL和密码，而无需为修改容器的主要XML定义文件而复杂或冒风险。

考虑以下基于XML的配置元数据片段，其中`DataSource`定义了带有占位符的值：

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

该示例显示了从外部`Properties`文件配置的属性。在运行时，将a`PropertySourcesPlaceholderConfigurer`应用于替换数据源某些属性的元数据。将要替换的值指定为形式的占位符，该形式`${property-name}`遵循Ant和log4j和JSP EL样式。

实际值来自标准Java`Properties`格式的另一个文件：

```
jdbc.driverClassName = org.hsqldb.jdbcDriver 
jdbc.url = jdbc：hsqldb：hsql：// production：9002 
jdbc.username = sa 
jdbc.password = root
```

因此，`${jdbc.username}`在运行时将字符串替换为值“ sa”，并且其他与属性文件中的键匹配的占位符值也适用。在`PropertySourcesPlaceholderConfigurer`为大多数属性和bean定义的属性占位符检查。此外，您可以自定义占位符前缀和后缀。

借助`context`Spring 2.5中引入的名称空间，您可以使用专用配置元素配置属性占位符。您可以在`location`属性中提供一个或多个位置作为逗号分隔的列表，如以下示例所示：

```xml
<context:property-placeholder location="classpath:com/something/jdbc.properties"/>
```

在`PropertySourcesPlaceholderConfigurer`不仅将查找在属性`Properties`指定的文件。默认情况下，如果无法在指定的属性文件中找到属性，则将检查Spring`Environment`属性和常规Java`System`属性。

> 您可以使用`PropertySourcesPlaceholderConfigurer`代替类名，当您必须在运行时选择特定的实现类时，这有时很有用。以下示例显示了如何执行此操作：
>
> ```xml
> <bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
>     <property name="locations">
>         <value>classpath:com/something/strategy.properties</value>
>     </property>
>     <property name="properties">
>         <value>custom.strategy.class=com.something.DefaultStrategy</value>
>     </property>
> </bean>
>
> <bean id="serviceStrategy" class="${custom.strategy.class}"/>
> ```
>
> 如果无法在运行时将类解析为有效的类，则在将要创建该bean时（在非延迟初始化bean 的`preInstantiateSingletons()`阶段），将无法解析该`ApplicationContext`bean。

##### 示例：`PropertyOverrideConfigurer`

在`PropertyOverrideConfigurer`另一个bean工厂后处理器，类似`PropertySourcesPlaceholderConfigurer`，但不同的是后者，原来的定义可以在所有的bean属性有缺省值或者根本没有值。如果覆盖`Properties`文件没有某个bean属性的条目，则使用默认的上下文定义。

注意，bean定义不知道会被覆盖，因此从XML定义文件中不能立即看出正在使用覆盖配置器。如果有多个`PropertyOverrideConfigurer`实例为同一个bean属性定义了不同的值，则由于覆盖机制，最后一个实例将获胜。

属性文件配置行采用以下格式：

```
beanName.property =值
```

下面的清单显示了格式的示例：

```
dataSource.driverClassName = com.mysql.jdbc.Driver 


dataSource.url = jdbc：mysql：mydb
```

此示例文件可与包含定义为`dataSource`具有`driver`和`url`属性的bean的容器定义一起使用 。

只要路径的每个组成部分（最终属性被覆盖）之外的所有组成部分都已经为非空（可能是由构造函数初始化），则也支持复合属性名。在以下示例中，bean`sammy`的`bob`property的`fred`property属性`tom`设置为标量值`123`：

```
tom.fred.bob.sammy = 123
```

> 指定的替代值始终是文字值。它们不会转换为bean引用。当XML bean定义中的原始值指定bean引用时，此约定也适用。

使用`context`Spring 2.5中引入的名称空间，可以使用专用配置元素配置属性覆盖，如以下示例所示：

```
<context:property-override location="classpath:override.properties"/>
```

#### 1.8.3 自定义实例化逻辑`FactoryBean`

您可以`org.springframework.beans.factory.FactoryBean`为本身就是工厂的对象实现接口。

该`FactoryBean`接口是可插入Spring IoC容器的实例化逻辑的一点。如果您拥有复杂的初始化代码，而不是（可能）冗长的XML量，可以用Java更好地表达，则可以创建自己的代码`FactoryBean`，在该类中编写复杂的初始化，然后将自定义`FactoryBean`插入容器。

该`FactoryBean`界面提供了三种方法：

* `Object getObject()`：返回此工厂创建的对象的实例。实例可以共享，具体取决于该工厂是否返回单例或原型。

* `boolean isSingleton()`：`true`如果`FactoryBean`返回单例或`false`其他则返回 。

* `Class getObjectType()`：返回`getObject()`方法返回的对象类型，或者`null`如果类型未知则返回该对象类型。

`FactoryBean`Spring框架中的许多地方都使用了该概念和接口。`FactoryBean`Spring附带了50多种接口实现。

当您需要向容器询问`FactoryBean`本身而不是由它产生的bean的实际实例时，请在调用的方法时在该bean的`id`前面加上“＆”符号（`&`）。因此，对于给定 与的，调用在容器回报的产品，而调用返回的 实例本身。```getBean()``ApplicationContext``FactoryBean``id``myBean``getBean("myBean")``FactoryBean``getBean("&myBean")``FactoryBean```



