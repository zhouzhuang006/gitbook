### 1.6 自定义Bean的性质 {#beans-factory-nature}

Spring框架提供了许多接口，可用于自定义Bean的性质。本节将它们分组如下：

* [生命周期回调](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle)

* [`ApplicationContextAware`和`BeanNameAware`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware)

* [其他`Aware`介面](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aware-list)

#### 1.6.1 生命周期回调

为了与容器对bean生命周期的管理进行交互，可以实现Spring`InitializingBean`和`DisposableBean`接口。容器要求`afterPropertiesSet()`前者和`destroy()`后者使bean在初始化和销毁bean时执行某些操作。

> 通常，JSR-250`@PostConstruct`和`@PreDestroy`注释被认为是在现代Spring应用程序中接收生命周期回调的最佳实践。使用这些注释意味着您的bean没有耦合到特定于Spring的接口。有关详细信息，请参见[使用`@PostConstruct`和`@PreDestroy`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)。如果你不希望使用JSR-250注解，但你仍然要删除的耦合，考虑`init-method`和`destroy-method`bean定义元数据。

在内部，Spring Framework使用`BeanPostProcessor`实现来处理它可以找到的任何回调接口并调用适当的方法。如果您需要自定义功能或其他生命周期行为，Spring默认不提供，则您可以`BeanPostProcessor`自己实现。有关更多信息，请参见[容器扩展点](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension)。

除了初始化和销毁回调，Spring管理的对象还可以实现`Lifecycle`接口，以便这些对象可以在容器自身生命周期的驱动下参与启动和关闭过程。

本节介绍了生命周期回调接口。

##### 初始化回调

`org.springframework.beans.factory.InitializingBean`容器在bean上设置了所有必需的属性后，该接口使bean可以执行初始化工作。该`InitializingBean`接口指定一个方法：

```java
void afterPropertiesSet() throws Exception;
```

我们建议您不要使用该`InitializingBean`接口，因为它不必要地将代码耦合到Spring。另外，我们建议使用[`@PostConstruct`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)注释或指定POJO初始化方法。对于基于XML的配置元数据，可以使用`init-method`属性指定具有无效无参数签名的方法的名称。通过Java配置，您可以使用的`initMethod`属性`@Bean`。请参阅[接收生命周期回调](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-lifecycle-callbacks)。考虑以下示例：

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

前面的示例与下面的示例（由两个清单组成）几乎具有完全相同的效果：

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但是，前面两个示例中的第一个示例并未将代码耦合到Spring。

##### 销毁回调

`org.springframework.beans.factory.DisposableBean`当包含该接口的容器被销毁时，实现该接口可使Bean获得回调。该`DisposableBean`接口指定一个方法：

```
void destroy() throws Exception;
```

我们建议您不要使用`DisposableBean`回调接口，因为它不必要地将代码耦合到Spring。另外，我们建议使用[`@PreDestroy`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)注释或指定bean定义支持的通用方法。对于基于XML的配置元数据，您可以在`destroy-method`上使用属性```。通过Java配置，您可以使用的``destroyMethod`属性`@Bean\`。请参阅[接收生命周期回调](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-lifecycle-callbacks)。考虑以下定义：

```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

前面的定义与下面的定义几乎具有完全相同的效果：

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```



```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但是，前面两个定义中的第一个没有将代码耦合到Spring。

> 您可以`destroy-method`为``元素的属性分配一个特殊 `(inferred)`值，该值指示Spring自动检测特定bean类上的公共`close`或 `shutdown`方法。（实现`java.lang.AutoCloseable`或`java.io.Closeable`匹配的任何类 。）您还可以`(inferred)`在元素的`default-destroy-method`属性 上设置此特殊值，``以将该行为应用于整个bean集（请参见[Default Initialization and Destroy Methods](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-default-init-destroy-methods)）。请注意，这是Java配置的默认行为。

##### 默认初始化和销毁方法

当你写的初始化和销毁不使用Spring的具体方法回调`InitializingBean`和`DisposableBean`回调接口，你的名字，如通常的写入方法`init()`，`initialize()`，`dispose()`，等等。理想情况下，此类生命周期回调方法的名称应在整个项目中标准化，以便所有开发人员都使用相同的方法名称并确保一致性。

您可以将Spring容器配置为“查找”命名的初始化，并销毁每个bean上的回调方法名称。这意味着作为应用程序开发人员，您可以编写应用程序类并使用称为的初始化回调`init()`，而不必为`init-method="init"`每个bean定义配置属性。Spring IoC容器在创建bean时（并按照[前面描述](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle)的标准生命周期回调协定）调用该方法。此功能还对初始化和销毁方法回调强制执行一致的命名约定。

假设您的初始化回调方法已命名，`init()`而destroy回调方法已命名`destroy()`。然后，您的课程类似于以下示例中的课程：

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

然后，您可以在类似于以下内容的Bean中使用该类：

```
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

`default-init-method`顶级```元素属性上存在该属性会导致Spring IoC容器将``init\`在bean类上调用的方法识别为初始化方法回调。创建和组装bean时，如果bean类具有这种方法，则会在适当的时间调用它。

您可以使用`default-destroy-method`顶级\`\`元素上的属性，以类似方式（在XML中）配置destroy方法回调 。

如果现有的Bean类已经具有按惯例命名的回调方法，则可以通过使用 自身的`init-method`and`destroy-method`属性指定（在XML中）方法名称来覆盖默认值\`\`。

Spring容器保证在为bean提供所有依赖项后立即调用已配置的初始化回调。因此，在原始bean引用上调用了初始化回调，这意味着AOP拦截器等尚未应用于bean。首先完全创建目标bean，然后应用带有其拦截器链的AOP代理（例如）。如果目标Bean和代理分别定义，则您的代码甚至可以绕过代理与原始目标Bean进行交互。因此，将拦截器应用于该`init`方法将是不一致的，因为这样做会将目标Bean的生命周期耦合到其代理或拦截器，并在代码直接与原始目标Bean交互时留下奇怪的语义。

##### 组合生命周期机制

从Spring 2.5开始，您可以使用三个选项来控制Bean生命周期行为：

* 在[`InitializingBean`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)和[`DisposableBean`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)回调接口

* 习惯`init()`和`destroy()`方法

* 在[`@PostConstruct`和`@PreDestroy`注释](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)。您可以结合使用这些机制来控制给定的bean。

> 如果为一个bean配置了多个生命周期机制，并且为每个机制配置了不同的方法名称，则将按照此注释后列出的顺序执行每个已配置的方法。但是，如果`init()`为多个生命周期机制中的多个生命周期机制配置了相同的方法名（例如， 对于初始化方法），则该方法将执行一次，如上[一节所述](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-default-init-destroy-methods)。

为同一个bean配置的具有不同初始化方法的多种生命周期机制称为：

1. 用注释的方法`@PostConstruct`

2. `afterPropertiesSet()`由`InitializingBean`回调接口定义

3. 定制配置的`init()`方法

销毁方法的调用顺序相同：

1. 用注释的方法`@PreDestroy`

2. `destroy()`由`DisposableBean`回调接口定义

3. 定制配置的`destroy()`方法

##### 启动和关机回调

该`Lifecycle`接口为具有自己生命周期要求（例如启动和停止某些后台进程）的任何对象定义了基本方法：

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现该`Lifecycle`接口。然后，当`ApplicationContext`自身接收到启动和停止信号时（例如，对于运行时的停止/重新启动场景），它将这些调用级联到`Lifecycle`在该上下文中定义的所有实现。它通过委派给来完成此操作`LifecycleProcessor`，如以下清单所示：

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

请注意，`LifecycleProcessor`本身是`Lifecycle`接口的扩展。它还添加了两种其他方法来对刷新和关闭的上下文做出反应。

> 请注意，常规`org.springframework.context.Lifecycle`接口是用于显式启动和停止通知的普通协议，并不意味着在上下文刷新时自动启动。为了对特定bean的自动启动进行精细控制（包括启动阶段），请考虑实施`org.springframework.context.SmartLifecycle`。另外，请注意，不能保证会在销毁之前发出停止通知。在常规关闭时，`Lifecycle`在传播常规销毁回调之前，所有Bean首先都会收到停止通知。但是，在上下文生存期内的热刷新或中止的刷新尝试中，仅调用destroy方法。

启动和关闭调用的顺序可能很重要。如果任何两个对象之间存在“依赖”关系，则依赖方在其依赖之后开始，而在依赖之前停止。但是，有时直接依赖项是未知的。您可能只知道某种类型的对象应该先于另一种类型的对象开始。在这些情况下，`SmartLifecycle`接口定义另一个选项，即`getPhase()`在其超级接口上定义的方法`Phased`。以下清单显示了`Phased`接口的定义：

```java
public interface Phased {

    int getPhase();
}
```

以下清单显示了`SmartLifecycle`接口的定义：

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，相位最低的对象首先启动。停止时，遵循相反的顺序。因此，实现`SmartLifecycle`并`getPhase()`返回其方法的对象`Integer.MIN_VALUE`将是第一个启动且最后一个停止的对象。在频谱的另一端，相位值`Integer.MAX_VALUE`表示应该最后启动对象，然后首先停止对象（可能是因为它取决于正在运行的其他进程）。当考虑相位值，同样重要的是要知道，对于任何“正常”的默认阶段`Lifecycle`目标没有实现`SmartLifecycle`的`0`。因此，任何负相位值都表明对象应在这些标准组件之前开始（并在它们之后停止）。对于任何正相位值，反之亦然。

定义的stop方法`SmartLifecycle`接受回调。`run()`在该实现的关闭过程完成之后，任何实现都必须调用该回调的方法。由于`LifecycleProcessor`接口 的默认实现`DefaultLifecycleProcessor`会在每个阶段内的对象组等待其超时值，以调用该回调，因此可以在必要时启用异步关闭。默认的每阶段超时是30秒。您可以通过定义`lifecycleProcessor`上下文中命名的bean来覆盖默认的生命周期处理器实例 。如果只想修改超时，则定义以下内容即可：

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如前所述，该`LifecycleProcessor`接口还定义了用于刷新和关闭上下文的回调方法。后者驱动关闭过程，就好像`stop()`已显式调用了它一样，但是它在上下文关闭时发生。另一方面，“刷新”回调启用了`SmartLifecycle`bean的另一个功能 。刷新上下文后（在所有对象都实例化和初始化之后），该回调将被调用。此时，默认的生命周期处理器将检查每个`SmartLifecycle`对象的`isAutoStartup()`方法返回的布尔值 。如果为`true`，则在该点启动该对象，而不是等待上下文或其自身的显式调用`start()`方法（与上下文刷新不同，对于标准上下文实现，上下文启动不会自动发生）。该`phase`值与任何“依赖式”的关系确定为前面所述的启动顺序。

##### 在非Web应用程序中正常关闭Spring IoC容器

> 本节仅适用于非Web应用程序。Spring的基于Web的`ApplicationContext`实现已经有了相应的代码，可以在相关Web应用程序关闭时正常关闭Spring IoC容器。

如果您在非Web应用程序环境中（例如，在富客户端桌面环境中）使用Spring的IoC容器，请向JVM注册一个关闭钩子。这样做可以确保正常关机，并在您的Singleton bean上调用相关的destroy方法，以便释放所有资源。您仍然必须正确配置和实现这些destroy回调。

要注册关闭钩子，请调用接口`registerShutdownHook()`上声明的方法`ConfigurableApplicationContext`，如以下示例所示：

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

#### 1.6.2 ApplicationContextAware 和 BeanNameAware

当`ApplicationContext`创建创建实现该`org.springframework.context.ApplicationContextAware`接口的对象实例时，该实例将获得对该 接口的引用`ApplicationContext`。以下清单显示了`ApplicationContextAware`接口的定义：

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean可以`ApplicationContext`通过`ApplicationContext`接口或通过将引用转换为该接口的已知子类（例如`ConfigurableApplicationContext`，公开其他功能）来以编程方式操纵创建它们的bean 。一种用途是通过编程方式检索其他bean。有时，此功能很有用。但是，通常应避免使用它，因为它会将代码耦合到Spring，并且不遵循控制反转样式，在该样式中，将协作者作为属性提供给bean。提供的其他方法`ApplicationContext`提供对文件资源的访问，发布应用程序事件以及访问`MessageSource`。这些附加功能在的[附加功能中进行了介绍`ApplicationContext`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-introduction)。

自动装配是获得对的引用的另一种方法`ApplicationContext`。的_传统的_`constructor`和`byType`自动装配模式（在所描述的[自动装配协作者](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-autowire)）可以提供类型的依赖`ApplicationContext`于构造器参数或设置器方法参数，分别。要获得更大的灵活性，包括能够自动连接字段和使用多个参数方法，请使用基于注释的自动装配功能。如果您这样做，则将`ApplicationContext`自动将其连接到需要该`ApplicationContext`类型的字段，构造函数参数或方法参数中（如果相关的字段，构造函数或方法带有`@Autowired`注释）。有关更多信息，请参见[使用`@Autowired`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation)。

当`ApplicationContext`创建一个实现该`org.springframework.beans.factory.BeanNameAware`接口的类时，该类将获得对其关联对象定义中定义的名称的引用。以下清单显示了BeanNameAware接口的定义：

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

回调正常bean属性的人口之后，但在一个初始化回调诸如调用`InitializingBean`，`afterPropertiesSet`或自定义的初始化方法。

#### 1.6.3 其他`Aware`介面

除了`ApplicationContextAware`和`BeanNameAware`（前面[已经](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware)讨论[过](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware)），Spring还提供了各种各样的`Aware`回调接口，这些接口使bean可以向容器指示它们需要某种基础结构依赖性。通常，名称表示依赖项类型。下表总结了最重要的`Aware`接口：

| 名称 | 注入依赖 | 在...中解释 |
| :--- | :--- | :--- |
| `ApplicationContextAware` | 宣告`ApplicationContext`。 | [`ApplicationContextAware`和`BeanNameAware`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | 附件的事件发布者`ApplicationContext`。 | [的其他功能`ApplicationContext`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-introduction) |
| `BeanClassLoaderAware` | 类加载器，用于加载Bean类。 | [实例化豆](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-class) |
| `BeanFactoryAware` | 宣告`BeanFactory`。 | [`ApplicationContextAware`和`BeanNameAware`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `BeanNameAware` | 声明bean的名称。 | [`ApplicationContextAware`和`BeanNameAware`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `BootstrapContextAware` | `BootstrapContext`容器在其中运行的资源适配器。通常仅在支持JCA的`ApplicationContext`实例中可用。 | [JCA CCI](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/integration.html#cci) |
| `LoadTimeWeaverAware` | 定义的编织器，用于在加载时处理类定义。 | [在Spring Framework中使用AspectJ进行加载时编织](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) |
| `MessageSourceAware` | 解决消息的已配置策略（支持参数化和国际化）。 | [的其他功能`ApplicationContext`](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-introduction) |
| `NotificationPublisherAware` | Spring JMX通知发布者。 | [通知事项](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/integration.html#jmx-notifications) |
| `ResourceLoaderAware` | 配置的加载程序，用于对资源的低级别访问。 | [资源资源](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#resources) |
| `ServletConfigAware` | 当前`ServletConfig`容器在其中运行。仅在可感知网络的Spring中有效`ApplicationContext`。 | [春季MVC](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/web.html#mvc) |
| `ServletContextAware` | 当前`ServletContext`容器在其中运行。仅在可感知网络的Spring中有效`ApplicationContext`。 | [春季MVC](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/web.html#mvc) |

再次注意，使用这些接口会将您的代码与Spring API绑定在一起，并且不遵循“控制反转”样式。因此，我们建议将它们用于需要以编程方式访问容器的基础结构Bean

