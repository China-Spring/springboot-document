# SpringBoot 2.0.X官方文档-第一章-006-1-构建代码

Spring Boot不需要任何特定的代码布局。但是，有一些最佳实践可以提供帮助。

> 使用“default”包

当一个类不包含**package**声明时，它被认为是在“default包”中。通常不鼓励使用“default包”，应该避免使用。它会给使用**@ComponentScan**、**@EntityScan**或**@SpringBootApplication**注解的Spring引导应用程序带来特别的问题，因为每个jar的每个类都被读取。

**我们建议您遵循Java推荐的程序包命名约定并使用反向域名（例如，com.example.project）。**

> 定位主应用程序类

我们通常建议您将主应用程序类放在其他类之上的根包中。[@SpringBootApplication](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/using-boot-using-springbootapplication-annotation.html)注解通常放在主类上，它隐式地为某些项定义了基本的“搜索包”。例如，如果您正在编写JPA应用程序，**@SpringBootApplication**则使用带注解的类的包来搜索**@Entity**项目。使用根包还允许组件扫描仅应用于您的项目。

**如果您不想使用@SpringBootApplication，那么它导入的@EnableAutoConfiguration和@ComponentScan注解定义了这种行为，因此您也可以使用它。**

下面的清单展示了一个典型的布局:

```bash
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

该**Application.java**文件将声明该**main**方法以及基本方法， **@SpringBootApplication**如下所示：

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

> 
> 
> 
> 
> 

