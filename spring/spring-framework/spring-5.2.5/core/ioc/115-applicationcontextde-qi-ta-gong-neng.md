### 1.15 `ApplicationContext`的其他功能 {#context-introduction}


如[本章简介](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans)中所述，该`org.springframework.beans.factory` 软件包提供了用于管理和操纵Bean的基本功能，包括以编程方式。的`org.springframework.context`封装增加了 [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html) 接口，它扩展了`BeanFactory`界面，除了延伸其它接口以更应用面向框架的方式提供附加的功能。许多人`ApplicationContext`以完全声明性的方式使用，甚至没有以编程方式创建它，而是依靠支持类，例如在Java EE Web应用程序正常启动过程中`ContextLoader`自动实例化一个 `ApplicationContext`。

为了`BeanFactory`以更加面向框架的方式增强功能，上下文包还提供以下功能：

- 通过`MessageSource`界面访问i18n样式的消息。
- 通过`ResourceLoader`界面访问资源，例如URL和文件。
- 事件发布，即`ApplicationListener`通过使用接口向实现接口的bean `ApplicationEventPublisher`。
- 加载多个（分层）上下文，使每个上下文都通过`HierarchicalBeanFactory`界面集中在一个特定的层上，例如应用程序的Web层 。

#### 1.15.1。国际化使用`MessageSource`

该`ApplicationContext`接口扩展了称为的接口`MessageSource`，因此提供了国际化（“ i18n”）功能。Spring还提供了 `HierarchicalMessageSource`接口，该接口可以分层解析消息。这些接口一起提供了Spring影响消息解析的基础。这些接口上定义的方法包括：

- `String getMessage(String code, Object[] args, String default, Locale loc)`：用于从中检索消息的基本方法`MessageSource`。如果找不到指定语言环境的消息，则使用默认消息。使用`MessageFormat`标准库提供的功能，传入的所有参数都将成为替换值。
- `String getMessage(String code, Object[] args, Locale loc)`：与以前的方法基本相同，但有一个区别：不能指定默认消息。如果找不到该消息，`NoSuchMessageException`则引发a。
- `String getMessage(MessageSourceResolvable resolvable, Locale locale)`：前述方法中使用的所有属性也都包装在名为的类中 `MessageSourceResolvable`，您可以将其与此方法一起使用。

当`ApplicationContext`被加载时，它自动搜索`MessageSource` 在上下文中定义的bean。Bean必须具有名称`messageSource`。如果找到了这样的bean，则对先前方法的所有调用都将委派给消息源。如果找不到消息源，则`ApplicationContext`尝试查找包含同名bean的父对象。如果是这样，它将使用该bean作为`MessageSource`。如果`ApplicationContext`无法找到任何消息源，则将 `DelegatingMessageSource`实例化为空 ，以便能够接受对上述方法的调用。

Spring提供了两种`MessageSource`实现，`ResourceBundleMessageSource`和 `StaticMessageSource`。两者都是`HierarchicalMessageSource`为了执行嵌套消息传递而实现的。在`StaticMessageSource`很少使用，但提供编程的方式向消息源添加消息。以下示例显示`ResourceBundleMessageSource`：

```xml
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

该示例假定您有三个资源包，称为`format`，`exceptions`并`windows` 在类路径中定义。任何解析消息的请求都通过JDK标准的通过`ResourceBundle`对象解析消息的方式来处理。就本示例而言，假定上述两个资源束文件的内容如下：

```
    ＃in format.properties 
    message =鳄鱼成群结队！
    ＃in exceptions.properties 
    parameter.required = {0}参数是必需的。
```

下一个示例显示了执行该`MessageSource`功能的程序。请记住，所有`ApplicationContext`实现也是`MessageSource` 实现，因此可以强制转换为`MessageSource`接口。

爪哇

科特林

```java
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```

以上程序的结果输出如下：

```
短吻鳄摇滚！
```

概括而言，`MessageSource`定义在名为的文件中`beans.xml`，该文件位于类路径的根目录下。该`messageSource`bean定义是指通过它的一些资源包的`basenames`属性。这是在列表中传递的三个文件`basenames`属性存在于你的classpath根目录的文件，被称为`format.properties`，`exceptions.properties`和 `windows.properties`分别。

下一个示例显示了传递给消息查找的参数。这些参数将转换为`String`对象，并插入到查找消息中的占位符中。

```xml
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.something.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

