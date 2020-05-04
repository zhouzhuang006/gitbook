# Spring框架概述

版本5.2.6.RELEASE

Spring使创建Java企业应用程序变得容易。它提供了在企业环境中使用Java语言所需的一切，并支持Groovy和Kotlin作为JVM上的替代语言，并且可以根据应用程序的需求灵活地创建多种体系结构。从Spring Framework 5.1开始，Spring需要JDK 8+（Java SE 8+），并提供对JDK 11 LTS的现成支持。建议将Java SE 8更新60作为Java 8的最低修补程序版本，但通常建议使用最新的修补程序版本。

Spring支持广泛的应用场景。在大型企业中，应用程序通常存在很长时间，并且必须在JDK和应用程序服务器上运行，这些应用程序的升级周期不受开发人员控制。其他服务器则可以作为单个jar运行，并且服务器可以嵌入云环境中。还有一些可能是不需要服务器的独立应用程序（例如批处理或集成工作负载）。

Spring是开源的。它拥有一个庞大而活跃的社区，可以根据各种实际用例提供持续的反馈。这有助于Spring在很长一段时间内成功地发展。

## 1.“春天”是什么意思 {#overview-spring}

术语“春天”在不同的上下文中表示不同的事物。它可以用来指代Spring Framework项目本身，而这一切都是从这里开始的。随着时间的流逝，其他Spring项目已经建立在Spring Framework之上。通常，当人们说“春天”时，它们表示整个项目系列。本参考文档重点关注基础：Spring框架本身。

Spring框架分为多个模块。应用程序可以选择所需的模块。核心容器的模块是核心，包括配置模型和依赖项注入机制。除此之外，Spring框架还为不同的应用程序体系结构提供了基础支持，包括消息传递，事务性数据和持久性以及Web。它还包括基于Servlet的Spring MVC Web框架，以及并行的Spring WebFlux反应式Web框架。

关于模块的注释：Spring的框架jar允许部署到JDK 9的模块路径（“拼图”）。为了在支持Jigsaw的应用程序中使用，Spring Framework 5 jar附带了“自动模块名称”清单条目，这些清单条目定义了与jar工件无关的稳定语言级别的模块名称（“ spring.core”，“ spring.context”等）。名称（罐子遵循相同的命名模式，用“-”代替“。”，例如“ spring-core”和“ spring-context”）。当然，Spring的框架jar可以在JDK 8和9+的类路径上正常工作。

## 2. Spring的历史和Spring框架 {#overview-history}

响应于早期[J2EE](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition)规范的复杂性，Spring于2003年应运而生。尽管有些人认为Java EE和Spring竞争，但事实上Spring是Java EE的补充。Spring编程模型不包含Java EE平台规范。相反，它与EE伞中精心选择的各个规范集成在一起：

