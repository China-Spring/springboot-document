# SpringBoot 2.0.X官方文档-013-日志记录

Spring Boot使用[Commons Logging](https://commons.apache.org/logging)进行所有内部日志记录，但保留底层日志实现。为[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)，[Log4J2](https://logging.apache.org/log4j/2.x/)和[Logback](http://logback.qos.ch/)提供了默认配置 。在每种情况下，记录器都预先配置为使用控制台输出，并且还提供可选的文件输出。

默认情况下，如果使用“Starters”，则使用Logback进行日志记录。还包括适当的Logback路由，以确保使用Java Util Logging，Commons Logging，Log4J或SLF4J的依赖库都能正常工作。

**Java有很多日志框架可供使用。如果以上列表看起来令人困惑，请不要担心。通常，您不需要更改日志记录依赖项，并且Spring Boot默认值可以正常工作。**

> 日志格式

Spring Boot的默认日志输出类似于以下示例：

```java
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

输出以下项目：

- 日期和时间 - 毫秒精度并且容易排序。
- 日志级别 - **ERROR**, **WARN**, **INFO**, **DEBUG**, **TRACE**.
- 进程ID。
- **---**分隔符来区分实际日志消息的开始。
- 线程名称 - 括在方括号中（可能会截断控制台输出）。
- 记录器名称 - 这通常是源类名（通常缩写）。
- 日志消息。

**Logback没有FATAL级别（对映ERROR）**

> 控制台输出

默认的日志配置会在控制台显示消息。 默认情况下会记录**ERROR**，**WARN**和**INFO**级别的消息。 您还可以通过**--debug**启动您的应用程序来启用“debug”模式。

```bash
$ java -jar myapp.jar --debug
```

**您还可以在application.properties中指定debug=true。**

当启用debug模式时，配置核心记录器（嵌入式容器，Hibernate和Spring Boot）的选择可以输出更多信息。 启用debug模式不会将应用程序配置为使用DEBUG级别记录所有消息。

或者，您可以使用**--trace**启动应用程序（或在您的**application.properties**中为**trace=true**）启用“trace”模式。 这将为核心记录器（嵌入式容器，Hibernate模式生成和整个Spring组合）启用trace日志。

> 日志颜色输出

如果您的终端支持ANSI，颜色输出可以增加可读性。 您可以将**spring.output.ansi.enabled**设置为支持的值来覆盖自动检测。

使用**％clr**关键字配置颜色编码。 在最简单的形式下，转换器将根据日志级别对输出进行着色，例如：

```bash
%clr(%5p)
```

下表描述了日志级别到颜色的映射：

|级别|颜色|
|---|---|
|FATAL|红色|
|ERROR|红色|
|WARN|黄色|
|INFO|绿色|
|DEBUG|绿色|
|TRACE|绿色|

或者，您可以通过将其作为转换选项指定应使用的颜色或样式。例如，要使文本变为黄色，请使用以下设置：

```bash
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和样式：

- blue
- cyan
- faint
- green
- magenta
- red
- yellow

> 文件输出

默认情况下，Spring Boot将仅将日志输出到控制台，不会写到文件。 如果要将控制台上的日志输出到日志文件，则需要设置**logging.file**或**logging.path**属性（例如在**application.properties**中）。

下表显示了如何logging.*一起使用这些属性：

表26.1 Logging属性

|logging.file|logging.path|实例|描述|
|---|---|---|---|
|(none)|(none)||仅控制台输出|
|Specific file|(none)|my.log|写入指定的日志文件。 名称可以是确切的位置或相对于当前目录。|
|(none)|Specific directory|/var/log|将spring.log写入指定的目录。 名称可以是确切的位置或相对于当前目录。|

日志文件在达到10MB时会轮换，并且与控制台输出一样，默认情况下会记录**ERROR**，**WARN**和**INFO**级别消息。可以使用**logging.file.max-siz**e属性更改大小限制。除非**logging.file.max-history**已设置属性，否则以前轮换的文件将无限期归档。

**日志记录系统在应用程序生命周期早期初始化，并且在通过@PropertySource注解加载的属性文件中将不会找到log属性。<br /><br />日志属性独立于实际的日志记录基础结构。 因此，特定配置key（如Logback的logback.configurationFile）不受Spring Boot管理。**

> 日志级别

所有支持的日志记录系统都可以在Spring Environment中设置log级别（例如在application.properties中），使用‘logging.level.*=LEVEL’，其中’LEVEL’是TRACE，DEBUG，INFO，WARN，ERROR，FATAL, OFF之一。 可以使用logging.level.root配置根记录器。 

示例application.properties：

```java
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

> 自定义日志配置

可以通过在类路径中包含适当的库来激活各种日志系统，并通过在类路径的根目录中提供合适的配置文件，或在Spring Environment属性logging.config指定的位置进一步配置。

您可以使用**org.springframework.boot.logging.LoggingSystem**系统属性强制Spring Boot使用特定的日志记录系统。该值应该是**LoggingSystem**实现的全名。 您还可以使用**none**值完全禁用Spring Boot的日志记录配置。

**由于在创建ApplicationContext之前初始化日志，因此无法在Spring @Configuration文件中控制@PropertySources的日志记录。 系统属性和常规的Spring Boot外部配置文件工作正常。**

根据您的日志记录系统，将会加载以下文件：

|Logging System|Customization|
|---|---|
|Logback|logback-spring.xml, logback-spring.groovy, logback.xml or logback.groovy|
|Log4j2|log4j2-spring.xml or log4j2.xml|
|JDK (Java Util Logging)|logging.properties|

**如果可能，我们建议您使用-spring变体进行日志记录配置（例如使用logback-spring.xml而不是logback.xml）。 如果使用标准配置位置，则Spring无法完全控制日志初始化。<br /><br />Java Util Logging存在已知的类加载问题，从“可执行jar”运行时会导致问题。 我们建议您尽可能避免。**

帮助定制一些其他属性从Spring环境转移到系统属性：

|Spring Environment|System Property|Comments|
|---|---|---|
|logging.exception-conversion-word|LOG_EXCEPTION_CONVERSION_WORD|记录异常时使用的转换字。|
|logging.file|LOG_FILE|如果已定义，则在默认日志配置中使用它。|
|logging.file.max-size|LOG_FILE_MAX_SIZE|最大日志文件大小（如果启用了LOG_FILE）。（仅支持默认的Logback设置。）|
|logging.file.max-history|LOG_FILE_MAX_HISTORY|要保留的最大归档日志文件数（如果启用了LOG_FILE）。（仅支持默认的Logback设置。）|
|logging.path|LOG_PATH|如果已定义，则在默认日志配置中使用它。|
|logging.pattern.console|CONSOLE_LOG_PATTERN|要在控制台上使用的日志模式（stdout）。（仅支持默认的Logback设置。）|
|logging.pattern.dateformat|LOG_DATEFORMAT_PATTERN|日志日期格式的Appender模式。（仅支持默认的Logback设置。）|
|logging.pattern.file|FILE_LOG_PATTERN|要在文件中使用的日志模式（如果LOG_FILE已启用）。（仅支持默认的Logback设置。）|
|logging.pattern.level|LOG_LEVEL_PATTERN|呈现日志级别时使用的格式（默认%5p）。（仅支持默认的Logback设置。）|
|PID|PID|当前进程ID（如果可能，则在未定义为OS环境变量时发现）。|

所有受支持的日志记录系统在解析其配置文件时都可以参考系统属性。有关**spring-boot.jar**示例，请参阅默认配置：

- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

**如果要在logging属性中使用占位符，则应使用[Spring Boot的语法](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-external-config-placeholders-in-properties)，而不是底层框架的语法。 值得注意的是，如果您使用Logback，您应该使用：作为属性名称与其默认值之间的分隔符，而不是:-。<br /><br />您可以通过覆盖LOG_LEVEL_PATTERN（或Logback的log.pattern.level）来添加MDC和其他ad-hoc内容到日志行。 例如，如果使用logging.pattern.level = user:％X {user}％5p，则默认日志格式将包含“user”的MDC条目（如果存在）。**

```bash
2015-09-30 12:30:04.031 user:someone INFO 22174 --- [  nio-8080-exec-0] demo.Controller
Handling authenticated request
```

> Logback 扩展

Spring Boot包含许多Logback扩展，可以帮助进行高级配置。您可以在**logback-spring.xml**配置文件中使用这些扩展名。

**您不能在标准logback.xml配置文件中使用扩展名，因为其加载时间太早。 您需要使用logback-spring.xml或定义logging.config属性。<br/><br/>扩展名不能与[Logback的配置扫描](http://logback.qos.ch/manual/configuration.html#autoScan)一起使用。 如果您尝试这样做，对配置文件进行更改将导致类似于以下记录之一的错误：**

```bash
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```

- 指定配置文件配置

**\<springProfile\>**标签允许您根据活动的Spring配置文件可选地包含或排除配置部分。 配置文件部分支持**\<configuration\>**元素在任何位置。 使用name属性指定哪个配置文件接受配置。 可以使用逗号分隔列表指定多个配置文件。以下清单显示了三个示例配置文件：

```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev, staging">
	<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
	<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

- 环境属性

标签允许您从Spring环境中显示属性，以便在Logback中使用。 如果您在logback中访问application.properties文件中的值，这将非常有用。 标签的工作方式与Logback标准的标签类似，但不是指定直接值，而是指定属性的来源（来自Environment）。 如果需要将属性存储在本地范围以外的位置，则可以使用scope属性。 如果在环境中未设置属性的情况下需要备用值，则可以使用defaultValue属性。

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
		defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
	<remoteHost>${fluentHost}</remoteHost>
	...
</appender>
```

**source必须在kebab示例中指定(例如my.property name)。但是，可以使用宽松的规则将属性添加到环境中。**



