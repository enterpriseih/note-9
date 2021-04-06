### Spring

##### 一、Spring简介

​		Spring 是分层的 full-stack（全栈） 轻量级开源框架，以 IoC 和 AOP 为内核，提供了展现层 Spring
MVC 和业务层事务管理等众多的企业级应⽤技术，还能整合开源世界众多著名的第三⽅框架和类库，已
经成为使⽤最多的 Java EE 企业应⽤开源框架 。

##### 二、内核思想

###### 1、IOC（Inversion of Control）控制反转

​		IOC是一种设计思想，可以用来降低计算机代码之间的耦合度。由一个调控系统管理创建设计好的对象，在需要使用对象时，由调控系统去提供。即原本由程序主动去创建依赖对象，现在由调控系统控制。由自己在对象中主动控制去直接获取依赖对象，反转为由调控系统帮我们查找及注入依赖对象。

###### 2、AOP（spect oriented Programming）面向切面编程

​		AOP面向切面编程，可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。是对OOP面向对象的延续和补充，拆分业务代码和横切逻辑代码 ，在不改变原有业务逻辑情况下，增强横切逻辑代码，根本上解耦合，避免横切逻辑代码重复。

###### 3、DI（Dependancy Injection）依赖注入

​		DI依赖注入是我们实现控制反转的一种手段。把程序运行所需要的外部资源通过容器加载注入给程序内的对象，例如：当实例化对象A需要依赖对象B时，由spring容器去实例化对象B注入给A。

##### 三、Spring架构

![img](Spring.assets/clipboard.png)

- **核心容器**：核心容器提供 Spring 框架的基本功能。核心容器的主要组件是BeanFactory，它是工厂模式的实现。BeanFactory 使用*控制反转*（IOC） 模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。
- **Spring 上下文**：Spring 上下文是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如 JNDI、EJB、电子邮件、国际化、校验和调度功能。
- **Spring AOP**：通过配置管理特性，Spring AOP 模块直接将面向切面的编程功能 , 集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理任何支持 AOP的对象。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖组件，就可以将声明性事务管理集成到应用程序中。
- **Spring DAO**：JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。
- **Spring ORM**：Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具，其中包括 JDO、Hibernate 和 iBatis SQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。
- **Spring Web 模块**：Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring 框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。
- **Spring MVC 框架**：MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

##### 四、Spring使用

###### bean标签属性

| 属性           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| id             | bean的唯⼀标识。                                             |
| class          | 指定创建Bean对象的全限定类名。                               |
| name           | 给bean提供⼀个或多个名称，多个名称⽤空格分隔。               |
| factory-bean   | 指定创建当前bean对象的⼯⼚bean的唯⼀标识。当指定了此属性之后，<br/>class属性失效。 |
| factory-method | ⽤于指定创建当前bean对象的⼯⼚⽅法，如配合factory-bean属性使⽤，<br/>则class属性失效。如配合class属性使⽤，则⽅法必须是static的。 |
| scope          | 指定bean对象的作⽤范围。通常情况下就是singleton。当要⽤到多例模式时，<br/>可以配置为prototype。 |
| init-method    | 指定bean对象的初始化⽅法，此⽅法会在bean对象装配后调⽤。必须是<br/>⼀个⽆参⽅法。 |
| destory-method | 指定bean对象的销毁⽅法，此⽅法会在bean对象销毁前执⾏。它只<br/>能为scope是singleton时起作⽤。 |
| lazy-init      | 是否延迟加载。                                               |

###### xml配置bean实例化的方式

* 使用⽆参构造函数

  ```xml
  <bean id="userService" class="com.lagou.service.impl.TransferServiceImpl"></bean>
  ```

* 使⽤静态⽅法

  ```xml
  <bean id="userService" class="com.lagou.factory.BeanFactory" 
  factory-method="getTransferService"></bean>
  ```

* 使⽤实例化⽅法

  ```xml
  <bean id="beanFactory"
  class="com.lagou.factory.instancemethod.BeanFactory"></bean>
  <bean id="transferService" factory-bean="beanFactory" factorymethod="getTransferService"></bean>
  ```

###### bean的作用域

| 作用域      | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| singleton   | （默认）在每个Spring IoC容器中，一个bean定义对应只会有唯一的一个bean实例。 |
| prototype   | 一个bean定义可以有多个bean实例。                             |
| request     | 一个bean定义对应于单个HTTP 请求的生命周期。也就是说，每个HTTP 请求都有一个bean实例，且该实例仅在这个HTTP 请求的生命周期里有效。该作用域仅适用于WebApplicationContext环境。 |
| session     | 一个bean 定义对应于单个HTTP Session 的生命周期，也就是说，每个HTTP Session 都有一个bean实例，且该实例仅在这个HTTP Session 的生命周期里有效。该作用域仅适用于WebApplicationContext环境。 |
| application | 一个bean 定义对应于单个ServletContext 的生命周期。该作用域仅适用于WebApplicationContext环境。 |
| websocket   | 一个bean 定义对应于单个websocket 的生命周期。该作用域仅适用于WebApplicationContext环境。 |

###### Spring事务传播行为

​		事务传播行为用来描述由某一个事务传播行为修饰的方法被嵌套进另一个方法的时事务如何传播。

![img](Spring.assets/clipboard-1616561482050.png)