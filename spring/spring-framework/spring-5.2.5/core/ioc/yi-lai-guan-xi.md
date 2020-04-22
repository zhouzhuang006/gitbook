## 依存关系

典型的企业应用程序不包含单个对象（或Spring的bean）。即使是最简单的应用程序，也有一些对象可以协同工作，以呈现最终用户视为一致的应用程序。下一部分将说明如何从定义多个独立的Bean定义到实现对象协作以实现目标的完全实现的应用程序。

### 依赖注入

依赖注入（DI）是一个过程，通过该过程，对象只能通过构造函数参数，工厂方法的参数或在构造或创建对象实例后在对象实例上设置的属性来定义其依赖关系（即，与它们一起工作的其他对象）。从工厂方法返回。然后，容器在创建bean时注入那些依赖项。从本质上讲，此过程是通过使用类的直接构造或服务定位器模式来控制其依赖关系的实例化或位置的Bean本身的逆过程（因此称为控制反转）。

使用DI原理，代码更加简洁，当为对象提供依赖项时，去耦会更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。结果，您的类变得更易于测试，尤其是当依赖项依赖于接口或抽象基类时，它们允许在单元测试中使用存根或模拟实现。

DI存在两个主要变体：[基于构造函数的依赖注入](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-constructor-injection)和[基于Setter的依赖注入](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-setter-injection)。

#### 基于构造函数的依赖注入

基于构造函数的DI是通过容器调用具有多个参数（每个参数代表一个依赖项）的构造函数来完成的。调用`static`带有特定参数的工厂方法来构造Bean几乎是等效的，并且本次讨论将构造函数和`static`工厂方法的参数视为类似。以下示例显示了只能通过构造函数注入进行依赖项注入的类：

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意，该类没有什么特别的。它是一个POJO，不依赖于特定于容器的接口，基类或注释。

**构造函数参数解析**

构造函数参数解析匹配通过使用参数的类型进行。如果Bean定义的构造函数参数中不存在潜在的歧义，则在实例化Bean时，在Bean定义中定义构造函数参数的顺序就是将这些参数提供给适当的构造函数的顺序。考虑以下类别：

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设`ThingTwo`和`ThingThree`类不是通过继承关联的，则不存在潜在的歧义。因此，以下配置可以正常工作，并且您无需在`<constructor-arg/>`元素中显式指定构造函数参数索引或类型。

```XML
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以发生匹配（与前面的示例一样）。当使用诸如的简单类型时 &lt;value&gt;true&lt;/value&gt;，Spring无法确定值的类型，因此在没有帮助的情况下无法按类型进行匹配。考虑以下类别：

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

**构造函数参数类型匹配**

在上述情况下，如果通过使用`type`属性显式指定构造函数参数的类型，则容器可以使用简单类型的类型匹配。如以下示例所示：

```XML
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

**构造函数参数索引**

您可以使用该`index`属性来明确指定构造函数参数的索引，如以下示例所示：

```XML
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义性之外，指定索引还可以解决歧义，其中构造函数具有两个相同类型的参数。

> 索引从0开始。

**构造函数参数名称**

您还可以使用构造函数参数名称来消除歧义，如以下示例所示：

```XML
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，要立即使用该功能，必须在启用调试标志的情况下编译代码，以便Spring可以从构造函数中查找参数名称。如果您不能或不希望使用debug标志编译代码，则可以使用[@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html)JDK注释显式命名构造函数参数。然后，样本类必须如下所示：

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#### 基于Setter的依赖注入

基于设置器的DI是通过在调用无参数构造函数或无参数`static`工厂方法以实例化您的bean之后，在您的bean上调用setter方法来完成的。下面的示例显示只能通过使用纯setter注入来依赖注入的类。此类是常规的Java。它是一个POJO，不依赖于容器特定的接口，基类或注释。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