爪哇

科特林

```java
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```

该`execute()`方法的调用结果如下：

```
userDao参数是必需的。
```

关于国际化（“ i18n”），Spring的各种`MessageSource` 实现遵循与标准JDK相同的语言环境解析和后备规则 `ResourceBundle`。总之，和继续该示例`messageSource`先前定义的，如果你想对英国（解析的消息`en-GB`）的语言环境，你需要创建的文件名为`format_en_GB.properties`，`exceptions_en_GB.properties`和 `windows_en_GB.properties`分别。

通常，语言环境解析由应用程序的周围环境管理。在以下示例中，手动指定了针对其解析（英国）消息的语言环境：

```
＃在exceptions_en_GB.properties 
parameter.required = Ebagum lad中，“ {0}”自变量是必需的，我说是必需的。
```

爪哇

科特林

```java
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

运行上述程序的结果输出如下：

```
Ebagum伙计，“ userDao”参数是必需的，我说是必需的。
```

您还可以使用该`MessageSourceAware`接口获取对`MessageSource`已定义的任何内容的引用 。在创建和配置Bean时，将在`ApplicationContext`实现该`MessageSourceAware`接口的中定义的任何Bean都 与应用程序上下文一起注入`MessageSource`。

|      | 作为替代`ResourceBundleMessageSource`，Spring提供了一个 `ReloadableResourceBundleMessageSource`类。该变体支持相同的捆绑文件格式，但比基于标准JDK的`ResourceBundleMessageSource`实现更灵活 。特别是，它允许从任何Spring资源位置（不仅是从类路径）读取文件，并支持热重装捆绑属性文件（同时在它们之间进行有效缓存）。有关[`ReloadableResourceBundleMessageSource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html) 详细信息，请参见javadoc。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.15.2。标准和自定义事件

`ApplicationContext`通过`ApplicationEvent` 类和`ApplicationListener`接口提供中的事件处理。如果将实现`ApplicationListener`接口的Bean 部署到上下文中，则每次 将Bean `ApplicationEvent`发布到时`ApplicationContext`，都会通知该Bean。本质上，这是标准的观察者设计模式。

|      | 从Spring 4.2开始，事件基础结构得到了显着改进，并提供了[基于注释的模型](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-functionality-events-annotation)以及发布任意事件的能力（即，对象不一定从扩展`ApplicationEvent`）。发布此类对象后，我们会为您包装一个事件。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下表描述了Spring提供的标准事件：

| 事件                         | 说明                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| `ContextRefreshedEvent`      | 在`ApplicationContext`初始化或刷新时发布（例如，通过使用接口`refresh()`上的方法`ConfigurableApplicationContext`）。在这里，“已初始化”表示已加载所有bean，检测并激活了后处理器bean，已预先实例化单例，并且该`ApplicationContext`对象已准备就绪可以使用。只要尚未关闭上下文，就可以多次触发刷新，前提是所选对象`ApplicationContext`实际上支持这种“热”刷新。例如，`XmlWebApplicationContext`支持热刷新，但`GenericApplicationContext`不支持 。 |
| `ContextStartedEvent`        | `ApplicationContext`使用界面`start()`上的方法 启动时发布`ConfigurableApplicationContext`。在这里，“启动”意味着所有`Lifecycle` bean都收到一个明确的启动信号。通常，此信号用于在显式停止后重新启动Bean，但也可以用于启动尚未配置为自动启动的组件（例如，尚未在初始化时启动的组件）。 |
| `ContextStoppedEvent`        | `ApplicationContext`通过使用界面`stop()`上的方法 停止时发布`ConfigurableApplicationContext`。此处，“已停止”表示所有`Lifecycle` bean均收到明确的停止信号。停止的上下文可以通过`start()`调用重新启动 。 |
| `ContextClosedEvent`         | 当发布时间`ApplicationContext`是由使用封闭`close()`方法的上`ConfigurableApplicationContext`接口或经由JVM关闭挂钩。在这里，“封闭”意味着所有单例豆将被销毁。关闭上下文后，它将达到使用寿命，无法刷新或重新启动。 |
| `RequestHandledEvent`        | 一个特定于Web的事件，告诉所有Bean HTTP请求已得到服务。请求完成后，将发布此事件。此事件仅适用于使用Spring的Web应用程序`DispatcherServlet`。 |
| `ServletRequestHandledEvent` | 该类的子类`RequestHandledEvent`添加了特定于Servlet的上下文信息。 |

