### SpringBoot

​		SpringBoot 的设计是为了让尽可能快的跑起来 Spring 应用程序并且尽可能减少你的配置文件，使约定优于配置(Convention over Configuration)。



#### 一、SpringBoot应用

##### 1、热部署

###### 	1.1 热部署配置

* **引入热部署依赖**

  ```xml
  <!-- 引入热部署依赖 -->
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-devtools</artifactId>
  </dependency>
  ```

* **IDEA配置开启自动编译**

  ![image-20210510201717633](SpringBoot.assets/image-20210510201717633.png)

* IDEA**配置在运行过程中自动编译（Ctrl+Shift+Alt+/”打开Maintenance选项框）**

  ![image-20210510201958184](SpringBoot.assets/image-20210510201958184.png)

###### 1.2 热部署排除资源

​		某些资源在更改后不一定需要触发重新启动。  可以在 application.properties 里配置

```properties 
spring.devtools.restart.exclude=static/**,public/**
```

##### 2、配置文件

###### 	**2.1 properties配置信息注入实体类**

* 引入SpringBoot配置处理器依赖

  ```xml
  <dependency>
  	<groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
      <optional>true</optional>
  </dependency>
  ```

* 实体类，使用 @Component 将类加入Spring IOC 容器中，@ConfigurationProperties(prefix = "person")  将配置文件中以person开头的属性注入到该类中

  ```java
  package com.lagou.pojo;
  
  import lombok.Data;
  import org.springframework.boot.context.properties.ConfigurationProperties;
  import org.springframework.stereotype.Component;
  
  import java.util.List;
  import java.util.Map;
  
  @Data
  @Component
  @ConfigurationProperties(prefix = "person")
  public class Person {
      private String name;
      private List<String> hobby;
      private String[] family;
      private Map<String, Object> map;
      private Pet pet;
  }
  ```

  ```java
  package com.lagou.pojo;
  
  import lombok.Data;
  
  @Data
  public class Pet {
      private String type;
      private String name;
  }
  ```

* 配置文件 application.properties

  ```properties
  person.name=ZhangSan
  person.family=mother,father
  person.hobby=吃饭,睡觉
  person.map.k1 = v1
  person.map.k2 = v2
  person.pet.name = 旺财
  person.pet.type = dog
  ```

* 运行结果

  ![image-20210510234054391](SpringBoot.assets/image-20210510234054391.png)

  

  ###### **2.2 yml配置方式  application.yml**

```yaml
spring:
  devtools:
    restart:
      exclude: [static/**,public/**]
person:
  name: LiSi
  family: [mother, father]
  hobby: [吃饭, 睡觉]
  map: {k1: v1,k2: v2}
  pet: {name: 大黄,type: dog}

```

​		

###### 	2.3 配置批量注入

​	@Configuration 标识该类为一个配置类

​	@EnableConfigurationProperties 使用该注解用于启用应用对另外一个注解 @ConfigurationProperties 的支持

​	@ConfigurationProperties  批量注入配置信息

```java
package com.lagou.pojo;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(JdbcProperties.class)
@ConfigurationProperties(prefix = "jdbc")
@Data
public class JdbcProperties {

    private String url;

    private String driver;

    private String username;

    private String password;
}

```



###### 	2.3 第三方配置

​	@ConfigurationProperties 用于注释类之外，还可以在公共 @Bean 方法上使用它。可以将属性绑定到控件之外的第三方组件。

```java
@Data
public class AnotherComponent {
	private boolean enabled;
	private InetAddress remoteAddress;
}
```

```java
@Configuration
public class MyService {
    @ConfigurationProperties("another")
    @Bean
    public AnotherComponent anotherComponent(){
    	return new AnotherComponent();
    }
}
```

```properties
another.enabled=true
another.remoteAddress=192.168.10.11
```



###### 	2.4 松散绑定

​			Spring Boot使用一些宽松的规则将环境属性绑定到@ConfigurationProperties bean，因此环境属性名和bean属性名之间不需要完全匹配

```java
@Data
@Component
@ConfigurationProperties("acme.my-person.person")
public class OwnerProperties {
	private String firstName;
}
```

```yaml
acme:
	my-person:
		person:
			first-name: 泰森
```



##### 3、日志框架

###### 3.1、SLF4J 结合各种日志框架的官方示例