将ApplicationContext支持基于构造函数和的Setter DI为它所管理的豆类。在已经通过构造函数方法注入了某些依赖项之后，它还支持基于setter的DI。您可以以的形式配置依赖项，将BeanDefinition其与PropertyEditor实例结合使用以将属性从一种格式转换为另一种格式。但是，大多数Spring用户并不直接（即以编程方式）使用这些类，而是使用XML bean 定义，带注释的组件（即带有@Component， @Controller等等的类）或@Bean基于Java的@Configuration类中的方法。然后将这些源在内部转换为的实例，BeanDefinition并用于加载整个Spring IoC容器实例。

> **基于构造函数或基于setter的DI？      
> **
>
> 由于可以混合使用基于构造函数的DI和基于setter的DI，因此，将构造函数用于强制性依赖项，将setter方法或配置方法用于可选性依赖项是一个很好的经验法则。请注意，可以 在setter方法上使用@Required批注，以使该属性成为必需的依赖项。但是，最好使用带有参数的程序验证的构造函数注入。
>
> Spring团队通常提倡构造函数注入，因为它使您可以将应用程序组件实现为不可变对象，并确保不存在必需的依赖项null。此外，构造函数注入的组件始终以完全初始化的状态返回到客户端（调用）代码。附带说明一下，大量的构造函数自变量是一种不好的代码味道，这意味着该类可能承担了太多的职责，应该对其进行重构以更好地解决关注点分离问题。
>
> Setter注入主要应仅用于可以在类中分配合理的默认值的可选依赖项。否则，必须在代码使用依赖项的任何地方执行非空检查。setter注入的一个好处是，setter方法可使该类的对象在以后重新配置或重新注入。因此，通过JMX MBean进行管理是用于setter注入的引人注目的用例。
>
> 使用最适合特定班级的DI风格。有时，在处理您没有源代码的第三方类时，将为您做出选择。例如，如果第三方类未公开任何setter方法，则构造函数注入可能是DI的唯一可用形式。

#### 依赖性解析过程

容器执行bean依赖项解析，如下所示：

* 使用`ApplicationContext`描述所有bean的配置元数据创建和初始化。可以通过XML，Java代码或注释指定配置元数据。

* 对于每个bean，其依赖项都以属性，构造函数参数或static-factory方法的参数的形式表示（如果使用它而不是普通的构造函数）。实际创建Bean时，会将这些依赖项提供给Bean。

* 每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。

* 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring能够以String类型提供值转换成所有内置类型，比如`int`，`long`，`String`，`boolean`，等等。

在创建容器时，Spring容器会验证每个bean的配置。但是，在实际创建Bean之前，不会设置Bean属性本身。创建容器时，将创建具有单例作用域并设置为预先实例化（默认）的Bean。范围在[Bean范围](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes)中定义。否则，仅在请求时才创建Bean。创建和分配bean的依赖关系及其依赖关系（依此类推）时，创建bean可能会导致创建一个bean图。请注意，这些依赖项之间的分辨率不匹配可能会显示得较晚-即在第一次创建受影响的bean时。

> **循环依赖**
>
> 如果主要使用构造函数注入，则可能会创建无法解决的循环依赖方案。
>
> 例如：类A通过构造函数注入需要B类的实例，而类B通过构造函数注入需要B类的实例。如果您为将类A和B相互注入而配置了bean，则Spring IoC容器会在运行时检测到此循环引用，并抛出`BeanCurrentlyInCreationException`。
>
> 一种可能的解决方案是编辑某些类的源代码，这些类将由设置者而不是构造函数来配置。或者，避免构造函数注入，而仅使用setter注入。换句话说，尽管不建议这样做，但是您可以使用setter注入配置循环依赖关系。
>
> 与典型情况（没有循环依赖关系）不同，Bean A和Bean B之间的循环依赖关系迫使其中一个Bean在完全完全初始化之前被注入另一个Bean（经典的“鸡与蛋”场景）。

通常，您可以信任Spring做正确的事。它在容器加载时检测配置问题，例如对不存在的Bean的引用和循环依赖项。在实际创建Bean时，Spring设置属性并尽可能晚地解决依赖关系。这意味着已经正确加载的Spring容器以后可以在请求对象时生成异常，如果在创建该对象或其依赖项之一时遇到问题-例如，由于缺少或无效，Bean会引发异常属性。某些配置问题的这种潜在的延迟可见性是为什么`ApplicationContext`默认情况下，实现会实例化单例bean。在实际需要这些bean之前，要花一些前期时间和内存来创建它们，您会在创建bean时发现配置问题`ApplicationContext`，而不是稍后发现。您仍然可以覆盖此默认行为，以便单例bean延迟初始化，而不是预先实例化。

