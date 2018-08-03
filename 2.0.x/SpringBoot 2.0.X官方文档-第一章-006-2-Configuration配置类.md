# SpringBoot 2.0.X官方文档-第一章-006-2-Configuration配置类

Spring Boot支持基于Java的配置。尽管可以使用 **SpringApplication** XML源，但我们通常建议您的主要源是单个**@Configuration**类。通常，定义**main**方法的类是主要的**@Configuration**。

**许多Spring配置示例已在Internet上发布，使用XML配置。如果可能，请始终尝试使用等效的基于Java的配置。搜索Enable*注释可能是一个很好的起点。**

> 导入其他配置类

您不必将所有**@Configuration**放入一个类中。可以使用**@Import**注解来导入其他配置类。或者，您可以使用**@ComponentScan**自动提取所有Spring组件，包括**@Configuration**类。

> 导入XML配置

如果您一定要使用基于XML的配置，我们建议您还是从一个**@Configuration**类开始。然后可以使用**@ImportResource**注释来加载XML配置文件。

