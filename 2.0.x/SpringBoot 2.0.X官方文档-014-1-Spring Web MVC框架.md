# SpringBoot 2.0.X官方文档-014-1-Spring Web MVC框架

[Spring Web MVC](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#mvc)框架（通常简称为“Spring MVC”）是一个丰富的(模型-视图-控制)Web框架。Spring MVC允许您创建特殊**@Controller**或**@RestController** bean来处理传入的HTTP请求。控制器中的方法通过使用**@RequestMapping**注解映射到HTTP 。

下面的代码显示了一个典型的提供JSON数据的**@RestController**:

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

	@RequestMapping(value="/{user}", method=RequestMethod.GET)
	public User getUser(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
	List<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}", method=RequestMethod.DELETE)
	public User deleteUser(@PathVariable Long user) {
		// ...
	}

}
```

Spring MVC是核心Spring Framework的一部分，详细信息可在[参考文档](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#mvc)中找到。也有涵盖Spring MVC中提供一些指南[spring.io/guides](https://spring.io/guides)。

> Spring MVC自动配置

Spring Boot为Spring MVC提供自动配置，适用于大多数应用程序。

自动配置在Spring的默认值之上添加了以下功能：

- 包含**ContentNegotiatingViewResolver**和**BeanNameViewResolver** beans
- 支持提供静态资源，包括对WebJars的支持（ 本文档稍后介绍）
- 自动注册**Converter**，**GenericConverter**和**Formatter** beans
- 支持**HttpMessageConverters**（ 本文档后面部分）
- 自动注册**MessageCodesResolver**（ 本文档后面部分）
- 支持静态**index.html**
- 支持自定义**Favicon**
- 自动使用**ConfigurableWebBindingInitializer** bean

如果您想保留Spring Boot MVC功能并且想要添加其他[MVC配置](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#mvc)（拦截器，格式化程序，视图控制器和其他功能），您可以添加自己的**@Configuration**类，类型为**WebMvcConfigurer**，但没有**@EnableWebMvc**。如果希望提供**RequestMappingHandlerMapping**、**RequestMappingHandlerAdapter**或**ExceptionHandlerExceptionResolver**的定制实例，可以声明一个**WebMvcRegistrationsAdapter**实例来提供这些组件。

如果您想完全控制Spring MVC，可以添加自己的**@EnableWebMvc**注解的**@Configuration**。

> HttpMessageConverters

Spring MVC使用**HttpMessageConverter**接口转换HTTP请求和响应。默认值包含在开箱即用中。例如，对象可以自动转换为JSON（通过使用Jackson库）或XML（如果可用，则使用Jackson XML扩展，或者如果Jackson XML扩展不可用，则使用JAXB）。默认情况下，字符串编码为**UTF-8**。

如果需要添加或自定义转换器，可以使用Spring Boot的 **HttpMessageConverters**类：：

```java
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration
public class MyConfiguration {

	@Bean
	public HttpMessageConverters customConverters() {
		HttpMessageConverter<?> additional = ...
		HttpMessageConverter<?> another = ...
		return new HttpMessageConverters(additional, another);
	}

}
```

**HttpMessageConverter**上下文中存在的任何bean都将添加到转换器列表中。您也可以以相同的方式覆盖默认转换器。

> 自定义JSON序列化程序和反序列化程序

如果您使用Jackson来序列化和反序列化JSON数据，您可能希望编写自己的类**JsonSerializer**和**JsonDeserializer**类。自定义序列化程序通常 通过模块向[Jackson注册](http://wiki.fasterxml.com/JacksonHowToCustomDeserializers)，但Spring Boot提供了另一种**@JsonComponent**注解，可以更容易地直接注册Spring Beans。

您可以直接在**JsonSerializer**或**JsonDeserializer**实现上使用**@JsonComponent**注解。您还可以在包含序列化/反序列化器的类中使用它作为内部类，如下例所示:

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

	public static class Serializer extends JsonSerializer<SomeObject> {
		// ...
	}

	public static class Deserializer extends JsonDeserializer<SomeObject> {
		// ...
	}

}
```

ApplicationContext中的所有**@JsonComponent** bean都自动注册到Jackson。因为**@JsonComponent**是由**@Component**注解的，所以通常使用组件扫描规则。

