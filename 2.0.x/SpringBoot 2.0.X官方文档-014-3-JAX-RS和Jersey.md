# SpringBoot 2.0.X官方文档-014-3-JAX-RS和Jersey

如果您更喜欢REST端点的JAX-RS编程模型，则可以使用其中一个可用的实现而不是Spring MVC。如果您在应用程序上下文中注册将Servlet或Filter作为@Bean，则[Jersey](https://jersey.github.io/) 1.x和[Apache CXF](https://cxf.apache.org/)可以很好地开箱即用。Jersey 2.x有一些原生的Spring支持，因此我们还在Spring Boot中为它提供了自动配置支持以及启动器.

要开始使用Jersey 2.x，请将其**spring-boot-starter-jersey**作为依赖项包含在内，然后您需要一个注册所有端点**@Bean**的类型**ResourceConfig**，如以下示例所示：

```java
@Component
public class JerseyConfig extends ResourceConfig {

	public JerseyConfig() {
		register(Endpoint.class);
	}

}
```

**Jersey对扫描可执行档案的支持相当有限。例如，在运行可执行的war文件时，它不能扫描WEB-INF/class中的包中的端点。为了避免这种限制，不应该使用package方法，应该使用register方法分别注册端点，如上例所示。**

对于更高级的定制，您还可以注册任意数量的bean来实现**ResourceConfigCustomizer**。

所有注册的端点都应该是带有HTTP资源注解(**@GET**等)的**@Component**，如下例所示:

```java
@Component
@Path("/hello")
public class Endpoint {

	@GET
	public String message() {
		return "Hello";
	}

}
```

由于Endpoint是一个Spring的@Component，它的生命周期由Spring管理，您可以使用**@Autowired**注解注入依赖项并使用**@Value**注解注入外部配置。默认情况下，Jersey servlet已注册并映射到**/\***。您可以通过添加将**@ApplicationPath**到**ResourceConfig**来更改映射。

默认情况下，Jersey将通过@Bean以名为jerseyServletRegistration的ServletRegistrationBean类型在Servlet进行设置。 默认情况下，servlet将被初始化，但是您可以使用**spring.jersey.servlet.load-on-startup**进行自定义。您可以通过创建一个自己的同名文件来禁用或覆盖该bean。 您也可以通过设置**spring.jersey.type=filter**（在这种情况下，@Bean来替换或替换为jerseyFilterRegistration），使用Filter而不是Servlet。 servlet有一个@Order，您可以使用**spring.jersey.filter.order**设置。可以使用**spring.jersey.init.***给出Servlet和过滤器注册的init参数，以指定属性的映射。

有一个[Jersey示例](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-samples/spring-boot-sample-jersey)，所以你可以看到如何设置。 还有一个[Jersey1.x示例](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-samples/spring-boot-sample-jersey1)。 请注意，在Jersey1.x示例中，spring-boot maven插件已经被配置为打开一些Jersey jar，以便它们可以被JAX-RS实现扫描（因为示例要求它们在Filter注册中进行扫描） 。 如果您的任何JAX-RS资源作为嵌套的jar打包，您可能需要执行相同操作。

