### 1.7 Bean定义继承 {#beans-child-bean-definitions}

Bean定义可以包含许多配置信息，包括构造函数参数，属性值和特定于容器的信息，例如初始化方法，静态工厂方法名称等。子bean定义从父定义继承配置数据。子定义可以覆盖某些值或根据需要添加其他值。使用父bean和子bean定义可以节省很多输入。实际上，这是一种模板形式。

如果您以`ApplicationContext`编程方式使用接口，则子bean定义由`ChildBeanDefinition`类表示。大多数用户不在此级别上与他们合作。相反，它们在诸如之类的类中声明性地配置Bean定义`ClassPathXmlApplicationContext`。当使用基于XML的配置元数据时，可以通过使用`parent`属性来指定子bean定义，并将父bean指定为该属性的值。以下示例显示了如何执行此操作：

```
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

> 注意该`parent`属性。

如果未指定子bean定义，则使用父定义中的bean类，但也可以覆盖它。在后一种情况下，子bean类必须与父类兼容（也就是说，它必须接受父类的属性值）。

子bean定义从父项继承范围，构造函数参数值，属性值和方法替代，并可以选择添加新值。`static`您指定的任何范围，初始化方法，destroy方法或工厂方法设置都会覆盖相应的父设置。

其余设置始终从子定义中获取：依赖项，自动装配模式，依赖项检查，单例和惰性初始化。

前面的示例通过使用`abstract`属性将父bean定义显式标记为抽象。如果父定义未指定类，请根据`abstract`需要显式标记父bean定义，如以下示例所示：

```
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

父bean不能单独实例化，因为它不完整，并且也被显式标记为`abstract`。当定义`abstract`为时，只能用作纯模板bean定义，用作子定义的父定义。尝试`abstract`通过将其称为另一个bean的ref属性来单独使用此类父bean或`getBean()`使用父bean ID 进行显式调用会返回错误。同样，容器的内部`preInstantiateSingletons()`方法将忽略定义为抽象的bean定义。

> `ApplicationContext`默认情况下预先实例化所有单例。因此，重要的是（至少对于单例bean），如果有一个（父）bean定义仅打算用作模板，并且此定义指定了一个类，则必须确保将_abstract_属性设置为_true_，否则应用程序上下文将实际（试图）预先实例化`abstract`bean。



