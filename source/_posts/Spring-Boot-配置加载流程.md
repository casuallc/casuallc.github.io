---
title: Spring Boot 配置加载流程
author: changqing
categories:
  - 2024
date: 2024-03-08 09:56:25
updated: 2024-03-08 09:56:25
tags:
---

# 配置文件加载方式
## 指定加载配置的几种方式
- 使用环境变量（java -Dxx xx.jar）
  
  -Dspring.config.location=

  -Dspring.config.additional-location=
  
- 使用命令行参数(java xx.jar 参数)
  
  --spring.config.location=

  --spring.config.additional-location=

- 使用 profile
  -Dspring.profiles.active=dev
  
  然后就会加载 application-dev.properties 文件

- 使用 include
  
  -Dspring.profiles.include=dev,test
  

注意：active 和 include 都是只能定义一次，不能重复定义。
  

## 常见配置位置
1. config 目录
2. 当前目录
3. classpath 下的 config 目录
4. classpath 目录

高优先级的配置会覆盖低优先级的配置。

<!-- more -->

## 完整的配置加载顺序
**Devtools global settings properties on your home directory (~/.spring-boot-devtools.properties when devtools is active).**

可以在服务器 home 目录放置配置文件：.spring-boot-devtools.properties，该文件中的配置对服务器上的所有 spring boot 项目都生效。

**@TestPropertySource annotations on your tests.**

**properties attribute on your tests. Available on @SpringBootTest and the test annotations for testing a particular slice of your application.**

**Command line arguments.**

命令行中传递的参数，例如：java -jar xxx.jar --server.port=9000

如果要禁用命令行传递参数，则设置 ``` SpringApplication.setAddCommandLineProperties(false) ```

**Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property).**

通过设置环境变量或者命令行传递参数的方式使配置生效。
例如：
```
方式一：
SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar

方式二：
java -Dspring.application.json='{"name":"test"}' -jar myapp.jar

方式三：
java -jar myapp.jar --spring.application.json='{"name":"test"}'

```


**ServletConfig init parameters.**

**ServletContext init parameters.**

**JNDI attributes from java:comp/env.**

**Java System properties (System.getProperties()).**

**OS environment variables.**

**A RandomValuePropertySource that has properties only in random.*.**

如果需要设置动态密码等配置，可以在配置文件中使用 random，例如：
```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

此类参数的解析类是：RandomValuePropertySource 

**Profile-specific application properties outside of your packaged jar (application-{profile}.properties and YAML variants).**

jar 包外的 application-{profile}.properties、yaml 文件。

**Profile-specific application properties packaged inside your jar (application-{profile}.properties and YAML variants).**

jar 包内的 application-{profile}.properties、yaml 文件。

profile 即启动程序指定的配置：``` spring.profiles.active ``` 指定的内容。需要特别说明的是，如果使用 ``` spring.config.location ``` 指定了具体的文件，则 ``` spring.profiles.active ``` 不再生效。

如果要同时支持两个，则需要用 ``` spring.config.location ``` 指定目录。

**Application properties outside of your packaged jar (application.properties and YAML variants).**

jar 包内的 application.properties 文件。

**Application properties packaged inside your jar (application.properties and YAML variants).**

jar 包外的 application.properties 文件。

**@PropertySource annotations on your @Configuration classes. Please note that such property sources are not added to the Environment until the application context is being refreshed. This is too late to configure certain properties such as logging.* and spring.main.* which are read before refresh begins.**

**Default properties (specified by setting SpringApplication.setDefaultProperties).**

## 引用配置文件中已有的配置
如果配置文件中有很多相似的配置，则可以使用占位符的方式在后面的配置中引用前面已经配置过的配置项。

```
app.name=MyApp
app.description=${app.name} is a Spring Boot application.
```

## 在应用启动前自定义配置处理器
Spring Boot 没有内置任何加密方法，但是可以利用 ``` EnvironmentPostProcessor  ``` 简介实现该功能，也可以使用 Spring Cloud Vault 功能。

在 SpringApplication 启动前可以添加自己的监听器，在监听器中处理配置等功能。

Spring Boot 从 META-INF/spring.factories 中加载一些自定义的监听器并执行，这里有几种方式注册监听器：
- 在 SpringApplication 中调用 addListeners 和 addInitializers，``` new SpringApplicationBuilder(CRMApplication.class).listeners(null).run(args); ```
- 设置 context.initializer.classes 或者 context.listener.classes 属性
- 在 META-INF/spring.factories 中添加，并作为一个 jar 包被引入其他应用中。

实现 EnvironmentPostProcessor 接口并加载，然后就可以加载新的配置或者修改配置，例如：
```java
public class EnvironmentPostProcessorExample implements EnvironmentPostProcessor {

	private final YamlPropertySourceLoader loader = new YamlPropertySourceLoader();

	@Override
	public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
		Resource path = new ClassPathResource("com/example/myapp/config.yml");
		PropertySource<?> propertySource = loadYaml(path);
		environment.getPropertySources().addLast(propertySource);
	}

	private PropertySource<?> loadYaml(Resource path) {
		if (!path.exists()) {
			throw new IllegalArgumentException("Resource " + path + " does not exist");
		}
		try {
			return this.loader.load("custom-resource", path).get(0);
		}
		catch (IOException ex) {
			throw new IllegalStateException("Failed to load yaml configuration from " + path, ex);
		}
	}

}

```

## 配置便捷注入
通常在代码中使用 ``` @Value("${property.name}") ``` 注入配置内容，如果很多配置具有相同的前缀，则可以使用 ``` ConfigurationProperties ``` 注入，例如：

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

	private boolean enabled;

	private InetAddress remoteAddress;

	private final Security security = new Security();

	public boolean isEnabled() { ... }

	public void setEnabled(boolean enabled) { ... }

	public InetAddress getRemoteAddress() { ... }

	public void setRemoteAddress(InetAddress remoteAddress) { ... }

	public Security getSecurity() { ... }

	public static class Security {

		private String username;

		private String password;

		private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

		public String getUsername() { ... }

		public void setUsername(String username) { ... }

		public String getPassword() { ... }

		public void setPassword(String password) { ... }

		public List<String> getRoles() { ... }

		public void setRoles(List<String> roles) { ... }

	}
}
```

配置文件中的配置项如下：
```properties
# with a value of false by default.
acme.enabled=false 

# with a type that can be coerced from String.
acme.remote-address=1.1.1.1

# with a nested "security" object whose name is determined by the name of the property. In particular, the return type is not used at all there and could have been SecurityProperties.
acme.security.username=name

acme.security.password=pwd

# with a collection of String.
acme.security.roles=role1,role2
```

同时还需要主动注册上边的配置类：
``` java
@Configuration
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

# 参考
1. https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/boot-features-external-config.html
2. https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/using-boot-devtools.html#using-boot-devtools-globalsettings
3. https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/howto-spring-boot-application.html#howto-customize-the-environment-or-application-context