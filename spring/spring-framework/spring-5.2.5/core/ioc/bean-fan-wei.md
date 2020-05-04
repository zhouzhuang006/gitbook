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

```
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

```
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