Spring Boot还提供了[JsonObjectSerializer](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectSerializer.java)和[JsonObjectDeserializer](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectDeserializer.java)基类，这些基类在序列化对象时为标准Jackson版本提供了有用的替代方法。有关详细信息，请参阅Javadoc中的[JsonObjectSerializer](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/api/org/springframework/boot/jackson/JsonObjectSerializer.html)和[JsonObjectDeserializer](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/api/org/springframework/boot/jackson/JsonObjectDeserializer.html)。

> MessageCodesResolver

Spring MVC有一个生成错误代码的策略，用于从绑定错误中呈现错误消息：**MessageCodesResolver**。如果设置了**spring.mvc.message-codes-resolver.format**属性**PREFIX_ERROR_CODE**或 **POSTFIX_ERROR_CODE**,Spring Boot为您创建一个(参见[DefaultMessageCodesResolver.Format](https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.Format.html)中的枚举)。

> 静态内容

默认情况下，Spring Boot从类路径中的**/static**（**/public**或**/resources**或/**META-INF/resources**）目录或者根目录中提供静态内容**ServletContext**。它使用来自Spring MVC的**ResourceHttpRequestHandler**，这样您就可以通过添加自己的**WebMvcConfigurer**并覆盖**addResourceHandlers**方法来修改这种行为。

在独立的Web应用程序中，来自容器的默认servlet也被启用，并且作为后备，如果Spring决定不处理它，则从ServletContext的根目录提供内容。 大多数情况下，这不会发生（除非您修改默认的MVC配置），因为Spring将始终能够通过DispatcherServlet处理请求。

默认情况下，会映射资源**/\*\***，但您可以使用**spring.mvc.static-path-pattern**属性对其进行调整 。例如，**/resources/\*\***可以按如下方式重新定位所有资源 ：

```bash
spring.mvc.static-path-pattern=/resources/**
```

您还可以使用**spring.resources.static-locations**属性自定义静态资源位置（将默认值替换为目录位置）。根Servlet上下文路径"/"也会自动添加为位置。