![image-20210517082641064](SpringBoot.assets/image-20210517082641064.png)

​		在项目中存在着各种不同的第三方 jar  时，可以使用一种和要替换的日志框架类完全一样的 jar 进行替换，这样不至于原来的第三方 jar 报错，而这个替换的 jar 其实使用了 SLF4J API.。这样项目中的日志就都可以通过 SLF4J API 结合自己选择的框架进行日志输出。

​		统一日志框架使用步骤归纳如下：1.排除系统中的其他日志框架。2.使用中间包替换要替换的日志框架。3.导入我们选择的 SLF4J 实现    

![image-20210517083415532](SpringBoot.assets/image-20210517083415532.png)

###### 3.2 自定义日志输出

```properties
# 日志配置
# 指定具体包的日志级别
logging.level.com.lagou=debug
# 控制台和日志文件输出格式
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level%logger{50} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
# 日志输出路径，默认文件spring.log
logging.file.path=spring.log
```

###### 3.3 替换日志框架

​		替换日志框架为 Log4j2   

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<artifactId>spring-boot-starter-logging</artifactId>
			<groupId>org.springframework.boot</groupId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```



#### 二、SpringBoot 源码

##### 1、SpringBoot 源码构建

###### 1.1 环境

* JDK 1.8+
* Maven 3.5+

###### 1.2 下载源码

https://github.com/spring-projects/spring-boot/releases  （当前使用spring-boot-2.2.9.RELEASE）

###### 1.3 编译源码

* 进⼊spring-boot源码根⽬录
* 执⾏mvn命令: mvn clean install -DskipTests -Pfast

###### 1.4 IDEA 打开工程

![image-20210518225730510](SpringBoot.assets/image-20210518225730510.png)

disable.checks 报红处理

```xml
<properties>
	<revision>2.2.9.RELEASE</revision>
	<main.basedir>${basedir}</main.basedir>
	<disable.checks>true</disable.checks>