如果不存在循环依赖关系，则在将一个或多个协作Bean注入从属Bean时，每个协作Bean都将被完全配置，然后再注入到从属Bean中。这意味着，如果bean A依赖于bean B，则Spring IoC容器会在对bean A调用setter方法之前完全配置beanB。换句话说，bean被实例化（如果不是预先实例化的singleton）。 ），设置其依赖项，并调用相关的生命周期方法（例如已[配置的init方法](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)或[InitializingBean回调方法](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)）。

#### 依赖注入的例子

以下示例将基于XML的配置元数据用于基于setter的DI。Spring XML配置文件的一小部分指定了一些bean定义，如下所示：

```XML
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，声明了setter以与XML文件中指定的属性匹配。以下示例使用基于构造函数的DI：

```XML
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

Bean定义中指定的构造函数参数用作的构造函数的参数`ExampleBean`。

现在考虑该示例的一个变体，在该变体中，不是使用构造函数，而是告诉Spring调用`static`工厂方法以返回对象的实例：

```XML
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

有相同的类型（尽管在此示例中是）。实例（非静态）工厂方法可以以本质上相同的方式使用（除了使用`factory-bean`属性代替`class`属性之外），因此在此不讨论这些细节。

### 依赖性和详细配置

如上[一节所述](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators)，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用或内联定义的值。Spring的基于XML的配置元数据为此目的在其`<property/>`和`<constructor-arg/>`元素中支持子元素类型。

#### 直值（原语，字符串等）

在`value`所述的属性`<property/>`元素指定属性或构造器参数的人类可读的字符串表示。Spring的[转换服务](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#core-convert-ConversionService-API)用于将这些值从转换`String`为属性或参数的实际类型。以下示例显示了设置的各种值：

```XML
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

下面的示例使用[p-namespace](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-p-namespace)进行更简洁的XML配置：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

前面的XML更简洁。但是，除非在创建bean定义时使用支持自动属性完成的IDE（例如IntelliJ IDEA或Spring Tools for Eclipse），否则错字是在运行时而不是设计时发现的。强烈建议您使用此类IDE帮助。

您还可以配置`java.util.Properties`实例，如下所示：

```XML
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring容器使用JavaBeans 机制将&lt;value/&gt;元素内的文本转换为 java.util.Properties实例PropertyEditor。这是一个不错的捷径，并且是Spring团队偏爱使用嵌套&lt;value/&gt;元素而非value属性样式的少数几个地方之一。

**该**`idref`**元素**

所述idref元件是一个简单的防错方法对通过id（一个字符串值-而不是参考）在该容器另一个bean的一个&lt;constructor-arg/&gt;或&lt;property/&gt; 元件。以下示例显示了如何使用它：

```XML
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的bean定义片段（在运行时）与下面的片段完全等效：