* Servlet API（[JSR 340](https://jcp.org/en/jsr/detail?id=340)）

* WebSocket API（[JSR 356](https://www.jcp.org/en/jsr/detail?id=356)）

* 并发实用程序（[JSR 236](https://www.jcp.org/en/jsr/detail?id=236)）

* JSON绑定API（[JSR 367](https://jcp.org/en/jsr/detail?id=367)）

* Bean验证（[JSR 303](https://jcp.org/en/jsr/detail?id=303)）

* JPA（[JSR 338](https://jcp.org/en/jsr/detail?id=338)）

* JMS（[JSR 914](https://jcp.org/en/jsr/detail?id=914)）

* 以及必要时用于事务协调的JTA / JCA设置。

Spring框架还支持依赖注入（[JSR 330](https://www.jcp.org/en/jsr/detail?id=330)）和通用注释（[JSR 250](https://jcp.org/en/jsr/detail?id=250)）规范，应用程序开发人员可以选择使用这些规范来代替Spring框架提供的特定于Spring的机制。

从Spring Framework 5.0开始，Spring至少需要Java EE 7级别（例如Servlet 3.1 +，JPA 2.1+）-同时提供与Java EE 8级别的较新API的现成集成。 （例如Servlet 4.0，JSON绑定API）在运行时遇到。这使Spring与Tomcat 8和9，WebSphere 9和JBoss EAP 7完全兼容。

随着时间的流逝，Java EE在应用程序开发中的作用已经演变。在Java EE和Spring的早期，创建了应用程序以将其部署到应用程序服务器。如今，借助Spring Boot，可以以对开发人员和云友好的方式创建应用程序，并嵌入Servlet容器并对其进行微不足道的更改。从Spring Framework 5开始，WebFlux应用程序甚至不直接使用Servlet API，并且可以在非Servlet容器的服务器（例如Netty）上运行。

春天继续创新和发展。除了Spring Framework，还有其他项目，例如Spring Boot，Spring Security，Spring Data，Spring Cloud，Spring Batch等。重要的是要记住，每个项目都有其自己的源代码存储库，问题跟踪程序和发布节奏。有关Spring项目的完整列表，请参见[spring.io/projects](https://spring.io/projects)。

## 3.设计哲学 {#overview-philosophy}

当您了解框架时，不仅要了解框架的工作而且要遵循的原则很重要。以下是Spring框架的指导原则：

* 提供各个级别的选择。Spring使您可以尽可能推迟设计决策。例如，您可以在不更改代码的情况下通过配置切换持久性提供程序。对于许多其他基础架构问题以及与第三方API的集成也是如此。

* 适应不同的观点。Spring拥有灵活性，并且对如何完成事情一无所知。它从不同的角度支持广泛的应用程序需求。

* 保持强大的向后兼容性。对Spring的发展进行了精心管理，以使各个版本之间几乎没有重大更改。Spring支持精心选择的JDK版本和第三方库，以方便维护依赖于Spring的应用程序和库。

* 关心API设计。Spring团队投入了大量的思想和时间来制作直观，并在许多版本和很多年中都适用的API。

* 为代码质量设置高标准。Spring框架非常强调有意义，最新和准确的javadoc。它是极少数可以声明干净代码结构且程序包之间没有循环依赖关系的项目之一。

## 4.反馈和贡献 {#overview-feedback}

对于操作方法问题或诊断或调试问题，我们建议使用StackOverflow，并且有一个[问题页面](https://spring.io/questions)列出了要使用的建议标签。如果您相当确定Spring框架中有问题或想提出功能，请使用[GitHub Issues](https://github.com/spring-projects/spring-framework/issues)。

如果您有解决方案或建议的解决方案，可以在[Github](https://github.com/spring-projects/spring-framework)上提交拉取请求。但是，请记住，对于除最琐碎的问题以外的所有问题，我们都希望在问题跟踪器中记录故障单，在该跟踪器中进行讨论并保留记录以备将来参考。

有关更多详细信息，请参见[CONTRIBUTING](https://github.com/spring-projects/spring-framework/blob/master/CONTRIBUTING.md)顶级项目页面上的准则。

## 5.入门 {#overview-getting-started}

如果您刚刚开始使用Spring，则可能需要通过创建基于[Spring Boot](https://projects.spring.io/spring-boot/)的应用程序来开始使用Spring Framework。Spring Boot提供了一种快速（且自以为是）的方法来创建可用于生产环境的基于Spring的应用程序。它基于Spring框架，相对于配置更倾向于约定俗成，旨在使您尽快启动并运行。

您可以使用[start.spring.io](https://start.spring.io/)生成一个基本项目，或遵循[“入门”指南之一](https://spring.io/guides)，例如“[入门RESTful Web Service的构建”](https://spring.io/guides/gs/rest-service/)。这些指南不仅易于理解，而且非常注重任务，并且大多数基于Spring Boot。它们还涵盖了Spring产品组合中的其他项目，您在解决特定问题时可能要考虑这些项目。

