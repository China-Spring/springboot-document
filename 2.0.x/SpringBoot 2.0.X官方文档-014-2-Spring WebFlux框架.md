# SpringBoot 2.0.X官方文档-014-2-Spring WebFlux框架

Spring WebFlux是Spring Framework 5.0中引入的新的反应式Web框架。与Spring MVC不同，它不需要Servlet API，完全异步且无阻塞，并通过[Reactor项目](https://projectreactor.io/)实现[Reactive Streams](http://www.reactive-streams.org/)规范。

Spring WebFlux有两种风格:功能性的和基于注解的。基于注解的方法非常接近Spring MVC模型，如下例所示:

```java
@RestController
@RequestMapping("/users")
public class MyRestController {

	@GetMapping("/{user}")
	public Mono<User> getUser(@PathVariable Long user) {
		// ...
	}

	@GetMapping("/{user}/customers")
	public Flux<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}

	@DeleteMapping("/{user}")
	public Mono<User> deleteUser(@PathVariable Long user) {
		// ...
	}

}
```

“WebFlux.fn "是一种功能变体，它将路由配置与请求的实际处理分离开来，如下例所示:

```java
@Configuration
public class RoutingConfiguration {

	@Bean
	public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
		return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
				.andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
				.andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
	}

}

@Component
public class UserHandler {

	public Mono<ServerResponse> getUser(ServerRequest request) {
		// ...
	}

	public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
		// ...
	}

	public Mono<ServerResponse> deleteUser(ServerRequest request) {
		// ...
	}
}
```

WebFlux是Spring Framework的一部分，详细信息可在其[参考文档](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn)中找到。

**您可以定义任意数量的RouterFunction bean来模块化路由器的定义。如果需要应用优先级，可以对bean进行排序。**

要开始，请将**spring-boot-starter-webflux**模块添加到您的应用程序中。

**在应用程序中同时添加spring-boot-starter-web和spring-boot-starter-webflux模块，会导致Spring Boot自动配置Spring MVC，而不是WebFlux。之所以选择这种行为，是因为许多Spring开发人员在其Spring MVC应用程序中添加了spring-boot-starter-webflux来使用反应性WebClient。您仍然可以通过将选择的应用程序类型设置为SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE)来强制执行您的选择。**

> Spring WebFlux自动配置

Spring Boot为Spring WebFlux提供自动配置，适用于大多数应用程序。

自动配置在Spring的默认值之上添加了以下功能：

- 为**HttpMessageReader**和**HttpMessageWriter**实例配置编解码器（ 本文档稍后将介绍）。
- 支持提供静态资源，包括对WebJars的支持（ 本文档后面将介绍）。

如果您想保留Spring引导WebFlux的特性，并且想要添加额外的[WebFlux配置](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/web.html#web-reactive)，您可以添加自己的**@Configuration**类，类型是**WebFluxConfigurer**，但是没有**@EnableWebFlux**。

如果您想完全控制Spring WebFlux，您可以添加您自己的@Configuration注解的@EnableWebFlux。

> 带有HttpMessageReaders和HttpMessageWriters的HTTP编解码器

Spring WebFlux使用**HttpMessageReader**和**HttpMessageWriter**接口来转换HTTP请求和响应。通过查看类路径中可用的库，CodecConfigurer将它们配置为具有合理的默认设置。

Spring Boot通过使用CodecCustomizer实例进一步自定义。例如，**spring.jackson.***配置密钥应用于Jackson编解码器。

如果需要添加或自定义编解码器，可以创建自定义**CodecCustomizer**组件，如以下示例所示：

```java
import org.springframework.boot.web.codec.CodecCustomizer;

@Configuration
public class MyConfiguration {

	@Bean
	public CodecCustomizer myCodecCustomizer() {
		return codecConfigurer -> {
			// ...
		}
	}

}
```

您还可以利用Boot的自定义JSON序列化程序和反序列化程序。

> 静态内容

默认情况下，Spring Boot从类路径中的**/static**（**/public**或**/resources**或/**META-INF/resources**）目录或者根目录中提供静态内容**ServletContext**。它使用来自Spring MVC的**ResourceHttpRequestHandler**，这样您就可以通过添加自己的**WebFluxConfigurer**并覆盖**addResourceHandlers**方法来修改这种行为。

默认情况下，会映射资源**/\*\***，但您可以使用**spring.webflux.static-path-pattern**属性对其进行调整 。例如，**/resources/\*\***可以按如下方式重新定位所有资源 ：

```bash
spring.webflux.static-path-pattern=/resources/**
```

您还可以使用自定义静态资源位置**spring.resources.static-locations**。这样做会将默认值替换为目录位置。如果这样做，默认的欢迎页面检测会切换到您的自定义位置。因此，如果您的任何位置在启动时**index.html**存在，则它是应用程序的主页。

除了前面提到的“标准”静态资源位置之外，[Webjars内容](https://www.webjars.org/)还有一个特殊的例子。任何在**/webjars/****中具有路径的资源都可以从jar文件中获取，如果它们被打包成webjars格式的话。

**Spring WebFlux应用程序并不严格依赖于Servlet API，因此它们不能作为war文件部署，也不能使用src/main/webapp目录。**

> 模板引擎

除REST Web服务外，您还可以使用Spring WebFlux来提供动态HTML内容。Spring WebFlux支持各种模板技术，包括Thymeleaf，FreeMarker和Mustache。

- [FreeMarker](https://freemarker.org/docs/)
- [Thymeleaf](http://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

### 错误处理

Spring Boot提供了一个**WebExceptionHandler**，以一种合理的方式处理所有错误。它在处理顺序中的位置在WebFlux提供的处理器之前，WebFlux被认为是最后一个。对于机器客户机，它生成一个JSON响应，其中包含错误、HTTP状态和异常消息的详细信息。对于浏览器客户机，有一个“whitelabel”错误处理程序，它以HTML格式呈现相同的数据。您还可以提供自己的HTML模板来显示错误(参见下一节)。

自定义此功能的第一步通常涉及使用现有机制，但替换或扩充错误内容。为此，您可以添加**ErrorAttributes**类型的bean.

要更改错误处理行为，可以实现**ErrorWebExceptionHandler**并注册该类型的bean定义。因为**WebExceptionHandler**非常低级，所以Spring Boot也提供了一个方便的**AbstractErrorWebExceptionHandler**，您可以用WebFlux函数的方式处理错误，如下例所示:

```java
public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {

	// Define constructor here

	@Override
	protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {

		return RouterFunctions
				.route(aPredicate, aHandler)
				.andRoute(anotherPredicate, anotherHandler);
	}

}
```

为了获得更完整的描述，您还可以直接子类化**DefaultErrorWebExceptionHandler**并覆盖特定的方法。

> 自定义错误页面

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到**/error**文件夹。错误页面可以是静态HTML（即，添加到任何静态资源文件夹下）或使用模板构建。文件名应该是确切的状态代码或系列掩码。

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

> Web过滤器

Spring WebFlux提供了一个**WebFilter**接口，可以实现它来过滤HTTP请求-响应交换。应用程序上下文中找到的WebFilter bean将自动用于筛选每个交换。

在过滤器的顺序很重要的地方，它们可以实现order或用**@Order**注解。Spring Boot自动配置可以为您配置Web过滤器。执行此操作时，将使用下表所示的order:

|Web Filter|Order|
|---|---|
|MetricsWebFilter|Ordered.HIGHEST_PRECEDENCE + 1|
|WebFilterChainProxy (Spring Security)|-100|
|HttpTraceWebFilter|Ordered.LOWEST_PRECEDENCE - 10|