```XML
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，因为使用idref标记可以使容器在部署时验证所引用的命名Bean实际上是否存在。在第二个变体中，不对传递给bean targetName属性的值进行验证client。拼写错误仅在client实际实例化bean 时才发现（可能会导致致命的结果）。如果该client bean是原型 bean，则可能在部署容器后很长时间才发现此错字和所产生的异常。

> 元素 的local属性在idref4.0 bean XSD中不再受支持，因为它不再提供常规bean引用上的值。升级到4.0模式时，将现有idref local引用更改idref bean为。

其中一个共同的地方（至少在早期比Spring 2.0版本）&lt;idref/&gt;元素带来的值在配置AOP拦截在 ProxyFactoryBeanbean定义。&lt;idref/&gt;在指定拦截器名称时使用元素可防止您拼写错误的拦截器ID。

#### 对其他Bean的引用（协作者）

所述ref元件是内部的最终元件&lt;constructor-arg/&gt;或&lt;property/&gt; 定义元素。在这里，您将Bean的指定属性的值设置为对容器管理的另一个Bean（协作者）的引用。引用的bean是要设置其属性的bean的依赖关系，并且在设置属性之前根据需要对其进行初始化。（如果协作者是单例bean，则它可能已经由容器初始化了。）所有引用最终都是对另一个对象的引用。范围和验证取决于是否通过bean或parent属性指定另一个对象的ID或名称。

通过标记的bean属性指定目标bean &lt;ref/&gt;是最通用的形式，并且允许在相同容器或父容器中创建对任何bean的引用，而不管它是否在同一XML文件中。该bean属性的值 可以id与目标Bean 的属性相同，也可以与目标Bean 的属性之一相同name。以下示例显示如何使用ref元素：

```XML
<ref bean="someBean"/>
```

通过parent属性指定目标Bean 将创建对当前容器的父容器中的Bean的引用。该parent 属性的值可以id与目标Bean 的属性或目标Bean的name属性中的值之一相同。目标Bean必须位于当前容器的父容器中。主要在具有容器层次结构并且要使用与父bean名称相同的代理将现有bean封装在父容器中时，才应使用此bean参考变量。以下清单对显示了如何使用该parent属性：

```XML
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```XML
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

> 元素 的local属性在ref4.0 bean XSD中不再受支持，因为它不再提供常规bean引用上的值。升级到4.0模式时，将现有ref local引用更改ref bean为。

#### 内部Bean

甲&lt;bean/&gt;内部的元件&lt;property/&gt;或&lt;constructor-arg/&gt;元件限定内部豆，如下面的示例所示：

```XML
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义的ID或名称。如果指定，则容器不使用该值作为标识符。容器还会忽略scope创建时的标志，因为内部Bean始终是匿名的，并且始终与外部Bean一起创建。不可能独立地访问内部bean或将它们注入到协作bean中而不是封装在bean中。

作为一个特例，可以从自定义范围接收破坏回调，例如，针对单例bean中包含的请求范围内的bean。内部bean实例的创建与其包含的bean绑定在一起，但是销毁回调使它可以参与请求范围的生命周期。这不是常见的情况。内部bean通常只共享其包含bean的作用域。

#### 馆藏

的&lt;list/&gt;，&lt;set/&gt;，&lt;map/&gt;，和&lt;props/&gt;元件设置Java的属性和参数Collection类型List，Set，Map，和Properties，分别。以下示例显示了如何使用它们：

```XML
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

映射键或值的值或设置值也可以是以下任意元素：

```
bean | ref | idref | list | set | map | props | value | null
```

**集合合并**

Spring容器还支持合并集合。应用程序开发人员可以定义父&lt;list/&gt;，&lt;map/&gt;，&lt;set/&gt;或&lt;props/&gt;元素，并有孩子&lt;list/&gt;，&lt;map/&gt;，&lt;set/&gt;或&lt;props/&gt;元素继承和父集合覆盖值。也就是说，子集合的值是合并父集合和子集合的元素的结果，子集合的元素将覆盖父集合中指定的值。

关于合并的本节讨论了父子bean机制。不熟悉父bean和子bean定义的读者可能希望在继续之前阅读 相关部分。

下面的示例演示了集合合并：

```XML
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意，在bean定义的merge=true属性的&lt;props/&gt;元素 上使用了属性。当bean解析并由容器实例化时，结果实例具有一个集合，该集合包含将子项的集合与父项的集合合并的结果 。以下清单显示了结果：adminEmails child child adminEmails Properties adminEmails adminEmails

> 管理员
>
> =administrator@example.com
>
> sales=sales@example.com support=support@example.co.uk

孩子Properties集合的值设置继承父所有属性元素&lt;props/&gt;，和孩子的为值support值将覆盖父集合的价值。

这一合并行为同样适用于&lt;list/&gt;，&lt;map/&gt;和&lt;set/&gt; 集合类型。在&lt;list/&gt;元素的特定情况下，将 维护与List集合类型关联的语义（即ordered值集合的概念）。父级的值位于所有子级列表的值之前。在的情况下Map，Set和Properties集合类型，没有顺序存在。因此，没有排序的语义在背后的关联的集合类型的效果Map，Set以及Properties实现类型，容器内部使用。

