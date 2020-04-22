## IoC容器 {#beans}

本章介绍了Spring的控制反转（IoC）容器。

### Spring IoC容器和Bean简介 {#beans-introduction}

本章介绍了控制反转（IoC）原理的Spring框架实现。IoC也称为依赖注入（DI）。在此过程中，对象仅通过构造函数参数，工厂方法的参数或在构造或从工厂方法返回后在对象实例上设置的属性来定义其依赖项（即，与它们一起使用的其他对象） 。然后，容器在创建bean时注入那些依赖项。此过程从根本上讲是通过使用类的直接构造或诸如服务定位器模式之类的机制来控制其依赖项的实例化或位置的bean本身的逆过程（因此称为 Inversion of Control）。

在`org.springframework.beans`和`org.springframework.context`包是Spring框架的IoC容器的基础。[`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口提供了一种高级配置机制，能够管理任何类型的对象。[`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)是的子接口`BeanFactory`。它增加了如下功能：

* 与Spring的AOP功能轻松集成

* 消息资源处理（用于国际化）

* 事件发布

* 应用层特定的上下文，例如`WebApplicationContext`用于Web应用程序中。

简而言之，`BeanFactory`提供了配置框架和基本功能，并且`ApplicationContext`增加了更多针对企业的功能。`ApplicationContext`是对一个完整的覆盖了`BeanFactory`，在Spring的IoC容器的描述本章重点使用。有关`BeanFactory`使用的详细信息，而不是`ApplicationContext, `请看[`BeanFactory`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-beanfactory)。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。Bean是由Spring IoC容器实例化，组装和以其他方式管理的对象。否则，bean仅仅是应用程序中许多对象之一。Bean及其之间的依赖关系反映在容器使用的配置元数据中。

