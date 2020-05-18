# MyBatis 应用分析与最佳实践

### 课程目标

1. 了解ORM框架发展历史，了解MyBatis 特性
2. 掌握MyBatis编程式开发方法和核心对象
3. 掌握MyBatis核心配置含义
4. 掌握MyBatis的高级用法与扩展方式



注： 除了Spring JDBC 之外， 本章所有代码都在 mybatis-standalone 工程中

GitHub : xxxx



### 1. 为什么要用Mybatis

聊聊原始的 JDBC 连接数据库

假设有一张blog表

```java
// 注册 JDBC 驱动
Class.forName("com.mysql.jdbc.Driver");

```