**馆藏合并的局限性**

您不能合并不同的集合类型（例如Map和List）。如果尝试这样做，Exception则会抛出一个适当的值。该merge属性必须在下面的继承的子定义中指定。merge在父集合定义上指定属性是多余的，不会导致所需的合并。

**强类型集合**

随着Java 5中通用类型的引入，您可以使用强类型集合。也就是说，可以声明一个Collection类型，使其仅包含（例如）String元素。如果使用Spring将强类型依赖注入Collection到Bean中，则可以利用Spring的类型转换支持，以便在将强类型Collection 实例的元素添加到之前将其转换为适当的类型Collection。以下Java类和bean定义显示了如何执行此操作：

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```XML
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当准备注入bean 的accounts属性时，可以通过反射获得something有关强类型的元素类型的泛型信息Map&lt;String, Float&gt;。因此，Spring的类型转换基础结构将各种值元素识别为type Float，并将字符串值（9.99, 2.75和 3.99）转换为实际Float类型。

**空字符串值和空字符串值**

Spring将属性等的空参数视为empty Strings。以下基于XML的配置元数据片段将email属性设置为空 String值（“”）。

```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

前面的示例等效于以下Java代码：

```
exampleBean.setEmail("");
```

该&lt;null/&gt;元素处理null的值。以下清单显示了一个示例：

```java
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

前面的配置等效于下面的Java代码：

```java
exampleBean.setEmail(null);
```

##### 具有p-命名空间的XML快捷方式 {#beans-p-namespace}

p-namespace允许您使用bean元素的属性（而不是嵌套 &lt;property/&gt;元素）来描述协作bean的属性值，或同时使用这两者。

Spring支持基于XML Schema定义的具有名称空间的可扩展配置格式。beans本章讨论的配置格式在XML Schema文档中定义。但是，p命名空间未在XSD文件中定义，仅存在于Spring的核心中。

以下示例显示了两个XML代码段（第一个使用标准XML格式，第二个使用p-命名空间），它们可以解析为相同的结果：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

该示例显示了email在bean定义中调用的p-namespace中的属性。这告诉Spring包含一个属性声明。如前所述，p名称空间没有架构定义，因此可以将属性名称设置为属性名称。

下一个示例包括另外两个bean定义，它们都引用了另一个bean：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

此示例不仅包括使用p-namespace的属性值，还使用特殊格式声明属性引用。第一个bean定义用于&lt;property name="spouse" ref="jane"/&gt;创建从bean john到bean 的引用 jane，而第二个bean定义p:spouse-ref="jane"用作属性来执行完全相同的操作。在这种情况下，spouse属性名称是，而该-ref部分表示这不是一个直接值，而是对另一个bean的引用。

> p命名空间不如标准XML格式灵活。例如，用于声明属性引用的格式与以结尾的属性发生冲突Ref，而标准XML格式则没有。我们建议您仔细选择方法，并与团队成员进行交流，以避免同时使用这三种方法生成XML文档。

##### 具有c-namespace的XML快捷方式 {#beans-c-namespace}

与具有p-namespace的XML Shortcut相似，在Spring 3.1中引入的c-namespace允许使用内联属性来配置构造函数参数，而不是嵌套constructor-arg元素。

以下示例使用c:名称空间执行与 基于构造函数的依赖注入相同的操作：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

该c:命名空间使用相同的约定作为p:一个（尾部-ref的bean引用），供他们的名字设置构造函数的参数。同样，即使未在XSD模式中定义它（也存在于Spring内核中），也需要在XML文件中声明它。

对于极少数情况下无法使用构造函数自变量名称的情况（通常，如果字节码是在没有调试信息的情况下编译的），可以对参数索引使用后备，如下所示：

```XML
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

> 由于XML语法的原因，索引表示法要求使用前导\_，因为XML属性名称不能以数字开头（即使某些IDE允许）。相应的索引符号也可用于&lt;constructor-arg&gt;元素，但不常用，因为声明的简单顺序通常就足够了。

实际上，构造函数解析[机制](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-ctor-arguments-resolution)在匹配参数方面非常有效，因此，除非您确实需要，否则我们建议在整个配置过程中使用名称表示法。

##### 复合属性名称 {#beans-compound-property-names}

设置bean属性时，可以使用复合属性名称或嵌套属性名称，只要路径中除最终属性名称之外的所有组件都没有null。考虑以下bean定义：

```XML
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

所述something豆具有fred属性，该属性具有bob属性，其具有sammy 特性，并且最终sammy属性被设置为值123。为了使其工作，bean 的fred属性something和的bob属性fred一定不能null在bean构建之后。否则，将NullPointerException引发a。

### 使用`depends-on`

如果一个bean是另一个bean的依赖项，则通常意味着将一个bean设置为另一个bean的属性。通常，您可以使用基于XML的配置元数据中的&lt;ref/&gt; 元素来完成此操作。但是，有时bean之间的依赖性不太直接。一个示例是何时需要触发类中的静态初始值设定项，例如用于数据库驱动程序注册。该depends-on属性可以在初始化使用此元素的bean之前显式强制初始化一个或多个bean。以下示例使用该depends-on属性表示对单个bean的依赖关系：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表达对多个bean的依赖关系，请提供一个bean名称列表作为该depends-on属性的值（逗号，空格和分号是有效的分隔符）：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

> 该depends-on属性既可以指定初始化时间依赖性，也可以仅在单例 bean 的情况下指定相应的销毁时间依赖性。定义depends-on与给定bean 的关系的从属bean 首先被销毁，然后再销毁给定bean本身。这样，depends-on还可以控制关​​机顺序。

### 懒加载初始化Bean

默认情况下，ApplicationContext实现会在初始化过程中积极创建和配置所有 单例 bean。通常，这种预初始化是可取的，因为与数小时甚至数天后相比，会立即发现配置或周​​围环境中的错误。如果不希望这种行为，则可以通过将bean定义标记为延迟初始化来防止单例bean的预实例化。延迟初始化的bean告诉IoC容器在首次请求时（而不是在启动时）创建一个bean实例。

在XML中，此行为由 元素lazy-init上的属性控制&lt;bean/&gt;，如以下示例所示：

```XML
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当上述配置是通过消耗ApplicationContext，所述lazy豆是不是提前被实例化时ApplicationContext开始，而not.lazy豆急切预实例化。

但是，当延迟初始化的bean是未进行延迟初始化的单例bean的依赖项ApplicationContext时，由于在启动时必须满足单例的依赖关系，所以Bean在启动时会创建延迟初始化的bean。延迟初始化的bean被注入到其他未延迟初始化的单例bean中。

您还可以通过使用元素default-lazy-init上的属性在容器级别上控制延迟初始化， &lt;beans/&gt;以下示例显示：

