# JDK 8 中的新特性



- [JDK 版本说明](https://www.oracle.com/technetwork/java/javase/jdk-relnotes-index-2162236.html?ssSourceSiteId=otncn)
- [JDK 8 版本说明](https://www.oracle.com/technetwork/java/javase/documentation/8u-relnotes-2225394.html?ssSourceSiteId=otncn)

Java Platform, Standard Edition 8 是一个拥有丰富特性的主要版本。本文档总结了 Java SE 8、JDK 8 以及 Oracle 的 Java SE 8 实现中的特性和增强。单击组件名称可获取该组件增强功能更详细的描述。

- #### [Java 编程语言](https://docs.oracle.com/javase/8/docs/technotes/guides/language/enhancements.html#javase8)

  -  Lambda 表达式是一个新的语言特性，已经在此版本中引入。该特性让您可以将功能视为方法参数，或者将代码视为数据。使用 Lambda 表达式，您可以更简洁地表示单方法接口（称为功能接口）的实例。 
  -  方法引用为已经具有名称的方法提供了易于理解的 lambda 表达式。 
  -  默认方法允许将新功能添加到库的接口中，并确保与为这些接口的旧版本编写的代码的二进制兼容性。 
  -  重复批注支持对同一个声明或类型的使用多次应用相同的批注类型。 
  -  类型批注支持在使用类型的任何地方应用批注，而不仅限于声明。与可插拔类型系统结合使用时，此特性可改进代码的类型检查。 
  -  改进类型推断。 
  -  方法参数反射。 

- #### [集合](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/changes8.html)

  -  新的`java.util.stream`包中的类提供了一个 Stream API，支持对元素流进行函数式操作。Stream API 集成在 Collections API 中，可以对集合进行批量操作，例如顺序或并行的 map-reduce 转换。 
  -  针对存在键冲突的 HashMap 的性能改进 

-  **[紧凑 profile](http://docs.oracle.com/javase/8/docs/technotes/guides/compactprofiles/)**包含 Java SE 平台的预定义子集，并且支持不需要在小型设备上部署和运行整个平台的应用。

- #### [安全性](https://docs.oracle.com/javase/8/docs/technotes/guides/security/enhancements-8.html)

  -  默认启用客户端 TLS 1.2
  -  `AccessController.doPrivileged` 的新变体支持代码断言其权限的子集，而不会阻止完全遍历堆栈来检查其他权限
  -  更强大的基于密码的加密算法 
  -  JSSE 服务器端支持 SSL/TLS 服务器名称指示 (SNI) 扩展 
  - 支持 AEAD 算法：SunJCE 提供程序得到了增强，支持 AES/GCM/NoPadding 密码实现以及  GCM 算法参数。而且 SunJSSE 提供程序也得到了增强，支持基于 AEAD 模式的密码套件。请参阅 Oracle 提供程序文档，JEP  115。 
  - 密钥库增强，包括新的域密钥库类型  `java.security.DomainLoadStoreParameter`,  和为 keytool 实用程序新增的命令选项`-importpassword`
  -  SHA-224 消息摘要 
  -  增强了对 NSA Suite B 加密的支持 
  -  更好地支持高熵随机数生成 
  -  新增了  `java.security.cert.PKIXRevocationChecker` 类，用于配置 X.509 证书的撤销检查 
  -  适用于 Windows 的 64 位 PKCS11 
  -  Kerberos 5 重放缓存中新增了 rcache 类型 
  -  支持 Kerberos 5 协议转换和受限委派 
  -  默认禁用 Kerberos 5 弱加密类型 
  -  适用于 GSS-API/Kerberos 5 机制的未绑定 SASL 
  -  针对多个主机名称的 SASL 服务 
  -  JNI 桥接至 Mac OS X 上的原生 JGSS 
  -  SunJSSE 提供程序中支持更强大的临时 DH 密钥 
  -  JSSE 中支持服务器端加密套件首选项自定义 

- #### [JavaFX](https://docs.oracle.com/javase/8/javase-clienttechnologies.htm)

  -  本版本中实施了新的 Modena 主题。有关更多信息，请参阅 [xexperience.com](https://fxexperience.com/2013/03/modena-theme-update/). 
  -  新的  `SwingNode` 类允许开发人员将 Swing 内容嵌入到 JavaFX 应用中。请参阅[SwingNode](https://docs.oracle.com/javase/8/javafx/api/javafx/embed/swing/SwingNode.html) javadoc 和 [将 Swing 内容嵌入 JavaFX 应用中。](http://docs.oracle.com/javase/8/javafx/interoperability-tutorial/embed-swing.htm). 
  -  新的 UI 控件包括 [`DatePicker`](http://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/DatePicker.html) 和  [`TreeTableView`](http://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/TreeTableView.html) 控件。 
  -  `javafx.print`  程序包为 JavaFX Printing API 提供了公共类。有关更多信息，请参阅 [javadoc](http://docs.oracle.com/javase/8/javafx/api/javafx/print/package-summary.html)
  -  3D 图形特性现在包括 3D 形状、摄像头、灯光、子场景、材料、挑选和抗锯齿。JavaFX 3D 图形库中新增了  `Shape3D` (`Box`, `Cylinder`, `MeshView`和  `Sphere`子类 ), `SubScene`, `Material`, `PickResult`, `LightBase` (`AmbientLight` 和 `PointLight` 子类) , 和 `SceneAntialiasing` API 类。此版本中的`Camera` API 类也已更新。请参阅 `javafx.scene.shape.Shape3D`, `javafx.scene.SubScene`, `javafx.scene.paint.Material`, `javafx.scene.input.PickResult`和, `javafx.scene.SceneAntialiasing`,  类的相关 javadoc 以及  [ JavaFX 3D 图形入门 ](http://docs.oracle.com/javase/8/javafx/graphics-tutorial/javafx-3d-graphics.htm) 文档。 
  -   `WebView` WebView 类包含新特性和改进。有关其他 HTML5 特性（包括 Web 套接字、Web 辅助进程和 Web 字体）的更多信息，请参阅 [ ](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/index.html)
  - ​           增强了文本支持，包括双向文本、复杂文本脚本（如泰语和印地语控件）以及文本节点中的多行多样式文本。            此版本添加了对 Hi-DPI 显示的支持。           

  - [ CSS Styleable* 类已成为公共 API。有关更多信息，请参阅 ](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/index.html)[javafx.css](http://docs.oracle.com/javase/8/javafx/api/javafx/css/package-frame.html) javadoc。 
  -  新的 [ScheduledService](http://docs.oracle.com/javase/8/javafx/api/javafx/concurrent/ScheduledService.html) 类允许自动重新启动服务。
  -  JavaFX 现在可用于 ARM 平台。适用于 ARM 的 JDK 包含 JavaFX 的基础组件、图形组件和控制组件。 

- #### [工具](http://docs.oracle.com/javase/8/docs/technotes/tools/enhancements-8.html)

  -  可通过 `jjs` 命令来调用 Nashorn 引擎。 
  -   `java` 命令用于启动 JavaFX 应用。 
  -  重新编写了 `java`  手册页。 
  -  可通过  `jdeps` 命令行工具来分析类文件。 
  -  Java Management Extensions (JMX) 支持远程访问诊断命令。 
  -  The `jarsigner`工具提供了一个选项用于请求获取时间戳机构 (TSA) 的签名时间戳。
  - Javac 工具
    - `javac` 命令的 `-parameters` 选项可用于存储正式参数名称，并启用反射 API 来检索正式参数名称。 
    -  命令现已正确实施了 Java 语言规范 (JLS) 第 15.21 节中的相等运算符的类型规则。 `javac`  
    -  The `javac`工具现在支持检查 `javadoc` 注释的内容，从而避免在运行`javadoc` 时生成的文件中产生各种问题，例如无效的 HTML 或可访问性问题。可通过新的`-Xdoclint` 选项来启用此特性。有关更多详细信息，请参阅运行“javac-X”时的输出。此特性也可以在`javac -X`". This feature is also available in the `javadoc`工具中使用，并且默认启用。 
    -   `javac` 工具现在支持根据需要生成原生标头。这样便无需在构建管道中单独运行 `javah` 工具。可以使用新的 `-h` 选项在 `javac` 中启用此特性，该选项用于指定写入头文件的目录。将为任何具有原生方法或者使用 `java.lang.annotation.Native`类型的新批注的类进行批注的常量字段生成头文件。 
  - Javadoc 工具
    -   `javadoc` 工具支持新的 `DocTree` API，让您可以将 Javadoc 注释作为抽象语法树来进行遍历。 
    -   `javadoc` 工具支持新的 Javadoc Access API，让您可以直接从 Java 应用中调用 Javadoc 工具，而无需执行新的进程。有关更多信息，请参阅[ javadoc 新特性](http://docs.oracle.com/javase/8/docs/technotes/guides/javadoc/whatsnew-8.html) 页面。 
    -   `javadoc`工具现在支持检查`javadoc` 注释的内容，从而避免在运行  `javadoc` 时生成的文件中产生各种问题，例如无效的 HTML 或可访问性问题。此特性默认为启用状态，可以通过新的`-Xdoclint` 选项加以控制。有关更多详细信息，请参阅运行 "`javadoc -X`" 时的输出。.  `javac` 工具也支持此特性，但默认情况下并未启用它。 

- #### [国际化](http://docs.oracle.com/javase/8/docs/technotes/guides/intl/enhancements.8.html)

  -  Unicode 增强，包括对 Unicode 6.2.0 的支持 
  -  采用 Unicode CLDR 数据和 java.locale.providers 系统属性 
  -  新增日历和区域设置 API 
  -  支持将自定义资源包作为扩展进行安装 

- #### [部署](http://docs.oracle.com/javase/8/docs/technotes/guides/jweb/enhancements-8.html)

  -  现在可以使用 `URLPermission`  允许沙盒小程序和 Java Web Start 应用连接回启动它们的服务器。不再授予 `SocketPermission` 。 
  -  在所有安全级别，主 JAR 文件的 JAR 文件清单中都需要 Permissions 属性。 

-  **[Date-Time 程序包 ](http://docs.oracle.com/javase/8/docs/technotes/guides/datetime/index.html)** — 一组新程序包，提供全面的日期-时间模型。 

- #### [脚本编写](http://docs.oracle.com/javase/8/docs/technotes/guides/scripting/enhancements.html#jdk8)

  -  Rhino Javascript 引擎已被替换为 [Nashorn](http://docs.oracle.com/javase/8/docs/technotes/guides/scripting/nashorn/) JavaScript 引擎 

- #### [Pack200](http://docs.oracle.com/javase/8/docs/technotes/guides/pack200/enhancements.html)

  -  Pack200 支持 JSR 292 引入的常量池条目和新字节码 
  -  JDK8 支持 JSR-292、JSR-308 和 JSR-335 指定的类文件更改 

- #### [IO 和 NIO](http://docs.oracle.com/javase/8/docs/technotes/guides/io/enhancements.html#jdk8)

  -  全新的基于 Solaris 事件端口机制的面向 Solaris 的 `SelectorProvider` 实现。要使用它，请将系统属性`java.nio.channels.spi.Selector` 的值设置为 `sun.nio.ch.EventPortSelectorProvider`. 
  -  减小 `/jre/lib/charsets.jar` 文件的大小 
  -  提高了 `java.lang.String(byte[], *)` 构造函数和 `java.lang.String.getBytes()`  方法的性能。 

- #### [java.lang 和 java.util 程序包](http://docs.oracle.com/javase/8/docs/technotes/guides/lang/enhancements.html#jdk8)

  -  并行数组排序 
  -  标准编码和解码 Base64 
  -  无符号算术支持 

- #### [JDBC](http://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/)

  -  删除了 JDBC-ODBC Bridge。 
  -  JDBC 4.2 引入了新特性。 

- #### Java DB

  -  JDK 8 包含 Java DB 10.10。 

- #### [网络](http://docs.oracle.com/javase/8/docs/technotes/guides/net/enhancements-8.0.html)

  -  已添加  `java.net.URLPermission` 类。 
  -  在 `java.net.HttpURLConnection`类中，如果安装了安全管理器，那么请求打开连接的调用需要权限。 

- #### [并发性](http://docs.oracle.com/javase/8/docs/technotes/guides/concurrency/changes8.html)

  -   `java.util.concurrent` 程序包中新增了一些类和接口。 
  -  Methods have been added to the `java.util.concurrent.ConcurrentHashMap` 类中新增了一些方法，支持基于新增流工具和 lambda 表达式的聚合操作。 
  -  `java.util.concurrent.atomic` 程序包中新增了一些类来支持可扩展、可更新的变量。 
  -   `java.util.concurrent.ForkJoinPool` 类中新增了一些方法来支持通用池。 
  -  新增的  `java.util.concurrent.locks.StampedLock` 类提供了一个基于能力的锁，可通过三种模式来控制读/写访问。

- #### [Java XML](http://docs.oracle.com/javase/8/docs/technotes/guides/xml/enhancements.html) - [JAXP](http://docs.oracle.com/javase/8/docs/technotes/guides/xml/jaxp/enhancements-8.html)

- #### [HotSpot](http://docs.oracle.com/javase/8/docs/technotes/guides/vm/)

  -  新增的硬件内部函数以便使用高级加密标准 (AES)。 

    ```undefined
    UseAES
    ```

     和 

    ```undefined
    UseAESIntrinsics
    ```

     标志用于为 硬件启用基于硬件的 AES 内部函数。硬件必须是 2010 年或更新的 Westmere 硬件。例如，要启用硬件 AES，请使用以下标志：             

     `-XX:+UseAES -XX:+UseAESIntrinsics`

    要禁用硬件 AES，请使用以下标志： `-XX:-UseAES -XX:-UseAESIntrinsics`

  -  删除了 PermGen。 

  -  方法调用的字节码指令支持 Java 编程语言中的默认方法。 

- #### [Java Mission Control 5.3 版本说明](http://www.oracle.com/technetwork/java/javase/jmc53-release-notes-2157171.html)

  -  JDK 8 包含 Java Mission Control 5.3。 