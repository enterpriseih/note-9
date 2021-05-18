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