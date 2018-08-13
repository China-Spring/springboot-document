# SpringBoot 2.0.X官方文档-010-SpringApplication

SpringApplication类提供了一种简便的方式来从main()方法引导启动****的Spring应用程序。在许多情况下，您可以委托给静态SpringApplication.run方法，如下例所示:

```java
public static void main(String[] args) {
	SpringApplication.run(MySpringConfiguration.class, args);
}
```

当您的应用程序启动时，您应该看到类似于以下输出的内容：

```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.0.4.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

默认情况下，会显示日志**INFO**日志消息，包括一些相关的启动详细信息，例如启动应用程序的用户。如果您需要其他日志级别，则可以进行设置，详见 [SpringBoot 2.0.X官方文档-013-日志记录](http://www.springall.com.cn/article/63)

> 启动失败

如果您的应用程序启动失败，注册的故障分析(FailureAnalyzers)程序将有机会提供专用的错误消息和解决问题的具体操作。例如，如果您在端口8080上启动一个web应用程序，并且该端口已经在使用，那么您应该看到以下消息类似的内容:

```bash
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

**Spring Boot提供了许多FailureAnalyzer实现，您可以[添加自己的实现](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#howto-failure-analyzer)。**

如果没有故障分析器能够处理异常，您仍然可以显示完整的情况报告，以便更好地理解错误。为此，需要启用debug属性或为**org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener**启用调试日志记录。

例如，如果您使用**java -jar**运行应用程序，则可以按如下方式启用**debug**属性：

```bash
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

> 自定义横幅(Banner)

可以通过在您的类路径中添加一个 banner.txt 文件，或者将banner.location设置到banner文件的位置来更改启动时打印的banner。 如果文件有一些不常用的编码，你可以设置banner.charset（默认为UTF-8）。除了文本文件，您还可以将banner.gif，banner.jpg或banner.png图像文件添加到您的类路径中，或者设置一个banner.image.location属性。 图像将被转换成ASCII艺术表现，并打印在任何文字banner上方。

您可以在banner.txt文件中使用以下占位符：

|变量|描述|
|---|---|
|${application.version}|在MANIFEST.MF中声明的应用程序的版本号。例如， Implementation-Version: 1.0 被打印为 1.0|
|${application.formatted-version}|在MANIFEST.MF中声明的应用程序版本号的格式化显示（用括号括起来，以v为前缀）。 例如 (v1.0)。|
|${spring-boot.version}|您正在使用的Spring Boot版本。 例如2.0.4.RELEASE|
|${spring-boot.formatted-version}|您正在使用的Spring Boot版本，格式化显示（用括号括起来并带有前缀v）。例如(v2.0.4.RELEASE)。|
|${Ansi.NAME} (${AnsiColor.NAME}, ${AnsiBackground.NAME}, ${AnsiStyle.NAME})|其中NAME是ANSI转义码的名称。 有关详细信息，请参阅 [AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v1.5.2.RELEASE/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)。|
|${application.title}|您的应用程序的标题在MANIFEST.MF中声明。 例如Implementation-Title：MyApp打印为MyApp。|

**如果要以编程方式生成banner，则可以使用SpringApplication.setBanner()方法。 使用org.springframework.boot.Banner 如接口，并实现自己的printBanner() 方法。**

您还可以使用spring.main.banner-mode属性来决定是否必须在System.out（控制台）上打印banner，使用配置的logger（log）或不打印（off）。

打印的横幅被注册为一个单例bean，名称如下:springBootBanner。

YAML映射为false，所以如果要禁用应用程序中的banner，请务必添加引号，如下例所示:

```yml
spring:
	main:
		banner-mode: "off"
```

> 自定义SpringApplication

如果SpringApplication默认值不符合您的需要，您可以改为创建本地实例并对其进行自定义。例如，要关闭banner，您可以写：

```java
public static void main(String[] args) {
	SpringApplication app = new SpringApplication(MySpringConfiguration.class);
	app.setBannerMode(Banner.Mode.OFF);
	app.run(args);
}
```

**传递给SpringApplication的构造函数参数是spring bean的配置源。 在大多数情况下，这些将引用@Configuration类，但它们也可以引用XML配置或应扫描的包。**

也可以使用application.properties文件配置SpringApplication。 有关详细信息，请参见[SpringBoot 2.0.X官方文档-011-外部配置](http://www.springall.com.cn/article/61)。

有关配置选项的完整列表，请参阅[SpringApplication Javadoc](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/api/org/springframework/boot/SpringApplication.html)。

> Fluent Builder API

如果您需要构建一个ApplicationContext层次结构（具有父/子关系的多个上下文），或者如果您只想使用“流式（fluent）”构建器API，则可以使用SpringApplicationBuilder。

SpringApplicationBuilder允许您将多个方法调用链接在一起，并包含父方法和子方法，它们允许您创建层次结构，如下例所示:

```java
new SpringApplicationBuilder()
		.sources(Parent.class)
		.child(Application.class)
		.bannerMode(Banner.Mode.OFF)
		.run(args);
```

**创建ApplicationContext层次结构时存在一些限制。例如，Web组件必须包含在子上下文中，并且Environment父组件和子上下文都使用相同的 组件。有关详细信息，请参阅[SpringApplicationBuilder Javadoc](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/api/org/springframework/boot/builder/SpringApplicationBuilder.html)。**

> Application events and listeners

除了通常的Spring Framework事件之外，例如 [ContextRefreshedEvent](https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)，SpringApplication发送一些额外的应用程序事件。

**在创建ApplicationContext之前，实际上触发了一些事件，因此您不能在@Bean上注册一个监听器。 您可以通过SpringApplication.addListeners(…) 或SpringApplicationBuilder.listeners(…)方法注册它们。<br/>
如果您希望自动注册这些侦听器，无论创建应用程序的方式如何，都可以将META-INF / spring.factories文件添加到项目中，并使用org.springframework.context.ApplicationListener引用您的侦听器。
org.springframework.context.ApplicationListener=com.example.project.MyListener**

当应用程序运行时，应用程序事件按以下顺序发送:

1. ApplicationStartingEvent在运行开始时发送，但在任何处理之前发送，除了侦听器和初始化器的注册之外。
2. 当要在上下文中使用的环境已知但在创建上下文之前已知时，将发送ApplicationEnvironmentPreparedEvent。
3. ApplicationPreparedEvent是在刷新启动之前发送的，但在bean定义加载之后。
4. 一个ApplicationStartedEvent是在上下文刷新之后，但在调用任何应用程序和命令行运行程序之前发送的。
5. 在调用任何应用程序和命令行运行程序之后发送ApplicationReadyEvent。它表明应用程序已经准备好为请求提供服务。
6. 如果启动时出现异常，将发送ApplicationFailedEvent。

**一般您不需要使用应用程序事件，但可以方便地知道它们存在。 在内部，Spring Boot使用事件来处理各种任务。**

应用程序事件通过Spring框架的事件发布机制发送。这种机制的一部分确保在子上下文中发布给侦听器的事件也在任何上下文中发布给侦听器。因此，如果您的应用程序使用SpringApplication实例的层次结构，那么侦听器可能会接收到相同类型的应用程序事件的多个实例。

为了让侦听器区分上下文事件和子代上下文事件，它应该请求注入其应用程序上下文，然后将注入的上下文与事件上下文进行比较。可以通过实现**ApplicationContextAware**或(如果侦听器是bean)使用**@Autowired**来注入上下文。

> Web 环境

SpringApplication尝试为您创建正确类型的ApplicationContext。用于确定WebApplicationType的算法相当简单:

- 如果有Spring MVC，就会使用**AnnotationConfigServletWebServerApplicationContext**
- 如果没有Spring MVC，并且有Spring WebFlux，那么就使用**AnnotationConfigReactiveWebServerApplicationContext**
- 否则,使用**AnnotationConfigApplicationContext**

这意味着如果您在同一个应用程序中使用Spring MVC和Spring WebFlux的新WebClient, Spring MVC将被默认使用。您可以通过调用setWebApplicationType(WebApplicationType)来轻松地覆盖它。

也可以完全控制调用setApplicationContextClass(…)时使用的ApplicationContext类型。

**在JUnit测试中使用SpringApplication时，通常需要调用setWebApplicationType(WebApplicationType.NONE)。**

> 访问应用程序参数

如果需要访问传递给SpringApplication.run(…)的应用程序参数，可以注入org.springframework.boot。ApplicationArguments bean。ApplicationArguments接口提供了对原始字符串[]参数以及解析选项和非选项参数的访问，如下例所示:

```java
import org.springframework.boot.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.stereotype.*;

@Component
public class MyBean {

	@Autowired
	public MyBean(ApplicationArguments args) {
		boolean debug = args.containsOption("debug");
		List<String> files = args.getNonOptionArgs();
		// if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
	}

}
```

**Spring Boot还在Spring环境中注册了CommandLinePropertySource。这允许您通过使用@Value注释注入单个应用程序参数。**

> 使用ApplicationRunner或CommandLineRunner

如果您需要在SpringApplication启动后运行一些特定的代码，您可以实现ApplicationRunner或CommandLineRunner接口。这两个接口以相同的方式工作，并提供一个运行方法，该方法在SpringApplication.run(…)完成之前调用。

CommandLineRunner接口以简单的字符串数组的形式提供对应用程序参数的访问，而ApplicationRunner使用前面讨论的ApplicationArguments接口。下面的例子展示了一个带有run方法的CommandLineRunner:

```java
import org.springframework.boot.*;
import org.springframework.stereotype.*;

@Component
public class MyBean implements CommandLineRunner {

	public void run(String... args) {
		// Do something...
	}

}
```

如果定义了几个CommandLineRunner或ApplicationRunner bean，必须按特定顺序调用，则可以另外实现org.springframework.core.Ordered的接口或使用org.springframework.core.annotation.Order 注解。

> Application退出

每个SpringApplication都向JVM注册一个关闭钩子，以确保ApplicationContext在退出时优雅地关闭。所有标准的Spring生命周期回调(例如DisposableBean接口或@PreDestroy注释)都可以使用。

此外，bean可以实现**org.springframework.boot.ExitCodeGenerator**接口(如果它们希望在调用SpringApplication.exit()时返回特定的退出代码)。然后，可以将此退出代码传递给System.exit()，以将其作为状态代码返回，如下例所示:

```java
@SpringBootApplication
public class ExitCodeApplication {

	@Bean
	public ExitCodeGenerator exitCodeGenerator() {
		return () -> 42;
	}

	public static void main(String[] args) {
		System.exit(SpringApplication
				.exit(SpringApplication.run(ExitCodeApplication.class, args)));
	}

}
```

另外，ExitCodeGenerator接口也可以由异常实现。当遇到这种异常时，Spring Boot返回由实现的getExitCode()方法提供的退出代码。

> 管理功能

通过指定**spring.application.admin.enabled**可以为应用程序启用管理相关的特性。这将在MBeanServer平台上公开SpringApplicationAdminMXBean。您可以使用这个特性远程管理Spring应用程序。这个特性对于任何服务包装器实现都是有用的。

**如果您想知道应用程序在哪个HTTP端口上运行，请使用local.server.port获取属性。**

**在启用这个特性时要小心，因为MBean公开了一个关闭应用程序的方法。**