您还可以创建和发布自己的自定义事件。以下示例显示了一个简单的类，该类扩展了Spring的`ApplicationEvent`基类：

爪哇

科特林

```java
public class BlackListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlackListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

要发布自定义`ApplicationEvent`，请在`publishEvent()`上调用方法 `ApplicationEventPublisher`。通常，这是通过创建一个实现`ApplicationEventPublisherAware`并注册为Spring bean 的类来完成的 。以下示例显示了此类：

爪哇

科特林

```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blackList;
    private ApplicationEventPublisher publisher;

    public void setBlackList(List<String> blackList) {
        this.blackList = blackList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blackList.contains(address)) {
            publisher.publishEvent(new BlackListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

在配置时，Spring容器检测到该`EmailService`实现 `ApplicationEventPublisherAware`并自动调用 `setApplicationEventPublisher()`。实际上，传入的参数是Spring容器本身。您正在通过其`ApplicationEventPublisher`界面与应用程序上下文进行 交互。

要接收该定制`ApplicationEvent`，您可以创建一个实现 `ApplicationListener`并注册为Spring bean的类。以下示例显示了此类：

爪哇

科特林

```java
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

注意，`ApplicationListener`通常使用自定义事件的类型来参数化（`BlackListEvent`在前面的示例中）。这意味着该`onApplicationEvent()`方法可以保持类型安全，而无需进行向下转换。您可以根据需要注册任意数量的事件侦听器，但是请注意，默认情况下，事件侦听器会同步接收事件。这意味着该`publishEvent()`方法将阻塞，直到所有侦听器都已完成对事件的处理为止。这种同步和单线程方法的一个优点是，当侦听器收到事件时，如果有可用的事务上下文，它将在发布者的事务上下文内部进行操作。如果需要其他用于事件发布的策略，请参阅javadoc中Spring的 [`ApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/event/ApplicationEventMulticaster.html)界面和[`SimpleApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html) 实现的配置选项。

以下示例显示了用于注册和配置上述每个类的Bean定义：

```xml
<bean id="emailService" class="example.EmailService">
    <property name="blackList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blackListNotifier" class="example.BlackListNotifier">
    <property name="notificationAddress" value="blacklist@example.org"/>
</bean>
```

将所有内容放在一起，当调用bean 的`sendEmail()`方法时`emailService`，如果有任何电子邮件消息应列入黑名单，则将`BlackListEvent`发布一个类型的自定义事件 。该`blackListNotifier`bean被注册为 `ApplicationListener`接收的`BlackListEvent`，在这一点上，可以通知有关各方。

|      | Spring的事件机制旨在在同一应用程序上下文内在Spring bean之间进行简单的通信。但是，对于更复杂的企业集成需求，单独维护的 [Spring Integration](https://projects.spring.io/spring-integration/)项目提供了对基于众所周知的Spring编程模型构建轻量级，[面向模式](https://www.enterpriseintegrationpatterns.com/)，事件驱动的架构的完整支持 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 基于注释的事件侦听器

从Spring 4.2开始，您可以使用`@EventListener`注释在托管Bean的任何公共方法上注册事件侦听器。该`BlackListNotifier`可改写如下：

爪哇

科特林

```java
public class BlackListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlackListEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

方法签名再次声明其侦听的事件类型，但是这次使用灵活的名称且未实现特定的侦听器接口。只要实际事件类型在其实现层次结构中解析您的通用参数，也可以通过通用类型来缩小事件类型。

如果您的方法应该侦听多个事件，或者您要完全不使用任何参数来定义它，则事件类型也可以在注释本身上指定。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

也可以通过使用`condition`定义[`SpEL`表达式](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#expressions)的注释的属性来添加其他运行时过滤，该[表达式](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#expressions)应匹配以针对特定事件实际调用该方法。

以下示例说明了仅当`content`事件的属性等于时，才可以重写我们的通知程序以进行调用 `my-event`：

爪哇

科特林

```java
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlackListEvent(BlackListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```

每个`SpEL`表达式针对专用上下文进行评估。下表列出了可用于上下文的项目，以便您可以将它们用于条件事件处理：

| 名称       | 位置     | 描述                                                         | 例                                                           |
| :--------- | :------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 事件       | 根对象   | 实际的`ApplicationEvent`。                                   | `#root.event` 要么 `event`                                   |
| 参数数组   | 根对象   | 用于调用方法的参数（作为对象数组）。                         | `#root.args`或`args`; `args[0]`访问第一个参数，等等。        |
| *参数名称* | 评价背景 | 任何方法参数的名称。如果由于某种原因这些名称不可用（例如，因为在已编译的字节码中没有调试信息），则还可以使用`#a<#arg>`其中`<#arg>`参数代表参数索引（从0开始）的语法提供各个参数。 | `#blEvent`或`#a0`（您也可以使用`#p0`或`#p<#arg>`参数符号作为别名） |

请注意`#root.event`，即使您的方法签名实际上指向已发布的任意对象，也可以使您访问基础事件。

如果由于处理另一个事件而需要发布一个事件，则可以更改方法签名以返回应发布的事件，如以下示例所示：

爪哇

科特林

```java
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

|      | [异步侦听](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-functionality-events-async) 器不支持此功能 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

此新方法`ListUpdateEvent`为`BlackListEvent`上述方法处理的每个操作发布一个新方法。如果您需要发布多个事件，则可以返回一个`Collection`事件。

##### 异步侦听器

如果您希望特定的侦听器异步处理事件，则可以重用 [常规`@Async`支持](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/integration.html#scheduling-annotation-support-async)。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event) {
    // BlackListEvent is processed in a separate thread
}
```

使用异步事件时，请注意以下限制：

- 如果异步事件侦听器抛出`Exception`，则不会传播到调用者。请参阅`AsyncUncaughtExceptionHandler`以获取更多详细信息。
- 异步事件侦听器方法无法通过返回值来发布后续事件。如果您需要发布另一个事件作为处理的结果，请插入一个 [`ApplicationEventPublisher`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html) 以手动发布事件。

##### 订购听众

如果需要先调用一个侦听器，则可以将`@Order` 注释添加到方法声明中，如以下示例所示：

爪哇

科特林

```java
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

##### 一般事件

您还可以使用泛型来进一步定义事件的结构。考虑使用 `EntityCreatedEvent`where `T`是创建的实际实体的类型。例如，您可以创建以下侦听器定义只接收`EntityCreatedEvent`了 `Person`：

爪哇

科特林

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

由于类型擦除，只有在触发的事件解析了事件侦听器所基于的通用参数（即`class PersonCreatedEvent extends EntityCreatedEvent { … }`）时，此方法才起作用 。

在某些情况下，如果所有事件都遵循相同的结构，这可能会变得很乏味（就像前面示例中的事件一样）。在这种情况下，您可以实现`ResolvableTypeProvider`超出运行时环境提供的范围之外的框架指导。以下事件显示了如何执行此操作：

爪哇

科特林

```java
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
    }
}
```

|      | 这不仅适用于`ApplicationEvent`作为事件发送的任何对象，而且适用于该对象。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.15.3。方便地访问低级资源

为了最佳使用和理解应用程序上下文，您应该熟悉Spring的`Resource`抽象，如[参考资料中所述](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#resources)。

应用程序上下文是`ResourceLoader`，可用于加载`Resource`对象。A `Resource`本质上是JDK `java.net.URL`类的功能更丰富的版本。实际上，在适当`Resource`的情况下`java.net.URL`，wrap 的实现包装了一个实例。A `Resource`可以以透明的方式从几乎任何位置获取低级资源，包括从类路径，文件系统位置，可使用标准URL描述的任何位置以及一些其他变体。如果资源位置字符串是没有任何特殊前缀的简单路径，则这些资源的来源是特定的，并且适合于实际的应用程序上下文类型。

您可以配置部署到应用程序上下文中的Bean，以实现特殊的回调接口`ResourceLoaderAware`，以便在初始化时以应用程序上下文本身作为传入自动回调`ResourceLoader`。您还可以公开type属性，`Resource`用于访问静态资源。它们像其他任何属性一样注入其中。您可以将这些`Resource` 属性指定为简单`String`路径，并在`Resource`部署Bean时依靠从这些文本字符串到实际对象的自动转换。

提供给`ApplicationContext`构造函数的一个或多个位置路径实际上是资源字符串，并且根据特定的上下文实现以简单的形式对其进行了适当处理。例如，`ClassPathXmlApplicationContext`将简单的位置路径视为类路径位置。您也可以使用带有特殊前缀的位置路径（资源字符串）来强制从类路径或URL中加载定义，而不管实际的上下文类型如何。

#### 1.15.4。Web应用程序的便捷ApplicationContext实例化

您可以`ApplicationContext`使用声明性地创建实例 `ContextLoader`。当然，您也可以`ApplicationContext`使用其中一种`ApplicationContext`实现以编程方式创建实例。

您可以`ApplicationContext`使用来注册一个`ContextLoaderListener`，如以下示例所示：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

侦听器检查`contextConfigLocation`参数。如果该参数不存在，那么侦听器将使用它`/WEB-INF/applicationContext.xml`作为默认值。当参数确实存在时，侦听器将`String`使用预定义的定界符（逗号，分号和空格）将分隔，并将这些值用作搜索应用程序上下文的位置。还支持蚂蚁风格的路径模式。示例是`/WEB-INF/*Context.xml`（对于名称以结尾 `Context.xml`且位于`WEB-INF`目录`/WEB-INF/**/*Context.xml` 中的所有文件）和（对于的任何子目录中的所有此类文件`WEB-INF`）。

#### 1.15.5。将Spring部署`ApplicationContext`为Java EE RAR文件

可以将Spring部署`ApplicationContext`为RAR文件，将上下文及其所有必需的Bean类和库JAR封装在Java EE RAR部署单元中。这等效于引导`ApplicationContext`能够访问Java EE服务器功能的独立服务器（仅托管在Java EE环境中）。对于部署无头WAR文件的情况，RAR部署是一种更自然的选择-实际上，这种WAR文件没有任何HTTP入口点，仅用于`ApplicationContext`在Java EE环境中引导Spring 。

对于不需要HTTP入口点而仅由消息端点和计划的作业组成的应用程序上下文，RAR部署是理想的选择。在这样的上下文中，Bean可以使用应用程序服务器资源，例如JTA事务管理器以及绑定到JNDI的JDBC `DataSource`实例和JMS `ConnectionFactory`实例，还可以通过该平台的JMX服务器注册-全部通过Spring的标准事务管理以及JNDI和JMX支持工具。应用程序组件还可以`WorkManager`通过Spring的`TaskExecutor`抽象与应用程序服务器的JCA 进行交互。

有关[`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html) RAR部署中涉及的配置详细信息，请参见该类的javadoc 。

为了将Spring ApplicationContext作为Java EE RAR文件进行简单部署：

1. 将所有应用程序类打包到RAR文件（这是具有不同文件扩展名的标准JAR文件）中。将所有必需的库JAR添加到RAR归档文件的根目录中。添加一个 `META-INF/ra.xml`部署描述符（如[javadoc中的`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html)所示）和相应的Spring XML bean定义文件（通常为 `META-INF/applicationContext.xml`）。
2. 将生成的RAR文件拖放到应用程序服务器的部署目录中。

|      | 这种RAR部署单元通常是独立的。它们不会将组件暴露给外界，甚至不会暴露给同一应用程序的其他模块。与基于RAR的交互`ApplicationContext`通常是通过与其他模块共享的JMS目标进行的。基于RAR的文件`ApplicationContext`还可以例如计划一些作业或对文件系统（或类似文件）中的新文件做出反应。如果需要允许来自外部的同步访问，则可以（例如）导出RMI端点，该端点可以由同一台计算机上的其他应用程序模块使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |
