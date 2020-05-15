### 1.16 `BeanFactory` {#beans-beanfactory}

该`BeanFactory`API为Spring的IoC功能提供了基础。它的特定合同主要与Spring的其他部分以及相关的第三方框架集成在一起使用，并且其`DefaultListableBeanFactory`实现是更高级别`GenericApplicationContext`容器中的关键委托。

`BeanFactory`和相关接口（例如`BeanFactoryAware`，`InitializingBean`， `DisposableBean`）对于其他框架组件的重要结合点。通过不需要任何注释，甚至不需要反射，它们可以在容器及其组件之间进行非常有效的交互。应用程序级Bean可以使用相同的回调接口，但通常更喜欢通过注释或通过程序配置进行声明式依赖注入。

请注意，核心`BeanFactory`API级别及其`DefaultListableBeanFactory` 实现不对配置格式或要使用的任何组件注释进行假设。所有这些风味都通过扩展名（例如`XmlBeanDefinitionReader`和`AutowiredAnnotationBeanPostProcessor`）进入，并在共享`BeanDefinition`对象上作为核心元数据表示形式进行操作。这就是使Spring的容器如此灵活和可扩展的本质。

#### 1.16.1。`BeanFactory`还是`ApplicationContext`？

本节说明`BeanFactory`和 `ApplicationContext`容器级别之间的区别以及对引导的影响。

`ApplicationContext`除非有充分的理由，否则应使用an，除非将`GenericApplicationContext`其及其子类`AnnotationConfigApplicationContext` 用作自定义引导的常见实现，除非您有充分的理由 。对于所有常见目的，这些都是Spring核心容器的主要入口点：加载配置文件，触发类路径扫描，以编程方式注册Bean定义和带注释的类，以及（从5.0版本开始）注册功能性Bean定义。

因为an `ApplicationContext`包含a的所有功能`BeanFactory`，所以通常建议在平原上使用`BeanFactory`，除非需要完全控制bean处理的场景。在一个`ApplicationContext`（例如， `GenericApplicationContext`实现）中，按照约定（即，按Bean名称或Bean类型-特别是后处理器）检测到几种Bean，而普通`DefaultListableBeanFactory`的对任何特殊的Bean均不可知。

对于许多扩展的容器功能，例如注释处理和AOP代理，[`BeanPostProcessor`扩展点](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp)至关重要。如果仅使用Plain `DefaultListableBeanFactory`，则默认情况下不会检测到此类后处理器并将其激活。这种情况可能会造成混乱，因为您的bean配置实际上没有任何问题。而是在这种情况下，需要通过其他设置完全引导容器。

下表列出了`BeanFactory`and `ApplicationContext`接口和实现所提供的功能。

| 特征                                    | `BeanFactory` | `ApplicationContext` |
| :-------------------------------------- | :------------ | :------------------- |
| Bean实例化/接线                         | 是            | 是                   |
| 集成生命周期管理                        | 没有          | 是                   |
| 自动`BeanPostProcessor`注册             | 没有          | 是                   |
| 自动`BeanFactoryPostProcessor`注册      | 没有          | 是                   |
| 方便的`MessageSource`访问（用于内部化） | 没有          | 是                   |
| 内置`ApplicationEvent`发布机制          | 没有          | 是                   |

要使用显式注册Bean后处理器`DefaultListableBeanFactory`，您需要以编程方式调用`addBeanPostProcessor`，如以下示例所示：

爪哇

科特林

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

要将a `BeanFactoryPostProcessor`应用于平原`DefaultListableBeanFactory`，您需要调用其`postProcessBeanFactory`方法，如以下示例所示：

爪哇

科特林

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```

在这两种情况下，显式登记步骤是不方便的，这就是为什么在各种`ApplicationContext`变体是优选的超过一个纯 `DefaultListableBeanFactory`在弹簧支持的应用程序，依靠特别是当`BeanFactoryPostProcessor`与`BeanPostProcessor`在一个典型的企业设置实例延长容器的功能性。

|      | 一个`AnnotationConfigApplicationContext`拥有注册的所有常见的标注后处理器并通过配置注解的封面下方额外的处理器，如可能带来`@EnableTransactionManagement`。在Spring基于注释的配置模型的抽象级别上，bean后处理器的概念仅是内部容器详细信息。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

