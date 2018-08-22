# SpringBoot 2.0.X官方文档-014-4-嵌入式Servlet容器的支持

Spring Boot包括对嵌入式[Tomcat](https://tomcat.apache.org/)， [Jetty](https://www.eclipse.org/jetty/)和[Undertow](http://undertow.io/)服务器的支持。大多数开发人员使用适当的“Starter”来获取完全配置的实例。默认情况下，嵌入式服务器将监听端口**8080**上的HTTP请求。

**如果您选择在[CentOS](https://www.centos.org/)上使用Tomcat，请注意，默认情况下，临时目录用于存储已编译的JSP，文件上传等。当您的应用程序正在运行导致故障时，该目录可能会被tmpwatch删除。 为了避免这种情况，您可能需要自定义tmpwatch配置，以便tomcat.*目录不被删除，或配置server.tomcat.basedir，以便嵌入式Tomcat使用不同的位置**

### Servlets,Filters和Listeners

当使用嵌入式servlet容器时，可以使用Spring bean或通过扫描Servlet组件（例如**HttpSessionListener**）注册Servlet规范中的Servlet，过滤器和所有监听器。

> 将Servlets，过滤器和监听器注册为Spring bean

任何**Servlet**，**Filter**或**Servlet \*Listener**实例都会作为Spring bean注册到嵌入式容器中。 可以非常方便地在配置过程中引用您的**application.properties**中的值。

默认情况下，如果容器中只包含一个Servlet，它将映射到**/**。 在多个Servlet bean的情况下，bean名称将作为路径前缀。 过滤器(Filters)将映射到**/\***，默认过滤所有请求。

如果基于惯例的映射不够灵活，可以使用**ServletRegistrationBean**，**FilterRegistrationBean**和**ServletListenerRegistrationBean**类来完成控制。

Spring Boot附带了许多可以定义Filter bean的自动配置。以下是过滤器及其各自顺序的一些示例（较低的顺序值表示较高的优先级）：

|Servlet Filter|Order|
|---|---|
|OrderedCharacterEncodingFilter|Ordered.HIGHEST_PRECEDENCE|
|WebMvcMetricsFilter|Ordered.HIGHEST_PRECEDENCE + 1|
|ErrorPageFilter|Ordered.HIGHEST_PRECEDENCE + 1|
|HttpTraceFilter|Ordered.LOWEST_PRECEDENCE - 10|

通常，让Filter bean处于无序状态是安全的。

如果需要特定的顺序，则应避免配置读取请求主体的过滤器Ordered.HIGHEST_PRECEDENCE,因为它可能违背应用程序的字符编码配置。如果Servlet筛选器包装请求，它应该配置一个小于或等于**FilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER**的顺序。

### Servlet上下文初始化

嵌入式servlet容器不会直接执行Servlet 3.0+ **javax.servlet.ServletContainerInitializer**接口或Spring的**org.springframework.web.WebApplicationInitializer**接口。 这样设计的目的旨在降低在war中运行的第三方库破坏Spring Boot应用程序的风险。

如果您需要在Spring Boot应用程序中执行servlet context 初始化，则应注册一个实现**org.springframework.boot.context.embedded.ServletContextInitializer**接口的bean。 单个onStartup方法提供对ServletContext的访问，并且如果需要，可以轻松地用作现有WebApplicationInitializer的适配器。

> 扫描Servlets,Filters,和Listeners

使用嵌入式容器时，可以使用**@ServletComponentScan**启用**@WebServlet**，**@WebFilter**和**@WebListener**注解类的自动注册。

**@ServletComponentScan在独立容器中不起作用，在该容器中将使用容器的内置发现机制。**

### ServletWebServerApplicationContext

在Spring Boot引导下，将会使用一种新类型的ApplicationContext来支持嵌入式的servlet容器。 **EmbeddedWebApplicationContext**是一种特殊类型的WebApplicationContext，它通过搜索单个**EmbeddedServletContainerFactory** bean来引导自身。 通常，**TomcatEmbeddedServletContainerFactory**，**JettyEmbeddedServletContainerFactory**或**UndertowEmbeddedServletContainerFactory**将被自动配置。

您通常不需要知道这些实现类。 大多数应用程序将被自动配置，并将代表您创建适当的ApplicationContext和EmbeddedServletContainerFactory。

### 自定义嵌入式Servlet容器

可以使用Spring Environment属性配置常见的servlet容器设置。 通常您可以在**application.properties**文件中定义属性。

常用服务器设置包括：

- 网络设置：侦听端口的HTTP请求（server.port），接口地址绑定到server.address等。
- 会话设置：会话是否持久化（server.session.persistence），会话超时（server.session.timeout），会话数据的位置（server.session.store-dir）和session-cookie配置（server.session.cookie.*）。
- 错误管理：错误页面的位置（server.error.path）等
- [SSL](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#howto-configure-ssl)
- [HTTP压缩](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#how-to-enable-http-response-compression)

Spring Boot尽可能地尝试公开常见设置，但并不总是可能的。 对于这些情况，专用命名空间提供服务器特定的定制（请参阅server.tomcat和server.undertow）。 例如，可以使用嵌入式servlet容器的特定功能来配置[访问日志](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#howto-configure-accesslogs)。

**有关完整列表，请参阅[ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)类。**

> 用程序定制

如果需要以编程方式配置嵌入式servlet容器，您可以注册一个实现**EmbeddedServletContainerCustomizer**接口的Spring bean。 **EmbeddedServletContainerCustomizer**提供对**ConfigurableEmbeddedServletContainer**的访问，其中包含许多自定义设置方法。

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

	@Override
	public void customize(ConfigurableServletWebServerFactory server) {
		server.setPort(9000);
	}

}
```

**TomcatServletWebServerFactory、JettyServletWebServerFactory和UndertowServletWebServerFactory都是ConfigurableServletWebServerFactory的专用变体，它们分别为Tomcat、Jetty和Undertow提供了额外的定制设置方法。**

> 直接自定义ConfigurableEmbeddedServletContainer

如果前面的定制技术太有限，你可以注册**TomcatServletWebServerFactory**，**JettyServletWebServerFactory**或**UndertowServletWebServerFactory** bean。

```java
@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
	TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
	factory.setPort(9000);
	factory.setSessionTimeout(10, TimeUnit.MINUTES);
	factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
	return factory;
}
```

setter方法提供了许多配置选项。 如果您需要做更多的自定义，还会提供几种保护方法“钩子”。 有关详细信息，请参阅[源代码文档](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/api/org/springframework/boot/web/servlet/server/ConfigurableServletWebServerFactory.html)。

> JSP限制

运行使用嵌入式servlet容器的Spring Boot应用程序（并打包为可执行存档）时，JSP支持存在一些限制。

- 可以使用Tomcat和war包，即可执行的war将会起作用，并且也可以部署到标准容器（不限于但包括Tomcat）中。 由于Tomcat中的硬编码文件模式，可执行的jar将无法正常工作。
- Undertow不支持JSP。
- 创建自定义的error.jsp页面将不会覆盖默认视图以进行错误处理，而应使用自定义错误页面。

有一个[JSP示例](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-samples/spring-boot-sample-web-jsp)，以便您可以看到如何设置。



