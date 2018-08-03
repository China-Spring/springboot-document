# SpringBoot 2.0.X官方文档-007-使用@SpringBootApplication注解

许多SpringBoot开发人员喜欢他们的应用程序使用自动配置、组件扫描并能够在“应用程序类”上定义额外的配置。可以使用一个**@SpringBootApplication**注解来启用这三个特性，即:

- **@EnableAutoConfiguration**：[SpringBoot 2.0.X官方文档-006-3-自动配置](http://www.springall.com.cn/article/52)
- **@ComponentScan**：**@Component**在应用程序所在的程序包上启用扫描（请参阅[SpringBoot 2.0.X官方文档-006-1-构建代码](http://www.springall.com.cn/article/50)）
- **@Configuration**：允许在上下文中注册额外的bean或导入其他配置类

**@SpringBootApplication**注解相当于使用**@Configuration**、**@EnableAutoConfiguration**和**@ComponentScan**的默认属性，如下面的示例所示:

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

**@SpringBootApplication还提供了别名定制的属性 @EnableAutoConfiguration和@ComponentScan。**

这些特性中没有一个是强制性的，您可以选择使用它支持的任何特性来替换这个单一注解。例如，您可能不想在应用程序中使用组件扫描:

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@EnableAutoConfiguration
@Import({ MyConfig.class, MyAnotherConfig.class })
public class Application {

	public static void main(String[] args) {
			SpringApplication.run(Application.class, args);
	}

}
```

在本例中，应用程序与任何其他SpringBoot应用程序一样，只是不会自动检测到**@Component**-annotated类，并且显式地导入用户定义的bean(参见**@Import**)。