除了前面提到的“标准”静态资源位置之外，[Webjars内容](https://www.webjars.org/)还有一个特殊的例子。任何在**/webjars/****中具有路径的资源都可以从jar文件中获取，如果它们被打包成webjars格式的话。

**如果您的应用程序将被打包为jar，请不要使用src/main/webapp目录。 虽然这个目录是一个通用的标准，但它只适用于war包，如果生成一个jar，它将被大多数构建工具忽略。**

Spring Boot还支持Spring MVC提供的高级资源处理功能，允许使用缓存静态资源或使用与Webjars的版本无关的URL。

要为Webjars使用版本无关的URL，请添加**webjars-locator-core**依赖项。然后声明你的Webjar。以jQuery为例，添加"/webjars/jquery/jquery.min.js"结果"/webjars/jquery/x.y.z/jquery.min.js"。x.y.z是Webjar版本。

**如果你使用JBoss，你需要声明webjars-locator-jboss-vfs依赖而不是webjars-locator-core。否则，所有Webjars都会解析为404。**

要使用缓存清除，以下配置会为所有静态资源配置缓存清除解决方案，从而有效地在URL**\<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>**中添加内容哈希，例如 ：

```bash
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

**由于ResourceUrlEncodingFilter为Thymeleaf和FreeMarker自动配置了资源链接，因此在运行时会在模板中重写这些链接 。您应该在使用JSP时手动声明此过滤器。其他模板引擎目前不是自动支持的，但可以使用自定义模板宏/帮助程序和使用[ResourceUrlProvider](https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/org/springframework/web/servlet/resource/ResourceUrlProvider.html)。**

使用JavaScript模块加载器动态加载资源时，不能重命名文件。这就是为什么其他策略也得到支持并可以合并的原因。“fixed”策略在URL中添加静态版本字符串而不更改文件名，如以下示例所示：

```bash
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

使用此配置，JavaScript模块位于"**/js/lib/**"使用固定版本控制策略（**/v12/js/lib/mymodule.js**），而其他资源仍使用内容one（**\<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/\>**）。

有关更多支持的选项，请参阅[ResourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java)。

**此功能已在专门的[博客文章](https://spring.io/blog/2014/07/24/spring-framework-4-1-handling-static-web-resources)和Spring Framework的[参考文档](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#mvc-config-static-resources)中进行了详细描述 。**

> 欢迎页面

Spring Boot支持静态和模板化欢迎页面。首先在配置的静态内容位置中查找**index.html**文件。如果找不到，则查找index模板。如果找到任何一个，它将自动用作应用程序的欢迎页面。

> 自定义Favicon

Spring Boot在配置的静态内容位置和类路径的根目录（按顺序）中查找favicon.ico。 如果文件存在，它将被自动用作应用程序的图标。

> 路径匹配和内容协商

Spring MVC可以通过查看请求路径并将其与应用程序中定义的映射匹配(例如，控制器方法上的**@GetMapping**注解)，将传入的HTTP请求映射到处理程序。

Spring Boot默认选择禁用后缀模式匹配，这意味着请求"**GET /projects/spring-boot.json**"不会与**@GetMapping("/projects/spring-boot")**映射匹配 。这被认为是[Spring MVC应用程序的最佳实践](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)。对于没有发送正确“接受”请求标头的HTTP客户端，此功能在过去主要有用; 我们需要确保将正确的内容类型发送给客户端。如今，内容协商更加可靠。

还有其他方法可以处理不一致发送正确“接受”请求标头的HTTP客户端。我们可以使用查询参数来确保将请求"**GET /projects/spring-boot?format=json**" 映射到**@GetMapping("/projects/spring-boot")**以下内容，而不是使用后缀匹配：

```bash
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

如果您了解警告并仍希望您的应用程序使用后缀模式匹配，则需要以下配置：

```bash
spring.mvc.contentnegotiation.favor-path-extension=true

# You can also restrict that feature to known extensions only
# spring.mvc.pathmatch.use-registered-suffix-pattern=true

# We can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```

> ConfigurableWebBindingInitializer

Spring MVC使用**WebBindingInitializer**为特定请求初始化**WebDataBinder**。 如果您用**@Bean**创建自己的**ConfigurableWebBindingInitializer** bean，Spring Boot将自动配置Spring MVC以使用它。

> 模板引擎

除REST Web服务外，您还可以使用Spring MVC来提供动态HTML内容。Spring MVC支持各种模板技术，包括Thymeleaf，FreeMarker和JSP。此外，许多其他模板引擎包括他们自己的Spring MVC集成。

Spring Boot包含对以下模板引擎的自动配置支持：

- [FreeMarker](https://freemarker.org/docs/)
- [Groovy](http://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)
- [Thymeleaf](http://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

**如果可能，应避免使用JSP，当使用嵌入式servlet容器时，JSP有[几个已知的限制](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-jsp-limitations)。**

当您使用其中一个模板引擎和默认配置时，您的模板将自动从**src/main/resources/templates**中获取。

**根据您运行应用程序的方式，IntelliJ IDEA以不同方式对类路径进行排序。从主方法在IDE中运行应用程序会产生与使用Maven或Gradle或其打包的jar运行应用程序时不同的顺序。这可能导致Spring Boot无法在类路径中找到模板。如果遇到此问题，可以在IDE中重新排序类路径，以便首先放置模块的类和资源。或者，您可以配置模板前缀以搜索templates类路径上的每个目录，如下所示： classpath*:/templates/。**

###  错误处理

默认情况下，Spring Boot提供/error映射，以合理的方式处理所有错误，并在servlet容器中注册为“global”错误页面。 对于机器客户端，它将产生JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。 对于浏览器客户端，有一个’whitelabel’错误视图，以HTML格式呈现相同的数据（定制它只需添加一个解析“error”的视图）。 要完全替换默认行为，您可以实现**ErrorController**并注册该类型的bean定义，或者简单地添加一个类型为**ErrorAttributes**的bean来使用现有机制，但只是替换内容。

**在BasicErrorController可以用作自定义基类 ErrorController。如果要为新内容类型添加处理程序（默认情况下是text/html专门处理并为其他所有内容提供后备），这将特别有用。为此，请扩展BasicErrorController，添加@RequestMapping具有produces属性的公共方法 ，并创建新类型的bean。**

您还可以定义一个**@ControllerAdvice**来自定义为特定控制器 and/or 异常类型返回的JSON文档。

```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

	@ExceptionHandler(YourException.class)
	@ResponseBody
	ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
	}

	private HttpStatus getStatus(HttpServletRequest request) {
		Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
		if (statusCode == null) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
		return HttpStatus.valueOf(statusCode);
	}

}
```

在上面的示例中，如果由AcmeController在同一个包中定义的控件抛出了YourException，则将使用CustomerErrorType POJO的json表示法而不是ErrorAttributes表示形式。

> 自定义错误页面

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到**/error**文件夹。错误页面可以是静态HTML（即，添加到任何静态资源文件夹下），也可以使用模板构建。文件名应该是确切的状态代码或系列掩码。

例如，要映射404到静态HTML文件，您的文件夹结构如下所示：

```bash
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

使用FreeMarker模板映射5xx所有错误，您的文件夹结构如下：

```bash
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftl
             +- <other templates>
```

对于更复杂的映射，您还可以添加实现**ErrorViewResolver**接口的bean ，如以下示例所示：

```java
public class MyErrorViewResolver implements ErrorViewResolver {

	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request,
			HttpStatus status, Map<String, Object> model) {
		// Use the request or status to optionally return a ModelAndView
		return ...
	}

}
```

> 映射Spring MVC之外的错误页面

对于不使用Spring MVC的应用程序，可以使用**ErrorPageRegistrar**接口直接注册**ErrorPages**。这种抽象直接与底层嵌入式servlet容器一起工作，即使你没有Spring MVC DispatcherServlet也可以工作。

```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
	return new MyErrorPageRegistrar();
}

// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

	@Override
	public void registerErrorPages(ErrorPageRegistry registry) {
		registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
	}

}
```

**如果您注册一个最终由Filter过滤的路径的ErrorPage（例如，像一些非Spring Web框架，例如Jersey和Wicket一样），则必须将Filter显式注册为ERROR dispatcher，例如。**

```java
@Bean
public FilterRegistrationBean myFilter() {
	FilterRegistrationBean registration = new FilterRegistrationBean();
	registration.setFilter(new MyFilter());
	...
	registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
	return registration;
}
```

请注意，默认值**FilterRegistrationBean**不包括**ERROR**调度程序类型。

小心：当部署到servlet容器时，Spring Boot使用其错误页面过滤器将具有错误状态的请求转发到相应的错误页面。如果尚未提交响应，则只能将请求转发到正确的错误页面。缺省情况下，WebSphere Application Server 8.0及更高版本在成功完成servlet的服务方法后提交响应。您应该通过设置**com.ibm.ws.webcontainer.invokeFlushAfterService**为**false**禁用此行为。

> Spring HATEOAS

如果您正在开发一种利用超媒体的RESTful API，Spring Boot可以为Spring HATEOAS提供自动配置，适用于大多数应用程序。 自动配置取代了使用**@EnableHypermediaSupport**的需求，并注册了一些Bean，以便轻松构建基于超媒体的应用程序，包括LinkDiscoverers（用于客户端支持）和配置为将响应正确地组织到所需表示中的ObjectMapper。 ObjectMapper将根据**spring.jackson.***属性或**Jackson2ObjectMapperBuilder** bean（如果存在）进行自定义。

您可以使用**@EnableHypermediaSupport**控制Spring HATEOAS配置。 请注意，这将禁用上述**ObjectMapper**定制。

> CORS支持

[跨域资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是[大多数浏览器](https://caniuse.com/#feat=cors)实现的[W3C规范](https://www.w3.org/TR/cors/)，允许您以灵活的方式指定授权何种跨域请求，而不是使用一些安全性较低且功能较弱的方法，如IFRAME或JSONP。

从版本4.2起，Spring MVC[支持CORS](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#cors)开箱即用。 在Spring Boot应用程序中的controller方法使用[@CrossOrigin注解](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#controller-method-cors-configuration)的CORS配置不需要任何特定的配置。 可以通过使用自定义的addCorsMappings(CorsRegistry)方法注册WebMvcConfigurer bean来定义[全局CORS配置](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#global-cors-configuration)：

```java
@Configuration
public class MyConfiguration {

	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurer() {
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/api/**");
			}
		};
	}
}
```


