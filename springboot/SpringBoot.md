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