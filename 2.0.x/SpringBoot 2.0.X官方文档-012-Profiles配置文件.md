# SpringBoot 2.0.X官方文档-012-Profiles配置文件

Spring Profiles提供了一种隔离应用程序配置部分并使其仅在特定环境中可用的方法。任何**@Component**或**@Configuration**可以标记**@Profile**以限制何时加载，如以下示例所示：

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {

	// ...

}
```

您可以使用**spring.profiles.active** Environment属性指定哪些配置文件处于活动状态。您可以使用本章前面介绍的方法指定属性。例如，您可以将其包含在您的中application.properties，如以下示例所示：

```bash
spring.profiles.active=dev,hsqldb
```

您还可以使用以下开关在命令行上指定它： **--spring.profiles.active=dev,hsqldb**。

> 添加激活配置文件

**spring.profiles.active**属性遵循与其他属性相同的优先级规则，PropertySource最高。这意味着您可以在**application.properties**中指定活动配置文件，然后使用命令行开关替换它们。

有时，将特定于配置文件的属性添加到激活的配置文件而不是替换它们是有用的。 **spring.profiles.include**属性可用于无条件添加激活配置文件。 **SpringApplication**入口点还具有用于设置其他配置文件的Java API（即，在由**spring.profiles.active**属性激活的那些配置文件之上）：请参阅setAdditionalProfiles()方法。

例如，当使用开关 **--spring.profiles.active=prod**运行具有以下属性的应用程序时，proddb和prodmq配置文件也将被激活：

```yml
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
  - proddb
  - prodmq
```

**请记住，spring.profiles可以在YAML文档中定义属性，以确定此特定文档何时包含在配置中。**

> 以编程方式设置配置文件

您可以通过在应用程序运行之前调用**SpringApplication.setAdditionalProfiles(…)**以编程方式设置激活配置文件。 也可以使用Spring的**ConfigurableEnvironment**接口激活配置文件。

> 特定于配置文件的配置文件

通过**@ConfigurationProperties**引用的**application.properties**（或**application.yml**）和文件的配置文件特定变体都被视为加载文件。

