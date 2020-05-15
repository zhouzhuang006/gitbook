### Bean总览 {#beans-definition}

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的（例如，以XML`<bean/>`定义的形式）。

在容器本身内，这些bean定义表示为`BeanDefinition`对象，这些对象包含（除其他信息外）以下元数据：

* 包限定的类名：通常，定义了Bean的实际实现类。

* Bean行为配置元素，用于声明Bean在容器中的行为（作用域，生命周期回调等）。

* 引用其他bean完成其工作所需的bean。这些引用也称为协作者或依赖项。

* 要在新创建的对象中设置的其他配置设置-例如，池的大小限制或要在管理连接池的bean中使用的连接数。

此元数据转换为构成每个bean定义的一组属性。下表描述了这些属性：

**表1. bean定义**

| 属性 | 解释 |
| :--- | :--- |
| Class | [实例化](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-class)Bean |
| Name | [命名](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-beanname) |
| Scope | [Bean范围](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes) |
| Constructor arguments | [依赖注入](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators) |
| Properties | [依赖注入](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators) |
| Autowiring mode | [自动装配协作器](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-autowire) |
| Lazy initialization mode | [懒加载初始化B](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-lazy-init)ean |
| Initalization method | [初始化回调](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method | [销毁回调](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) |

除了包含有关如何创建特定bean的信息的bean定义之外，`ApplicationContext`实现还允许注册在容器外部（由用户）创建的现有对象。这是通过通过方法访问ApplicationContext的BeanFactory来完成的`getBeanFactory()`，该方法返回BeanFactory

`DefaultListableBeanFactory`实现。`DefaultListableBeanFactory`通过`registerSingleton(..)`和`registerBeanDefinition(..)`

方法支持此注册。但是，典型的应用程序只能与通过常规bean定义元数据定义的bean一起使用。

> Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他自省步骤中正确地推理它们。虽然在某种程度上支持覆盖现有的元数据和现有的单例实例，但是在运行时（与对工厂的实时访问同时）对新bean的注册未得到正式支持，并且可能导致并发访问异常，bean容器中的状态不一致或都。

#### Bean命名 {#beans-beanname}

每个bean具有一个或多个标识符。这些标识符在承载Bean的容器内必须唯一。一个bean通常只有一个标识符。但是，如果需要多个，则可以将多余的别名视为别名。

在基于XML配置文件，您可以使用`id`属性，该`name`属性，或两者来指定bean标识符。该`id`属性使您可以精确指定一个ID。按照惯例，这些名称是字母数字（“ myBean”，“ someService”等），但它们也可以包含特殊字符。如果要为bean引入其他别名，还可以在`name`属性中指定它们，并用逗号（`,`），分号（`;`）或空格分隔。作为历史记录，在Spring 3.1之前的版本中，该`id`属性被定义为一种`xsd:ID`类型，该类型限制了可能的字符。从3.1开始，它被定义为`xsd:string`类型。请注意，Bean`id`唯一性仍由容器强制执行，尽管XML解析器不再如此。

您可以不为bean提供`name`或`id`。如果不提供`name`或`id`显式提供，容器将为该bean生成一个唯一的名称。但是，如果要通过名称引用该bean，则必须通过使用`ref`元素或服务定位器样式查找来提供名称。不提供名称的相关的使用[内部bean](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-inner-beans)和[自动装配有关](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-autowire)。

> Bean命名约定
>
> 约定是在命名bean时将标准Java约定用于实例字段名称。也就是说，bean名称以小写字母开头，并从那里用驼峰式大小写。这样的名字的例子包括accountManager，accountService，userDao，loginController，等等。
>
> 一致地命名Bean使您的​​配置更易于阅读和理解。另外，如果您使用Spring AOP，则在将建议应用于名称相关的一组bean时，它会很有帮助。
>
> 通过在类路径中进行组件扫描，Spring会按照前面描述的规则为未命名的组件生成Bean名称：本质上，采用简单的类名称并将其初始字符转换为小写。但是，在（不寻常的）特殊情况下，如果有多个字符并且第一个和第二个字符均为大写字母，则会保留原始大小写。这些规则与java.beans.Introspector.decapitalize（由Spring在此处使用）定义的规则相同。

##### 在Bean定义之外别名Bean {#beans-beanname-alias}

在bean定义本身中，可以通过使用由`id`属性指定的最多一个名称和属性中任意数量的其他名称的组合来为bean提供多个名称`name`。这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如，通过使用特定于该组件本身的bean名称，让应用程序中的每个组件都引用一个公共依赖项。

但是，在实际定义bean的地方指定所有别名并不总是足够的。有时需要为在别处定义的bean引入别名。在大型系统中通常是这种情况，在大型系统中，配置是在每个子系统之间分配的，每个子系统都有自己的对象定义集。在基于XML的配置元数据中，您可以使用`<alias/>`元素来完成此任务。以下示例显示了如何执行此操作：

```XML
<alias name="fromName" alias="toName"/>
```

在这种情况下，`fromName`在使用该别名定义之后，命名为Bean（位于同一容器中）的豆也可以称为`toName`。

例如，子系统A的配置元数据可以使用名称来引用数据源`subsystemA-dataSource`。子系统B的配置元数据可以使用的名称引用数据源`subsystemB-dataSource`。组成使用这两个子系统的主应用程序时，主应用程序使用名称引用数据源`myApp-dataSource`。要使所有三个名称都引用同一个对象，可以将以下别名定义添加到配置元数据中：

```XML
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在，每个组件和主应用程序都可以通过唯一的名称引用数据源，并保证不与任何其他定义冲突（有效地创建名称空间），但它们引用的是同一bean。

> Java配置
>
> 如果使用Java配置，则`@Bean`注释可用于提供别名。有关详细信息，请参见[使用`@Bean`注释](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-java-bean-annotation)。

#### 实例化Bean {#beans-factory-class}

Bean定义实质上是创建一个或多个对象的方法。当被询问时，容器将查看命名bean的配方，并使用该bean定义封装的配置元数据来创建（或获取）实际对象。

如果使用基于XML的配置元数据，则`class`在`<bean/>`元素的属性中指定要实例化的对象的类型（或类）。此`class`属性（在内部是实例的`Class`属性`BeanDefinition`）通常是必需的。（有关异常，请参阅[使用实例工厂方法实例化](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-class-instance-factory-method)和[Bean定义继承](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions)。）可以通过以下`Class`两种方式之一使用该属性：

* 通常，在容器本身通过反射性地调用其构造函数直接创建Bean的情况下，指定要构造的Bean类，这在某种程度上等同于使用`new`运算符的Java代码。

* 要指定包含`static`要创建对象而调用的工厂方法的实际类，在不太常见的情况下，容器将`static`在类上调用工厂方法来创建Bean。从`static`工厂方法的调用返回的对象类型可以是相同的类，也可以是完全不同的另一类。

> 内部Class名称
>
> 如果要为`static`嵌套类配置Bean定义，则必须使用嵌套类的二进制名称。
>
> 例如，如果`SomeThing`在`com.example`包中有一个名为的类，并且`SomeThing`该类有一个`static`名为的嵌套类`OtherThing`，则`class`bean定义上的属性值将为`com.example.SomeThing$OtherThing`。
>
> 请注意，名称中使用了`$`字符，以将嵌套的类名与外部类名分开。

##### 用构造函数实例化 {#beans-factory-class-ctor}

当通过构造函数方法创建bean时，所有普通类都可以被Spring使用并与之兼容。也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码。只需指定bean类就足够了。但是，根据您用于该特定bean的IoC的类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您要管理的任何类。它不仅限于管理真正的JavaBean。大多数Spring用户更喜欢实际的JavaBean，它们仅具有默认（无参数）构造函数，并具有根据容器中的属性建模的适当的setter和getter。您还可以在容器中具有更多奇特的非bean样式类。例如，如果您需要使用绝对不符合JavaBean规范的旧式连接池，则Spring也可以对其进行管理。

使用基于XML的配置元数据，您可以如下指定bean类：

```XML
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

有关向构造函数提供参数（如果需要）并在构造对象之后设置对象实例属性的机制的详细信息，请参见[注入依赖项](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators)。

##### 用静态工厂方法实例化 {#beans-factory-class-static-factory-method}

在定义使用静态工厂方法创建的bean时，请使用`class`属性指定包含`static`工厂方法的类，并使用命名`factory-method`为属性的属性来指定工厂方法本身的名称。您应该能够调用此方法（使用可选参数，如稍后所述）并返回一个活动对象，该对象随后将被视为已通过构造函数创建。这种bean定义的一种用法是`static`用旧版代码调用工厂。

以下bean定义指定通过调用工厂方法来创建bean。该定义不指定返回对象的类型（类），而仅指定包含工厂方法的类。在此示例中，该`createInstance()`方法必须是静态方法。以下示例显示如何指定工厂方法：

```XML
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了一个可与前面的bean定义一起使用的类：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关为工厂方法提供（可选）参数并在从工厂返回对象之后设置对象实例属性的机制的详细信息，请参见[Dependencies and Configuration in Detail](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-properties-detailed)。

##### 使用实例工厂方法实例化 {#beans-factory-class-instance-factory-method}

类似于通过[静态工厂方法进行](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-class-static-factory-method)实例化，使用实例工厂方法进行实例化会从容器中调用现有bean的非静态方法来创建新bean。要使用此机制，请将`class`属性保留为空，并在`factory-bean`属性中指定当前（或父或祖先）容器中的Bean名称，该容器包含将被调用以创建对象的实例方法。使用`factory-method`属性设置工厂方法本身的名称。以下示例显示了如何配置此类Bean：

```XML
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

以下示例显示了相应的类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含一个以上的工厂方法，如以下示例所示：

```XML
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

以下示例显示了相应的类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明，工厂Bean本身可以通过依赖项注入（DI）进行管理和配置。[详细信息，](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-properties-detailed)请参见[依赖性和配置](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-properties-detailed)。

> 在Spring文档中，“ factory bean”是指在Spring容器中配置并通过实例或 静态工厂方法创建对象的bean 。相反， FactoryBean（请注意，大写字母）是指特定于Spring的 FactoryBean 。