</properties>
```

###### 1.5 新建一个module

![image-20210518230014825](SpringBoot.assets/image-20210518230014825.png)

Springboot 依赖版本要注意和下载的一样

![image-20210518230124593](SpringBoot.assets/image-20210518230124593.png)

建好测试类

![image-20210518230208377](SpringBoot.assets/image-20210518230208377.png)



##### 2、SpringBoot 自动配置原理

##### 3、SpringBoot run 方法执行流程

###### 3.1 run 方法执行流程及作用

![image-20210523202015106](SpringBoot.assets/image-20210523202015106.png)

![image-20210523202430807](SpringBoot.assets/image-20210523202430807.png)

![image-20210523202449025](SpringBoot.assets/image-20210523202449025.png)

![image-20210523202520369](SpringBoot.assets/image-20210523202520369.png)

run 方法执行的六个步骤

1. **获取并启动监听器**
2. **构造应用上下文环境**
3. **初始化应用上下文**
4. **刷新应用上下文前的准备阶段**
5. **刷新应用上下文**
6. **刷新应用上下文后的扩展接口**

```java
	public ConfigurableApplicationContext run(String... args) {
		//记录程序运行时间
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		// ConfigurableApplicationContext Spring 的上下文
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		//【1、获取并启动监听器】
		SpringApplicationRunListeners listeners = getRunListeners(args);
        // 启动监听器
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			//【2、构造应用上下文环境】
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			//处理需要忽略的Bean
			configureIgnoreBeanInfo(environment);
			//打印banner
			Banner printedBanner = printBanner(environment);
			///【3、初始化应用上下文】
			context = createApplicationContext();
			//实例化SpringBootExceptionReporter.class，用来支持报告关于启动的错误
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			//【4、刷新应用上下文前的准备阶段】
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			//【5、刷新应用上下文】
			refreshContext(context);
			//【6、刷新应用上下文后的扩展接口】
			afterRefresh(context, applicationArguments);
			//时间记录停止
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			//发布容器启动完成事件
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```



##### 4、自定义 Start

###### 4.1 SpringBoot Start 机制

​		SpringBoot中的starter是一种非常重要的机制，能够抛弃以前繁杂的配置，将其统一集成进starter，应用者只需要在maven中引入starter依赖，SpringBoot就能自动扫描到要加载的信息并启动相应的默认配置。Starter让我们摆脱了各种依赖库的处理，需要配置各种信息的困扰。
​		SpringBoot会自动通过classpath路径下的类发现需要的Bean，并注册进IOC容器。SpringBoot提供了针对日常企业应用研发各种场景的spring-boot-starter依赖模块。所有这些依赖模块都遵循着约定成俗的默认配置，并允许我们调整这些配置，即遵循“约定大于配置”的理念。  

###### 4.2 自定义starter的命名规则

​		SpringBoot提供的starter以 spring-boot-starter-xxx 的方式命名的。

​		官方建议自定义的starter使用 xxx-spring-boot-starter 命名规则。以区分SpringBoot生态提供的starter。

###### 4.3 自定义starter代码实现

* 新建 maven jar 工程，工程名为 zdy-spring-boot-starter，导入依赖：

  ```xml
  <dependencies>
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-autoconfigure</artifactId>
  		<version>2.2.9.RELEASE</version>
  	</dependency>
  </dependencies>
  ```

* 编写 JavaBean

  ```java
  package com.lagou.pojo;
  
  import lombok.Data;
  import org.springframework.boot.context.properties.ConfigurationProperties;
  import org.springframework.boot.context.properties.EnableConfigurationProperties;
  
  @Data
  @EnableConfigurationProperties
  @ConfigurationProperties(prefix = "simplebean")
  public class SimpleBean {
  
  	private int id;
  	private String name;
  }
  
  ```

* 编写配置类 MyAutoConfiguration

  ```java
  package com.lagou.config;
  
  import com.lagou.pojo.SimpleBean;
  import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  public class MyAutoConfiguration {
  
  	static {
  		System.out.println("MyAutoConfiguration init.... ");
  	}
  
  	@Bean
  	public SimpleBean simpleBean(){
  		return new SimpleBean();
  	}
  
  }
  
  ```

* resources下创建/META-INF/spring.factories ，配置编写的自动配置类

  ![image-20210608011912085](SpringBoot.assets/image-20210608011912085.png)

  ```properties
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.lagou.config.MyAutoConfiguration
  ```

* 使用自定义 starter，导入自定义 starter 依赖，在全局配置文件中配置属性值

  ```xml
  <dependency>
  	<groupId>com.lagou</groupId>
  	<artifactId>zdy-spring-boot-starter</artifactId>
  	<version>1.0-SNAPSHOT</version>
  </dependency>
  ```

  ```properties
  simplebean.id=1
  simplebean.name=zhangsan
  ```

###### 4.4 热插拔技术

<img src="SpringBoot.assets/image-20210608012345053.png" alt="image-20210608012345053" style="zoom:50%;" />

​		@Enablexxx注解就是一种热拔插技术，加了这个注解就可以启动对应的starter，当不需要对应的starter的时候只需要把这个注解注释掉就行。

​		改造自定义 starter 工程支持热插拔

* 新增标记类ConfigMarker

  ```java
  package com.lagou.config;
  
  public class ConfigMarker {
  }
  ```

* 新增 EnableRegisterServer  注解

  ```java
  package com.lagou.config;
  
  
  import org.springframework.context.annotation.Import;
  
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Import(ConfigMarker.class)
  public @interface EnableRegisterServer {
  }
  ```

* 改造 MyAutoConfiguration 新增条件注解 @ConditionalOnBean(ConfigMarker.class) ，
  @ConditionalOnBean 这个是条件注解，前面的意思代表只有当期上下文中含有 ConfigMarker对象，被标注的类才会被实例化 

  ```java
  package com.lagou.config;
  
  import com.lagou.pojo.SimpleBean;
  import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  @ConditionalOnBean(ConfigMarker.class)
  public class MyAutoConfiguration {
  
  	static {
  		System.out.println("MyAutoConfiguration init.... ");
  	}
  
  	@Bean
  	public SimpleBean simpleBean(){
  		return new SimpleBean();
  	}
  
  }
  ```

##### 5、内嵌tomcat原理

SpringBoot支持Tomcat，Jetty，Undertow作为底层容器。
SpringBoot默认使用Tomcat，一旦引入spring-boot-starter-web模块，就默认使用Tomcat容器。

spring-boot-starter-web核心就是引入了tomcat和SpringMVC。

###### 5.1 切换servlet容器

* 将tomcat依赖移除
* 引入其他Servlet容器依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
        <exclusion>
        <!--移除spring-boot-starter-web中的tomcat-->
		<artifactId>spring-boot-starter-tomcat</artifactId>
		<groupId>org.springframework.boot</groupId>
		<exclusion>
     </exclusions>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<!--引入jetty-->
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

###### 5.2 内嵌 tomcat 源码

**进入 SpringBoot 启动类 @SpringBootApplication**

![image-20210609003501644](SpringBoot.assets/image-20210609003501644.png)

**找到自动配置注解 @EnableAutoConfiguration** 

![image-20210609003635626](SpringBoot.assets/image-20210609003635626.png)

**@EnableAutoConfiguration 注解对 AutoConfigurationImportSelector 进行了引入**

![image-20210609003744460](SpringBoot.assets/image-20210609003744460.png)

**首先调用 selectImport() 方法，在该方法中调用了 getAutoConfigurationEntry() 方法，在之中又调用了getCandidateConfigurations() 方法，getCandidateConfigurations() 方法就去 META-INF/spring.factory 配置文件中加载相关配置类**

![image-20210609004334480](SpringBoot.assets/image-20210609004334480.png)

![image-20210609004513058](SpringBoot.assets/image-20210609004513058.png)

![image-20210609004554458](SpringBoot.assets/image-20210609004554458.png)

**tomcat 加载在 ServletWebServerFactoryAutoConfiguration 配置类中**  

![image-20210609004945404](SpringBoot.assets/image-20210609004945404.png)

**通过@Import注解将EmbeddedTomcat、EmbeddedJetty、EmbeddedUndertow等嵌入式容器类加载进来了**  

![image-20210609005902421](SpringBoot.assets/image-20210609005902421.png)

**EmbeddedTomcat 中实例化了 tomcat 工厂类**

![image-20210609010031183](SpringBoot.assets/image-20210609010031183.png)

**在 TomcatServletWebServerFactory 的 getWebServer() 方法中，实例化了一个 tomcat，设置了 tomcat 相关信息**

![image-20210609010156844](SpringBoot.assets/image-20210609010156844.png)

**一直进入 getTomcatWebServer() 方法，最终在 initialize() 方法中启动了 tomcat**

![image-20210609010614426](SpringBoot.assets/image-20210609010614426.png)

###### 5.3 getWebServer() 的调用分析  



##### 6、自动配置SpringMVC功能

​		普遍 web 项目使用 SpringMVC 需要配置 web.xml ，配置对应的DispatcherServlet 。

 ```xml
<servlet>
	<description>spring mvc servlet</description>
	<servlet-name>springMvc</servlet-name>
	<servletclass>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springMvc</servlet-name>
	<url-pattern>*.do</url-pattern>
</servlet-mapping>
 ```

​		在 Servlet3.0 规范中，要添加一个Servlet，处理采用xml配置的方式，还可以通过代码添加

```java
servletContext.addServlet(name, this.servlet);
```

###### 6.1 自动配置DispatcherServlet和DispatcherServletRegistry

SpringMVC 自动配置类

  ```java
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
  ```

![image-20210614162516870](SpringBoot.assets/image-20210614162516870.png)

DispatcherServletAutoConfiguration主要包含两个内部类DispatcherServletConfiguration 和DispatcherServletRegistrationConfiguration 。

![image-20210616005457790](SpringBoot.assets/image-20210616005457790.png)

![image-20210616005412719](SpringBoot.assets/image-20210616005412719.png)

​		前者是配置DispatcherServlet，后者是配置DispatcherServlet的注册类。什么是注册类？我们知道Servlet实例是要被添加（注册）到如tomcat这样的ServletContext里的，这样才能够提供请求服务。所以，DispatcherServletRegistrationConfiguration将生成一个Bean，负责将
DispatcherServlet给注册到ServletContext中 。

**DispatcherServletConfiguration**这个内部类

​		@Conditional指明了一个前置条件判断，由DefaultDispatcherServletCondition实现。主要是判
断了是否已经存在DispatcherServlet，如果没有才会触发解析。
​		@ConditionalOnClass指明了当ServletRegistration这个类存在的时候才会触发解析，生成的
DispatcherServlet才能注册到ServletContext中。
​		最后，@EnableConfigrationProperties将会从application.properties这样的配置文件中读取
spring.http和spring.mvc前缀的属性生成配置对象HttpProperties和WebMvcProperties 。

​		dispatcherServlet方法将生成一个DispatcherServlet的Bean对象。比较简单，就是获取一个实
例，然后添加一些属性设置。
​		multipartResolver方法主要是把你配置的MultipartResolver的Bean给重命名一下，防止你不是用
multipartResolver这个名字作为Bean的名字。

**DispatcherServletRegistrationConfiguration**这个内部类

​		@Conditional有一个前置判断，DispatcherServletRegistrationCondition主要判断了该注册类的Bean是否存在。
​		@ConditionOnClass也判断了ServletRegistration是否存在。
​		@EnableConfigurationProperties生成了WebMvcProperties的属性对象。
​		@Import导入了DispatcherServletConfiguration，也就是我们上面的配置对象。   

​		只有一个方法DispatcherServletRegistrationBean，生成了DispatcherServletRegistrationBean。核心逻辑就是实例化了一个Bean，设置了一些参数，如dispatcherServlet、loadOnStartup等。 



###### 6.2 注册DispatcherServlet到ServletContext

​		看到了DispatcherServlet和DispatcherServletRegistrationBean这两个Bean的自动配置。DispatcherServlet我们很熟悉，DispatcherServletRegistrationBean负责将DispatcherServlet注册到ServletContext当中。

DispatcherServletRegistrationBean 类图

![image-20210616204111334](SpringBoot.assets/image-20210616204111334.png) 

从ServletContextInitializer接口一直往下查看，这里直接将DispatcherServlet给add到了servletContext当中 。

![image-20210616204445339](SpringBoot.assets/image-20210616204445339.png)

SpringBoot 启动时依次去调用对象的 onStartup 方法，就是会调用到DispatcherServletRegistrationBean 的 onStartup 方法，这个类并没有这个方法，所以最终会调用到父类 RegistrationBean 的 onStartup 方法

![image-20210616205606625](SpringBoot.assets/image-20210616205606625.png)



#### 三、SpringBoot数据源及连接池配置

##### 1、数据源配置

###### 1.1 maven配置MySql数据库驱动

```xml
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```

###### 1.2 数据库连接配置

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/lagou?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root
```

###### 1.3、配置spring-boot-starter-jdbc  

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

###### 1.4 测试

![image-20210617204417456](SpringBoot.assets/image-20210617204417456.png)



##### 2、连接池配置方式

###### 2.1 SpringBoot 默认提供了三种数据库连接方式

* HikariCP

* Commons DBCP2

* Tomcat JDBC Connection Pool

  

###### 2.2 spring boot2.x版本默认使用HikariCP

maven配置

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

改用 Commons DBCP2 ，maven配置

```xml
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-dbcp2</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
	<exclusions>
		<exclusion>
			<groupId>com.zaxxer</groupId>
			<artifactId>HikariCP</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

SpringBoot哪里配置默认使用HikariCP

![image-20210617204951113](SpringBoot.assets/image-20210617204951113.png)

![image-20210617205007884](SpringBoot.assets/image-20210617205007884.png)



##### 3、数据源自动配置原理

/spring-boot-2.2.9.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories 中找到

```xml
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
```

![image-20210617221831673](SpringBoot.assets/image-20210617221831673.png)

查看注解@Conditional(PooledDataSourceCondition.class)，根据判断条件，实例化这个类，指定了配置文
件中，必须有type这个属性

![image-20210617222133911](SpringBoot.assets/image-20210617222133911.png)

```properties
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
```

spring.datasource.type作用是设置当前连接池类型

如果要改默认连接池，需要导入对应jar包，并且更改spring.datasource.type。



##### 4、配置Druid连接池

###### 4.1 引入对应 Maven 依赖

```xml
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid-spring-boot-starter</artifactId>
	<version>1.1.10</version>
</dependency>
```

###### 4.2 配置文件配置 Druid

​		这边使用的是application.yml，也可以在application.properties配置

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql:///springboot_h?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    initialization-mode: always
    # 使用druid数据源
    type: com.alibaba.druid.pool.DruidDataSource
    # 数据源其他配置
    # 初始化连接数，设为0表示无限制
    initialSize: 5
    # 最小的空闲连接
    minIdle: 5
    # 最大连接数
    maxActive: 20
    # 最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制
    maxWait: 60000
    # 检查空闲连接的频率，单位毫秒, 非正整数时表示不进行检查
    timeBetweenEvictionRunsMillis: 60000
    # 池中某个连接的空闲时长达到 N 毫秒后, 连接池在下次检查空闲连接时，将回收该连接,要小于防火墙超时设置net.netfilter.nf_conntrack_tcp_timeout_established的设置
    minEvictableIdleTimeMillis: 300000
    # 检查池中的连接是否仍可用的SQL语句，drui会连接到数据库执行该SQL, 如果正常返回，则表示连接可用，否则表示连接不可用
    validationQuery: SELECT 1 FROM DUAL
    # 当程序请求连接，池在分配连接时，是否先检查该连接是否有效。(高效)
    testWhileIdle: true
    # 程序申请连接时，进行连接有效性检查（低效，影响性能）
    testOnBorrow: false
    # 程序返还连接时，进行连接有效性检查（低效，影响性能）
    testOnReturn: false
    # 缓存通过以下两个方法发起的SQL:
    # public PreparedStatement prepareStatement(String sql)
    # public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency)
    poolPreparedStatements: true
    # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    # 每个连接最多缓存多少个SQL
    maxPoolPreparedStatementPerConnectionSize: 20
    # 是否开启Druid的监控统计功能
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

###### 4.3 编写 druid 配置类 DruidConfig 

​		如果没有 druid 配置类，配置文件里对 druid 的配置就不会生效

```java
package com.lagou.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class DruidConfig {
	
	@ConfigurationProperties(prefix = "spring.datasource")
	@Bean
	public DataSource dataSource(){
		return  new DruidDataSource();
	}

}
```

###### 4.4 引入 slf4j-log4j12 适配器

​		因为 druid 配置里用到了 log4j，而 springBoot2.0 以后使用的日志框架已经不再使用 log4j 了。此时应该引入相应的适配器。

```xml
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-log4j12</artifactId>
</dependency>
```



##### 5、SpringBoot 整合 MyBatis

导入对应MyBatis包，配置好配置文件

```xml
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.2</version>
</dependency>
```

![image-20210620220811682](SpringBoot.assets/image-20210620220811682.png)

可以在启动类配置mapper包扫描，Spring会把这个包下的类都加入Spring容器，这样mapper接口就不用每个去添加@Mapper注解

![image-20210620220829168](SpringBoot.assets/image-20210620220829168.png)



##### 6、MyBatis 自动配置原理

找到 MyBatis 自动配置类

![image-20210621202651512](SpringBoot.assets/image-20210621202651512.png)

@ConditionalOnMissingBean作用：在没有类的时候调用，创建sqlsessionFactory，sqlsessionfactory最主要的是创建并保存了Configuration类 。

获取SqlSessionFactoryBean的getObject()中的对象注入Spring容器，也就是
SqlSessionFactory对象，最终调用了MyBatis的初始化流程  

![image-20210621203358747](SpringBoot.assets/image-20210621203358747.png)

![image-20210621204014198](SpringBoot.assets/image-20210621204014198.png)

![image-20210621204041238](SpringBoot.assets/image-20210621204041238.png)

扫描 mapper 接口以及生成代理对象

启动类配置扫描 mapper 接口注解

![image-20210621205417523](SpringBoot.assets/image-20210621205417523.png)

![image-20210621205659500](SpringBoot.assets/image-20210621205659500.png)

MapperScannerRegistrar 实现了 ImportBeanDefinitionRegistrar 接口，重写了 registerBeanDefinitions 方法，在Spring 实例话的时候回去调用改方法

![image-20210621205723644](SpringBoot.assets/image-20210621205723644.png)

#### 注解

##### 	属性注入常用注解  

###### 	@Configuration

​			声明一个类作为配置类

###### 	@Bean

​			声明在方法上，将方法的返回值加入Bean容器

###### 	@Value

​			属性注入

```java
@Value("${com.location}")
private String location;
```

###### 	@ConfigurationProperties

​			批量属性注入，通常使用@EnableConfigurationProperties启用该注解用于启用

![image-20210514011925107](SpringBoot.assets/image-20210514011925107.png)

###### 	@PropertySource

​			指定外部属性文件。在类上添加 

![image-20210514013946636](SpringBoot.assets/image-20210514013946636.png)