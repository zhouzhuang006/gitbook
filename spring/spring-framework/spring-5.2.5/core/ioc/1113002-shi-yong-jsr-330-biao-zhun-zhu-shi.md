### 1.11 使用JSR 330标准注释 {#beans-standard-annotations}

从Spring 3.0开始，Spring提供对JSR-330标准注释（依赖注入）的支持。这些注释的扫描方式与Spring注释的扫描方式相同。要使用它们，您需要在类路径中有相关的jar。

> 如果使用Maven，`javax.inject`则可以在标准Maven存储库（https://repo1.maven.org/maven2/javax/inject/javax.inject/1/）中找到该工件 。您可以将以下依赖项添加到文件pom.xml中：`    javax.inject    javax.inject    1 `

#### 1.11.1 与`@Inject`和的依赖注入`@Named`

除了`@Autowired`，您可以使用`@javax.inject.Inject`以下方法：

```java
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        // ...
    }
}
```

与一样`@Autowired`，您可以`@Inject`在字段级别，方法级别和构造函数参数级别使用。此外，您可以将注入点声明为 `Provider`，以允许按需访问范围更短的bean，或者通过`Provider.get()`调用延迟访问其他bean 。以下示例提供了前面示例的变体：

```java
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        // ...
    }
}
```

如果要为应注入的依赖项使用限定名称，则应使用`@Named`批注，如以下示例所示：

```java
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

与一样`@Autowired`，`@Inject`也可以与`java.util.Optional`或 一起使用`@Nullable`。这在这里更为适用，因为`@Inject`它没有`required`属性。以下示例展示了如何使用`@Inject`和 `@Nullable`：

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        // ...
    }
}
```



```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        // ...
    }
}
```

#### 1.11.2 `@Named`和`@ManagedBean`：`@Component`注释的标准等效项

代替`@Component`，您可以使用`@javax.inject.Named`或`javax.annotation.ManagedBean`，如以下示例所示：

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

在`@Component`不指定组件名称的情况下使用非常常见。 `@Named`可以类似的方式使用，如以下示例所示：

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

使用`@Named`或时`@ManagedBean`，可以使用与使用Spring注释完全相同的方式来使用组件扫描，如以下示例所示：

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

> 与相比`@Component`，JSR-330 `@Named`和JSR-250 `ManagedBean` 注释是不可组合的。您应该使用Spring的构造型模型来构建自定义组件注释。

#### 1.11.3 JSR-330标准注释的局限性

当使用标准注释时，您应该知道某些重要功能不可用，如下表所示：

| 弹簧                   | javax.inject。*       | javax.inject限制/注释                                        |
| :--------------------- | :-------------------- | :----------------------------------------------------------- |
| @Autowired             | @注入                 | `@Inject`没有“必填”属性。可以与Java 8一起使用`Optional`。    |
| @零件                  | @Named / @ManagedBean | JSR-330不提供可组合的模型，仅提供一种识别命名组件的方法。    |
| @Scope（“ singleton”） | @辛格尔顿             | JSR-330的默认范围类似于Spring的`prototype`。但是，为了使其与Spring的默认默认值保持一致，默认情况下，在Spring容器中声明的JSR-330 bean是a `singleton`。为了使用之外的范围`singleton`，您应该使用Spring的`@Scope`注释。`javax.inject`还提供了 [@Scope](https://download.oracle.com/javaee/6/api/javax/inject/Scope.html)批注。但是，此仅用于创建自己的注释。 |
| @Qualifier             | @Qualifier / @命名    | `javax.inject.Qualifier`只是用于构建自定义限定符的元注释。具体的`String`限定词（如`@Qualifier`带有值的Spring的限定词）可以通过关联`javax.inject.Named`。 |
| @值                    | --                    | 没有等效                                                     |
| @需要                  | --                    | 没有等效                                                     |
| @懒                    | --                    | 没有等效                                                     |
| 对象工厂               | 提供者                | `javax.inject.Provider`是Spring的直接替代方法`ObjectFactory`，只是`get()`方法名称较短。它也可以与Spring `@Autowired`或非注释构造函数和setter方法结合使用。 |

