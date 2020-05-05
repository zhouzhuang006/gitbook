## 1.5 Bean范围

创建bean定义时，将创建一个配方来创建该bean定义所定义的类的实际实例。bean定义是配方的想法很重要，因为它意味着与类一样，您可以从一个配方中创建许多对象实例。

您不仅可以控制要插入到从特定bean定义创建的对象中的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的范围。这种方法功能强大且灵活，因为您可以选择通过配置创建的对象的范围，而不必在Java类级别上烘烤对象的范围。可以将Bean定义为部署在多个范围之一中。Spring框架支持六个范围，其中只有在使用web感知时才可用`ApplicationContext`。您还可以创建[自定义范围。](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-custom)

下表描述了受支持的范围：

**Bean作用域**

| 范围 | 描述 |
| :--- | :--- |
| [单例](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-singleton) | （默认值）将每个Spring IoC容器的单个bean定义范围限定为单个对象实例。 |
| [原型](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-prototype) | 将单个bean定义的作用域限定为任意数量的对象实例。 |
| [请求](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-request) | 将单个bean定义的范围限定为单个HTTP请求的生命周期。也就是说，每个HTTP请求都有一个在单个bean定义后面创建的bean实例。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |
| [会议](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-session) | 将单个bean定义的范围限定为HTTP的生命周期`Session`。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |
| [应用](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-application) | 将单个bean定义的作用域限定为的生命周期`ServletContext`。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |
| [网络套接字](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | 将单个bean定义的作用域限定为的生命周期`WebSocket`。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |

> 从Spring 3.0开始，线程作用域可用，但默认情况下未注册。有关更多信息，请参见的文档 SimpleThreadScope。有关如何注册此自定义范围或任何其他自定义范围的说明，请参阅 使用自定义范围。

### 1.5.1 单例范围

仅管理一个singleton bean的一个共享实例，并且所有对具有ID或与该bean定义相匹配的ID的bean的请求都会导致该特定的bean实例由Spring容器返回。

换句话说，当您定义一个bean定义并且其作用域为单例时，Spring IoC容器将为该bean定义所定义的对象创建一个实例。该单个实例存储在此类单例bean的高速缓存中，并且对该命名bean的所有后续请求和引用都返回该高速缓存的对象。下图显示了单例作用域的工作方式：

![](/assets/import2.png)

Spring的singleton bean的概念不同于“四人帮（Gang of Four，GoF）模式”一书中定义的singleton模式。GoF单例对对象的范围进行硬编码，以使每个ClassLoader只能创建一个特定类的一个实例。最好将Spring单例的范围描述为每个容器和每个bean。这意味着，如果您在单个Spring容器中为特定类定义一个bean，则Spring容器将创建该bean定义所定义的类的一个且只有一个实例。单例作用域是Spring中的默认作用域。要将bean定义为XML中的单例，可以定义bean，如以下示例所示：

```XML
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

#### 1.5.2 原型范围 {#beans-factory-scopes-prototype}

每次对特定bean提出请求时，bean部署的非单一原型范围都会导致创建一个新bean实例。也就是说，将Bean注入到另一个Bean中，或者您可以通过`getBean()`容器上的方法调用来请求它。通常，应将原型作用域用于所有有状态Bean，将单例作用域用于无状态Bean。

下图说明了Spring原型范围：

![](/assets/import3.png)

（数据访问对象（DAO）通常不配置为原型，因为典型的DAO不拥有任何对话状态。对于我们而言，重用单例图的核心更为容易。）

以下示例将bean定义为XML原型：

```XML
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域相反，Spring不管理原型Bean的完整生命周期。容器实例化，配置或组装原型对象，然后将其交给客户端，而没有该原型实例的进一步记录。因此，尽管在不考虑范围的情况下在所有对象上都调用了初始化生命周期回调方法，但对于原型而言，不会调用已配置的销毁生命周期回调。客户端代码必须清除原型作用域内的对象并释放原型Bean拥有的昂贵资源。要使Spring容器释放由原型作用域的bean占用的资源，请尝试使用自定义[bean后处理器](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp)，其中包含对需要清理的bean的引用。

在某些方面，Spring容器在原型作用域bean方面的角色是Java`new`运算符的替代。超过该时间点的所有生命周期管理必须由客户端处理。（有关Spring容器中bean的生命周期的详细信息，请参阅[Lifecycle Callbacks](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle)。）

#### 1.5.3 具有原型Bean依赖关系的Singleton Bean {#beans-factory-scopes-sing-prot-interaction}

当您使用对原型bean有依赖性的单例作用域Bean时，请注意，依赖关系在实例化时已解决。因此，如果将依赖项原型的bean依赖项注入到单例范围的bean中，则将实例化新的原型bean，然后将依赖项注入到单例bean中。原型实例是曾经提供给单例范围的bean的唯一实例。

但是，假设您希望单例作用域的bean在运行时重复获取原型作用域的bean的新实例。您不能将原型作用域的bean依赖项注入到您的单例bean中，因为当Spring容器实例化单例bean并解析并注入其依赖项时，该注入仅发生一次。如果在运行时不止一次需要原型bean的新实例，请参见[方法注入](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection)

#### 1.5.4 请求，会话，应用程序和WebSocket范围 {#beans-factory-scopes-other}

在`request`，`session`，`application`，和`websocket`范围只有当你使用一个基于web的Spring可`ApplicationContext`实现（例如`XmlWebApplicationContext`）。如果您使用这些范围与普通的Spring IoC容器，如`ClassPathXmlApplicationContext`，一个`IllegalStateException`是抱怨一个未知的bean作用域被抛出。

##### 初始Web配置 {#beans-factory-scopes-other-web-configuration}

为了支持豆的范围界定在`request`，`session`，`application`，和`websocket`（即具有web作用域bean），需要做少量的初始配置定义你的豆之前。（对于标准示波器，不需要此初始设置：`singleton`和`prototype`）

如何完成此初始设置取决于您的特定Servlet环境。

如果您在Spring Web MVC中访问作用域化的bean，实际上是在Spring处理的请求中，则`DispatcherServlet`不需要特殊的设置。`DispatcherServlet`已经公开了所有相关状态。

如果您使用Servlet 2.5 Web容器，并且在Spring之外处理请求`DispatcherServlet`（例如，使用JSF或Struts时），则需要注册`org.springframework.web.context.request.RequestContextListener  ServletRequestListener`。对于Servlet 3.0+，可以使用该`WebApplicationInitializer`接口以编程方式完成此操作。或者，或者对于较旧的容器，将以下声明添加到Web应用程序的`web.xml`文件中：

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

另外，如果您的监听器设置存在问题，请考虑使用Spring的`RequestContextFilter`。过滤器映射取决于周围的Web应用程序配置，因此您必须适当地对其进行更改。以下清单显示了Web应用程序的过滤器部分：

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`，`RequestContextListener`和`RequestContextFilter`都做完全相同的事情，即将HTTP请求对象绑定到`Thread`为该请求提供服务的对象。这使得在请求链和会话范围内的bean可以在调用链的更下游使用。

##### 要求范围 {#beans-factory-scopes-request}

考虑将以下XML配置用于bean定义：

```
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring容器LoginAction通过loginAction为每个HTTP请求使用bean定义来创建bean 的新实例。也就是说， loginActionbean的作用域位于HTTP请求级别。您可以根据需要更改创建实例的内部状态，因为从同一loginActionbean定义创建的其他实例看不到这些状态变化。它们特定于单个请求。当请求完成处理时，将丢弃作用于该请求的Bean。

使用注释驱动的组件或Java配置时，@RequestScope可以使用注释将组件分配给request作用域。以下示例显示了如何执行此操作：

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

##### 会议范围

考虑将以下XML配置用于bean定义：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

Spring容器`UserPreferences`通过在`userPreferences`单个HTTP的生存期内使用bean定义来创建bean 的新实例`Session`。换句话说，`userPreferences`bean有效地在HTTP `Session`级别范围内。与请求范围的Bean一样，您可以根据需要更改创建的实例的内部状态，因为知道其他`Session`也在使用从相同`userPreferences`Bean定义创建的实例的HTTP 实例也看不到这些状态变化，因为它们特定于单个HTTP `Session`。当`Session`最终丢弃HTTP时，作用于该特定HTTP的bean `Session`也将被丢弃。

使用注释驱动的组件或Java配置时，可以使用 `@SessionScope`注释将组件分配给`session`作用域。

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

##### 范围豆作为依赖项

Spring IoC容器不仅管理对象（bean）的实例化，而且还管理协作者（或依赖项）的连接。如果要将（例如）HTTP请求范围的Bean注入（例如）另一个作用域更长的Bean，则可以选择注入AOP代理来代替已定义范围的Bean。也就是说，您需要注入一个代理对象，该对象公开与范围对象相同的公共接口，但还可以从相关范围（例如HTTP请求）中检索实际目标对象，并将方法调用委托给该真实对象。

> 您也可以``在范围为的Bean之间`singleton`使用，引用然后经过可序列化的中间代理，因此可以在反序列化时重新获得目标单例Bean。在声明``范围bean时`prototype`，共享代理上的每个方法调用都会导致创建新的目标实例，然后将该调用转发到该目标实例。同样，作用域代理不是以生命周期安全的方式从较短的作用域访问bean的唯一方法。您还可以将注入点（即，构造函数或setter参数或自动连接的字段）声明为`ObjectFactory`，从而允许`getObject()`每次需要时调用按需检索当前实例，而无需保留该实例或将其单独存储。作为扩展变体，您可以声明`ObjectProvider`，它提供了几个附加的访问变体，包括`getIfAvailable`和`getIfUnique`。对此的JSR-330变体将被调用`Provider`，`Provider` 并在`get()`每次检索尝试中与声明和相应的调用一起使用。有关JSR-330总体的更多详细信息，请参见[此处](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-standard-annotations)。

以下示例中的配置仅一行，但是了解其背后的“原因”和“方式”很重要：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

> 定义代理的行

要创建这样的代理，请将子``元素插入到有作用域的bean定义中（请参阅[选择要创建的代理类型](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection-proxies)和 [基于XML Schema的配置](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#xsd-schemas)）。为什么Bean的定义范围为`request`，`session`并且自定义范围级别需要``元素？考虑以下单例bean定义，并将其与您需要为上述范围定义的内容进行对比（请注意，以下 `userPreferences`bean定义不完整）：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例bean（`userManager`）注入了对HTTP `Session`范围的bean（`userPreferences`）的引用。这里的要点是， `userManager`bean是单例的：每个容器仅实例化一次，并且它的依赖项（在这种情况下，仅一个，即`userPreferences`bean）也仅注入一次。这意味着`userManager`Bean仅在完全相同的`userPreferences`对象（即最初注入该对象的对象）上操作。

将寿命较短的作用域bean注入寿命较长的作用域bean时，这不是您想要的行为（例如，将HTTP `Session`范围的协作bean作为依赖项注入到singleton bean中）。相反，您只需要一个`userManager` 对象，并且在HTTP的生存期内`Session`，您需要一个`userPreferences`特定于HTTP 的对象`Session`。因此，容器创建了一个对象，该对象公开了与`UserPreferences`类完全相同的公共接口（理想情况下是作为`UserPreferences`实例的对象），该`UserPreferences`对象可以从范围确定机制（HTTP请求`Session`等）中获取实际 对象。容器将此代理对象注入到`userManager`Bean中，而后者并不知道此`UserPreferences`引用是代理。在此示例中，当 `UserManager`实例在依赖项注入`UserPreferences` 对象上调用方法，实际上是在代理上调用方法。然后，代理 `UserPreferences`从HTTP（在这种情况下）`Session`获取真实`UserPreferences`对象，并将方法调用委托给检索到的真实对象。

因此，在将Bean `request-`和`session-scoped`Bean注入到协作对象中时，需要以下配置（正确和完整） ，如以下示例所示：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

###### 选择要创建的代理类型

默认情况下，当Spring容器为使用\`\`元素标记的bean创建代理时，将创建基于CGLIB的类代理。

> CGLIB代理仅拦截公共方法调用！不要在此类代理上调用非公共方法。它们没有被委派给实际的作用域目标对象。

另外，您可以通过指定元素`false`的`proxy-target-class`属性值，将Spring容器配置为为此类作用域的Bean创建基于标准JDK接口的代理\`\`。使用基于JDK接口的代理意味着您不需要应用程序类路径中的其他库即可影响此类代理。但是，这也意味着作用域Bean的类必须实现至少一个接口，并且作用域Bean注入到其中的所有协作者必须通过其接口之一引用该Bean。以下示例显示基于接口的代理：

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择基于类或基于接口的代理的更多详细信息，请参阅[代理机制](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aop-proxying)。

#### 1.5.5 自定义范围

Bean作用域机制是可扩展的。您可以定义自己的作用域，甚至重新定义现有作用域，尽管后者被认为是不好的做法，并且您不能覆盖内置作用域`singleton`和`prototype`作用域。

##### 创建自定义范围

要将自定义范围集成到Spring容器中，您需要实现`org.springframework.beans.factory.config.Scope`本节中描述的 接口。有关如何实现自己的范围的想法，请参阅`Scope` Spring Framework本身和[`Scope`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/Scope.html)javadoc 附带的实现 ，其中详细说明了需要实现的方法。

该`Scope`接口有四种方法可以从作用域中获取对象，将其从作用域中删除，然后将其销毁。

例如，会话范围实现返回会话范围的Bean（如果不存在，则该方法将其绑定到会话上以供将来参考之后，将返回该Bean的新实例）。以下方法从基础范围返回对象：

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

会话范围的实现，例如，从基础会话中删除了会话范围的bean。应该返回该对象，但是如果找不到具有指定名称的对象，则可以返回null。以下方法将对象从基础范围中删除：

```java
Object remove(String name)
```

以下方法注册在销毁作用域或销毁作用域中的指定对象时作用域应执行的回调：

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

有关 销毁回调的更多信息，请参见[javadoc](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback)或Spring范围实现。

以下方法获取基础范围的会话标识符：

```java
String getConversationId()
```

每个范围的标识符都不相同。对于会话范围的实现，此标识符可以是会话标识符。

##### 使用自定义范围

在编写和测试一个或多个自定义`Scope`实现之后，您需要使Spring容器意识到您的新作用域。以下方法是`Scope`在Spring容器中注册新方法的主要方法：

爪哇

科特林

```java
void registerScope(String scopeName, Scope scope);
```

此方法在`ConfigurableBeanFactory`接口上声明，该接口可通过 Spring附带的`BeanFactory`大多数具体`ApplicationContext`实现上的属性获得。

该`registerScope(..)`方法的第一个参数是与范围关联的唯一名称。Spring容器本身中的此类名称示例为`singleton`和 `prototype`。该`registerScope(..)`方法的第二个参数是`Scope`您希望注册和使用的自定义实现的实际实例。

假设您编写了自定义`Scope`实现，然后注册它，如下面的示例所示。

> 下一个示例使用`SimpleThreadScope`Spring附带的，但默认情况下未注册。对于您自己的自定义`Scope` 实现，说明将是相同的。

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后，您可以按照您的custom的作用域规则创建bean定义， `Scope`如下所示：

```xml
<bean id="..." class="..." scope="thread">
```

使用自定义`Scope`实现，您不仅限于范围的程序注册。您还可以`Scope`使用`CustomScopeConfigurer`该类以声明方式进行注册 ，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

> 当您放置在`FactoryBean`实现中时，作用域是工厂Bean本身，而不是从中返回的对象`getObject()`。