```XML
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 自动装配协作器

Spring容器可以自动装配协作bean之间的关系。您可以通过检查的内容，让Spring自动为您的bean解决协作者（其他bean）ApplicationContext。自动装配具有以下优点：

自动装配可以大大减少指定属性或构造函数参数的需要。（在本章其他地方讨论的其他机制，例如Bean模板 ，在这方面也很有价值。）

随着对象的发展，自动装配可以更新配置。例如，如果您需要向类中添加依赖项，则无需修改配置即可自动满足该依赖项。因此，自动装配在开发过程中特别有用，而不必担心在代码库变得更稳定时切换到显式接线的选择。

使用基于XML的配置元数据时（请参阅Dependency Injection），您可以使用元素的autowire属性为 bean定义指定自动装配模式&lt;bean/&gt;。自动装配功能具有四种模式。您可以为每个bean指定自动装配，因此可以选择要自动装配的装配。下表描述了四种自动装配模式：

**表2.自动装配模式**

| 模式 | 说明 |
| :--- | :--- |
| `no` | （默认）无自动装配。Bean引用必须由`ref`元素定义。对于较大的部署，建议不要更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。在某种程度上，它记录了系统的结构。 |
| `byName` | 按属性名称自动布线。Spring寻找与需要自动装配的属性同名的bean。例如，如果一个bean定义被设置为按名称自动装配并且包含一个`master`属性（即它具有一个`setMaster(..)`方法），那么Spring将查找一个名为的bean定义，`master`并使用它来设置该属性。 |
| `byType` | 如果容器中恰好存在一个属性类型的bean，则使该属性自动连接。如果存在多个错误，则会引发致命异常，这表明您可能无法`byType`对该bean使用自动装配。如果没有匹配的bean，则什么都不会发生（未设置该属性）。 |
| `constructor` | 类似于`byType`但适用于构造函数参数。如果容器中不存在构造函数参数类型的一个bean，则将引发致命错误。 |

使用byType或constructor自动装配模式，您可以连接阵列和键入的集合。在这种情况下，将提供容器中与期望类型匹配的所有自动装配候选，以满足相关性。Map如果期望的密钥类型为，则可以自动连接强类型实例String。自动装配Map 实例的值包括与期望类型匹配的所有bean实例，并且 Map实例的键包含相应的bean名称。

#### 自动接线的局限性和缺点

当在项目中一致使用自动装配时，自动装配效果最佳。如果通常不使用自动装配，那么使用开发人员仅连接一个或两个bean定义可能会使开发人员感到困惑。

考虑自动装配的局限性和缺点：

* 显式依赖项property和constructor-arg设置始终会覆盖自动装配。您无法自动装配简单的属性，例如基元 Strings，和Classes（以及此类简单属性的数组）。此限制是设计使然。

* 自动装配不如显式接线精确。尽管如前所述，Spring还是谨慎地避免在可能产生意外结果的模棱两可的情况下进行猜测。Spring管理的对象之间的关系不再明确记录。

* 接线信息可能不适用于可能从Spring容器生成文档的工具。

* 容器中的多个bean定义可能与要自动装配的setter方法或构造函数参数指定的类型匹配。对于数组，集合或 Map实例，这不一定是问题。但是，对于期望单个值的依赖项，不会任意解决此歧义。如果没有唯一的bean定义可用，则引发异常。

在后一种情况下，您有几种选择：

* 放弃自动布线，转而使用明确的布线。

* 如下一节所述，通过将其autowire-candidate属性设置为来避免自动装配bean定义。false

* 通过将单个bean定义primary的&lt;bean/&gt;元素属性设置为，将其指定为主要候选对象 true。

* 如基于注释的容器配置中所述，通过基于注释的配置实现更细粒度的控件。

#### 从自动装配中排除Bean

在每个bean的基础上，您可以从自动装配中排除一个bean。使用Spring的XML格式，将元素的autowire-candidate属性设置&lt;bean/&gt;为false。容器使特定的bean定义对自动装配基础结构不可用（包括注释样式配置，例如@Autowired）。

> 该autowire-candidate属性旨在仅影响基于类型的自动装配。它不会影响按名称显示的显式引用，即使未将指定的Bean标记为自动装配候选，该名称也可得到解析。结果，如果名称匹配，按名称自动装配仍然会注入一个bean。

您还可以基于与Bean名称的模式匹配来限制自动装配候选。顶级&lt;beans/&gt;元素在其default-autowire-candidates属性内接受一个或多个模式 。例如，要将自动装配候选状态限制为名称以结尾的任何bean Repository，请提供值\*Repository。要提供多种模式，请在以逗号分隔的列表中定义它们。true或falseBean定义autowire-candidate属性的显式值 始终优先。对于此类bean，模式匹配规则不适用。

这些技术对于您不希望通过自动装配将其注入到其他bean中的bean非常有用。这并不意味着排除的bean本身不能使用自动装配进行配置。相反，bean本身不是自动装配其他bean的候选对象。

#### 1.4.6 方法注入

在大多数应用场景中，容器中的大多数bean是[singletons](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-singleton)。当单例Bean需要与另一个单例Bean协作或非单例Bean需要与另一个非单例Bean协作时，通常可以通过将一个Bean定义为另一个Bean的属性来处理依赖性。当bean的生命周期不同时会出现问题。假设单例bean A可能需要使用非单例（原型）bean B，也许是在A的每个方法调用上使用。容器仅创建一次单例bean A，因此只有一次机会来设置属性。每次需要一个容器时，容器都无法为bean A提供一个新的bean B实例。

一个解决方案是放弃某些控制反转。您可以通过实现接口，并通过每次[容器](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-client)A都需要[容器](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-client)B[的](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-client)[调用来](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-client)请求（通常是新的）bean B实例，来[使bean A知道容器](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-aware)。以下示例显示了此方法：`ApplicationContextAware`[`getBean("B")`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-client)

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

前面的内容是不理想的，因为业务代码知道并耦合到Spring框架。方法注入是Spring IoC容器的一项高级功能，使您可以干净地处理此用例。

您可以在[此博客条目中](https://spring.io/blog/2004/08/06/method-injection/)了解有关方法注入动机的更多信息 。

##### 查找方法注入

查找方法注入是容器覆盖容器管理的Bean上的方法并返回容器中另一个命名Bean的查找结果的能力。查找通常涉及原型bean，如上[一节中所述](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection)。Spring框架通过使用从CGLIB库生成字节码来动态生成覆盖该方法的子类来实现此方法注入。

> 为了使此动态子类起作用，Spring Bean容器子类的类也不能为\`final\`，要覆盖的方法也不能为\`final\`。对具有\`abstract\`方法的类进行单元测试需要您自己对该类进行子类化，并提供该\`abstract\`方法的存根实现。组件扫描也需要具体的方法，这需要具体的类。另一个关键限制是，查找方法不适用于工厂方法，尤其不适\`@Bean\`用于配置类中的方法，因为在这种情况下，容器不负责创建实例，因此无法在其上创建运行时生成的子类。苍蝇。



对于CommandManager前面的代码片段中的类，Spring容器动态地覆盖该createCommand\(\) 方法的实现。该CommandManager班没有任何Spring的依赖，因为返工例所示：

```java
package fiona.apple;

