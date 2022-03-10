[TOC]

# Spring

## Part 1. 概述

### 一、Spring 是什么

> Spring是Java EE编程领域的一个全栈轻量级开源框架，核心是 IoC 和 AOP。

### 二、Spring 的优势

1. 方便解耦，简化开发
2. 支持AOP
3. 支持事务管理
4. 方便测试
5. 支持集成其他框架
6. 通过学习源码提升Java技术

### 三、Spring 的体系结构

![Spring体系机构](https://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/images/spring-overview.png)

## Part 2. IoC

### 一、IoC 的概念

#### (一) 耦合与解耦

##### (1) 耦合

即程序间的依赖关系。高耦合意味着高依赖，即程序模块之间的关系复杂性高，独立性差，维护成本就越高。

```java
public class AccountServiceImpl implements IAccountService { 
    private IAccountDao accountDao = new AccountDaoImpl();
} /* 此时Service层依赖DAO层，如果此时没有DAO实现类，将编译失败。独立性差。*/
```

##### (2) 解耦

早期 JDBC 加载驱动的时候可以使用 `DriverManager.registerDriver(new com.mysql.jdbc.Driver());` 来实现。但是依赖了 MySQL 的 jdbc 驱动类，如果这时候更换了数据库品牌(比如 Oracle)，需要修改源码来重新加载数据库驱动。这显然不是我们想要的。但是改成 `Class.forName("com.mysql.jdbc.Driver");`，就可以通过反射来加载驱动形成解耦，修改字符串就可以更换数据库，更加方便。

解耦的思路:

1. 使用反射来创建对象，而避免使用new关键字

2. 通过读取配置文件来获取要创建的对象的全限定类名

#### (二) 工厂模式解耦

我们知道，在三层架构中 **表现层调用业务层，业务层调用持久层** 。我们可以使用一个工厂类来管理创建 Service 和 DAO 对象，上层调用下层的时候直接叫工厂返回所需要的下层对象。

- 需要一个配置文件来配置保存我们的 service 和 dao。每个对象拥有：[唯一标识符-全限定类名]

- 读取配置文件内容，反射创建对象

#### (三) 控制反转 - IoC

##### (1) IoC 容器

由于我们是很多对象，肯定要找个集合来存。这时候有 Map 和 List 供选择。

到底选 Map 还是 List 就看我们有没有查找需求。有查找需求，选 Map。

所以我们把这个 map 称之为容器。

##### (2) 控制反转

原先我们在获取对象时，都是采用 new 的方式。是主动的。

现在我们获取对象时，去跟工厂要，有工厂为我们查找或者创建对象。是被动的。

这种被动接收的方式获取对象的思想就是 **控制反转**，它是 spring 框架的核心之一。

### 二、Spring 中基于 XML 的IoC 配置

#### (一) 配置使用 IoC

##### (1) 下载必备 jar 包

- 核心容器四个包

  - spring-beans-*.jar

    Beans 模块是所有应用都要用到的，它包含访问配置文件、创建和管理 bean 以及进行Inversion of Control / Depen-dency Injection（IoC/DI）操作相关的所有类。


  - spring-context.*.jar

    Spring 的上下文即 IoC 容器，通过上下文可以获得容器中的 Bean。 ApplicationContext 接口是 Context 模块的关键。 Context 模块构建于 Core 和 Beans 模块基础之上，提供了一种类似于 JNDI 注册器的框架式的对象访问方法。


  - spring-core.*.jar

    Core 模块主要包含 Spring 框架基本的核心工具类，Spring 的其他组件要都要使用到这个包里的类，Core 模块是其他组件的基本核心。当然你也可以在自己的应用系统中使用这些工具类。


  - spring-expression.*.jar (SpEl)

    Expression Language 模块提供了一个强大的表达式语言用于在运行时查询和操纵对象。

- 核心容器依赖的日志包

  - commons-logging-*.jar
  - log4j-*.jar

  commons-logging 相当于一个日志接口，log4j 相当于该接口的实现，如果不添加 log4j 包也可以，因为 commons-logging 也有一个简单的实现会自动使用。

##### (2) 建立配置文件

参照 [官网 XML 配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-metadata) 创建对应的对象。

##### (3) 获取 IoC 核心容器

```java
ApplicationContext ac1 = new ClassPathXmlApplicationContext("bean.xml");
ApplicationContext ac2 = new FileSystemXmlApplicationContext("bean.xml");

User user1 = (User) ac1.getBean("beanId");
User user2 = ac2.getBean("beanId", User.class);
```

#### (二) IoC 内部详解

##### (1) BeanFactory 和 ApplicationContext 的区别

- 关系

  - BeanFactory 是 Spring 容器中的顶层接口。

  - ApplicationContext 是 BeanFactory 的子接口。

- 区别

  - ApplicationContext - **立即加载策略**:  :只要一读取配置文件，默认情况下就会创建对象。在被创建的对象的构造方法中验证。 (适用于单例)

  - BeanFactory - **延迟加载策略**: 什么时候根据 id 获取对象，什么时候创建对象。(适用于多例)

    ```java
    // 使用 BeanFactory 的方式获取 Bean
    Resource rc = new ClassPathResource("Bean.xml");
    BeanFactory bf = new XMLBeanFactory(resource);
    bf.getBean("beanName");
    ```

##### (2) Spring 中的 Bean 管理 ✨

- 创建 Bean 的三种方式

  - 使用默认**无参构造方法**构建

    XML 配置文件中使用 id 和 class 配对创建 bean 默认是调用了 bean 对象的默认无参构造函数，如果 bean 中没有默认无参构造函数，将会创建失败。

  - 使用普通工厂模式中的方法创建对象 (先创建工厂 bean 对象，再用工厂创建并返回 DAO bean 对象)

    ```xml
    <Bean id="beanFactory" class="xx.xx.BeanFactory"></Bean>
    <Bean id="xxDAO" factory-bean="beanFactory" factory-method="createxxDAO"></Bean>
    ```

    先把工厂的创建交给 Spring 来管理，然后再使用工厂 bean 来调用里面的方法创建 DAO bean对象。

  - 使用工厂中的**静态方法**创建对象 (工厂的创建不归 Spring 管，工厂用静态方法创建 bean 对象)

    ```xml
    <Bean id="xxDAO" class="xx.xx.BeanFactory" factory-method="createxxDAO"></Bean>
    ```

    需要 `createxxDAO` 是静态方法。

- Bean 对象的作用范围

  Bean 标签的 scope 属性用于指定 Bean 的作用范围

  - singleton 单例的（默认）
  - prototype 多例的
  - request 作用于 Web 的请求范围 (一次 Http请求)
  - session 作用于 Web 的会话范围
  - global session 作用于集群/全局的会话范围，非集群环境则为 session。 (应用案例: 验证码储存)

- Bean 对象的生命周期(初级理解，源码详解待扩展)

  - 单例对象

    出生——当容器创建时，对象出生

    活着——只要容器还在，对象就在

    死亡——容器销毁，对象死亡

    详解版:

    ![img](https://images2018.cnblogs.com/blog/1368782/201804/1368782-20180409204927593-1218853531.png)

  - 多例对象

    - 出生——使用时，Spring为我们创建
    - 活着——对象只要在使用过程中就一直存在
    - 死亡——超时且没有别的对象引用时，由 GC 回收

##### (3) Spring 中的依赖注入

1. 概念

   > 依赖注入是指程序运行过程中，如果需要调用另一个对象协助时，无须在代码中创建被调用者，而是依赖于外部的注入

2. 允许注入的数据类型

   - 基本类型和 String
   - 其他 Bean 类型(在配置文件或者注解配置过的 Bean)
   - 复杂类型/集合类型

3. 注入的方式

   - 使用构造函数注入

     - 无参构造: 默认的 id + class 的 bean 标签对应

     - 有参构造: 在 bean 内部使用 `<construcor-arg>` 标签，它有5种属性

       - type: 给指定数据类型的属性注入数据，仅支持一个特定类型

       - index: 用于给构造函数的参数列表的对应索引变量赋值，索引从0开始

       - name: 用于给构造函数中指定名称的参数赋值

       - value: 给基本类型和 String 类型属性赋值

       - ref: 可以引用Spring容器里的 bean 作为数据类型

         ```xml
         <bean id="xx" class="">
             <construcor-arg type="java.lang.String" value="test"></construcor-arg>
             <construcor-arg type="java.lang.Integer" name="age" value="18"></construcor-arg>
             <construcor-arg name="birthday" ref="now"></construcor-arg>
         </bean>
         <bean id="now" class="java.utils.Date"></bean> <!-- 被birthday引用 -->
         ```

         

   - 使用 set 方法注入 / 使用p名称空间注入集合属性

   - 使用注解

### 三、Spring 中基于注解的 IoC 配置

### 四、IoC 的应用

## Part 3. AOP

### 一、AOP 的概念

### 二、Spring 中基于 XML 的 AOP 配置

### 三. Spring 中基于注解的 AOP 配置

## Part 4. JdbcTemlate

## Part 5. Spring 的事务控制