// no more Spring imports!

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

在包含要注入的方法的客户端类（CommandManager在本例中为）中，要注入的方法需要以下形式的签名：

```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法为abstract，则动态生成的子类将实现该方法。否则，动态生成的子类将覆盖原始类中定义的具体方法。考虑以下示例：

```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

只要需要新的bean 实例，被标识为的bean 就会commandManager调用其自己的createCommand\(\)方法myCommand。myCommand如果确实需要，您必须小心地将bean 部署为原型。如果是单例，myCommand 则每次都返回Bean 的相同实例。



另外，在基于注释的组件模型中，您可以通过@Lookup注释声明一个查找方法，如以下示例所示：

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更习惯地说，您可以依赖于针对查找方法的声明返回类型解析的目标bean：

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

请注意，通常应使用具体的存根实现声明此类带注释的查找方法，以使其与默认情况下忽略抽象类的Spring组件扫描规则兼容。

此限制不适用于显式注册或显式导入的Bean类。

> 访问范围不同的目标bean的另一种方法是ObjectFactory/ Provider注入点。请参阅将范围Bean作为依赖项。
>
> 您可能还会发现ServiceLocatorFactoryBean（在 org.springframework.beans.factory.config包装中）有用。

#### 任意方法替换

与查找方法注入相比，方法注入的一种不太有用的形式是能够用另一种方法实现替换托管bean中的任意方法。您可以放心地跳过本节的其余部分，直到您真正需要此功能为止。

借助基于XML的配置元数据，您可以使用replaced-method元素将现有方法实现替换为已部署bean的另一个实现。考虑以下类，该类具有一个computeValue我们要重写的方法：

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

实现该`org.springframework.beans.factory.support.MethodReplacer`接口的类提供了新的方法定义，如以下示例所示：

```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

用于部署原始类并指定方法重写的Bean定义类似于以下示例：

```XML
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

您可以&lt;arg-type/&gt;在元素内使用一个或多个元素&lt;replaced-method/&gt; 来指示要覆盖的方法的方法签名。仅当方法重载且类中存在多个变体时，才需要对参数签名。为了方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。例如，以下所有匹配项 java.lang.String：

```
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的选择，所以通过让您仅键入与参数类型匹配的最短字符串，此快捷方式可以节省很多输入。













