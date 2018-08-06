# SpringBoot 2.0.X官方文档-009-开发工具

Spring Boot包含一组开发工具，可以使应用程序开发体验更加愉快。**spring-boot-devtools**模块可以包含在任何项目中，以提供额外的开发功能。要包含devtools支持，请将模块依赖项添加到您的构建配置中，如以下Maven和Gradle列表所示：

Maven

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```

Gradle

```bash
dependencies {
	compile("org.springframework.boot:spring-boot-devtools")
}
```

**在运行完全打包的应用程序时，会自动禁用开发工具。如果您的应用程序是从java -jar启动的，或者是从一个特殊的类加载程序启动的，那么它被认为是一个“(production application)生产应用程序”。将依赖项标记为Maven中的可选项或在Gradle中只使用compileOnly是防止devtools临时应用到使用项目的其他模块的最佳实践。**

**重新打包默认情况下不包含devtools。如果您想使用某个远程devtools特性，您需要禁用excludeDevtools build属性。Maven和Gradle插件都支持这个属性。**

> 属性默认值

Spring Boot支持几个库使用缓存来提高性能。例如，[模板引擎](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-spring-mvc-template-engines)缓存已编译的模板以避免重复解析模板文件。此外，Spring MVC可以在提供静态资源时为响应添加HTTP缓存头。

虽然缓存在生产中非常有用，但在开发过程中可能会适得其反，从而使您无法看到刚刚在应用程序中进行的更改。因此，spring-boot-devtools默认禁用缓存选项。

缓存选项通常由**application.properties**文件中配置。例如，Thymeleaf提供该**spring.thymeleaf.cache**配置。**spring-boot-devtools**模块不需要手动设置这些属性，而是自动应用合理的开发时配置

**有关devtools应用的属性的完整列表，请参阅 DevToolsPropertyDefaultsPostProcessor。**

> 自动重启

使用**spring-boot-devtools**的应用程序在类路径更改时自动重新启动。在IDE中工作时，这可能是一个有用的特性，因为它为代码更改提供了一个非常快速的反馈循环。默认情况下，指向文件夹的类路径中的任何条目都将受到监视，以查看是否有更改。注意，某些资源(如静态资产和视图模板)不需要重新启动应用程序。

**由于DevTools监视类路径资源，因此触发重新启动的唯一方法是更新类路径。导致更新类路径的方式取决于您使用的IDE。在Eclipse中，保存修改后的文件会导致更新类路径并触发重新启动。在IntelliJ IDEA中，构建项目（Build -> Build Project）具有相同的效果。**

**只要启用了forking，您就可以使用支持的构建插件(Maven和Gradle)来启动应用程序，因为DevTools需要一个独立的应用程序类加载器才能正常运行。默认情况下，Gradle和Maven会在类路径上检测DevTools。**

**与LiveReload一起使用时，自动重启非常有效。 有关详细信息，请参阅LiveReload部分。如果使用JRebel，则禁用自动重新启动以支持动态类重新加载。其他devtools功能（例如LiveReload和属性覆盖）仍然可以使用。**

**DevTools依赖于应用程序上下文的关机挂钩在重新启动时关闭它。如果您已经禁用了关闭钩子(SpringApplication.setRegisterShutdownHook(false))，那么它将无法正常工作。**

**当决定是否在类路径中的条目应该触发重新启动时，DevTools自动忽略命名的项目spring-boot， spring-boot-devtools，spring-boot-autoconfigure，spring-boot-actuator，和spring-boot-starter。**

**DevTools需要自定义ResourceLoader使用的ApplicationContext。如果您的应用程序已经提供了一个，它将被打包。不支持直接覆盖getResource方法ApplicationContext。**

Spring Boot提供的重启技术使用两个类加载器。不更改的类（例如，来自第三方jar的类）将加载到基 类加载器中。您正在积极开发的类将加载到重新启动的 类加载器中。重新启动应用程序时，将重新启动的类加载器创建为一个新的类加载器。这种方法意味着应用程序重新启动通常比“冷启动”快得多，因为基本类加载器已经可用并已填充。

如果您发现重新启动对于您的应用程序来说不够快或遇到类加载问题，您可以考虑ZeroTurnaround重新加载JRebel等技术 。这些工作通过在加载类时重写类使它们更适合重新加载。

> 日志记录评估中的变化

默认情况下，每次应用程序重新启动时，都会记录一个显示条件评估增量的报告。该报告显示了在进行更改（例如添加或删除Bean以及设置配置属性）时对应用程序的自动配置所做的更改。

要禁用报告的日志记录，请设置以下属性：

```bash
spring.devtools.restart.log-condition-evaluation-delta=false
```

> 排除资源

当某些资源发生更改时，不一定需要触发重新启动。例如，可以就地编辑Thymeleaf模板。默认情况下，在/META-INF/maven、/META-INF/resources、/resources、/static、/public或/template中更改资源不会触发重新启动，但会触发实时重新加载。如果您想自定义这些排除，可以使用**spring.devtools.restart.exclude**属性。例如，为了只排除/static和/public，您将设置以下属性:

```bash
spring.devtools.restart.exclude=static/**,public/**
```

**如果要保留这些默认值并添加其他排除项，请改用 spring.devtools.restart.additional-exclude属性。**

> 监听附加路径

当您对不在类路径中的文件进行更改时，您可能希望重新启动或重新加载应用程序。为此，请使用**spring.devtools.restart.additional-paths**属性配置监视其他路径。您可以使用前面描述的**spring.devtools.restart.exclude**属性来控制其他路径下的更改是触发完全重新启动还是实时重新加载。

> 禁用重启

如果您不想使用重新启动功能，可以使用**spring.devtools.restart.enabled**属性将其禁用。在大多数情况下，您可以在您的**application.properties**文件中设置此属性（这样做仍会初始化重新启动的类加载器，但它不会监视文件更改）。

如果您需要完全禁用重新启动支持(例如，因为它不能使用特定的库)，您需要在调用**SpringApplication.run(…)**之前将**spring.devtools.restart.enabled** System属性设置为false，如下面的示例所示:

```java
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```

> 使用触发器文件

如果您使用的IDE不断地编译已更改的文件，您可能更希望只在特定的时间触发重新启动。为此，您可以使用“触发器文件”，这是一个特殊的文件，当您希望实际触发重新启动检查时，必须对其进行修改。修改文件只会触发检查，只有在Devtools检测到它必须做一些事情时才会重新启动。触发器文件可以手动更新或使用IDE插件更新。

要使用触发器文件，请将**spring.devtools.restart.trigger-file**属性设置为触发器文件的路径。

**您可能希望将spring.devtools.restart.trigger-file设置为全局设置，以便所有项目都以相同的方式运行。**

> 自定义重启类加载器

如前面在Restart vs Reload部分中所述，使用两个类加载器实现了重启功能。对于大多数应用程序，此方法运行良好。但是，它有时会导致类加载问题。

默认情况下，IDE中的任何打开项目都使用“restart”类加载器加载，并且任何常规**.jar**文件都使用“base”类加载器加载。如果您处理多模块项目，并且并非每个模块都导入到IDE中，则可能需要自定义内容。为此，您可以创建一个**META-INF/spring-devtools.properties**文件。

spring-devtools.properties文件可以包含前缀为restart.exclude的属性。include元素是应该被拉高到“restart”的类加载器的项目，以及exclude要素是应该向下推入“base”类加载器的项目。该属性的值是应用于类路径的正则表达式模式，如以下示例所示：

```bash
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```

**只要属性以restart.include或restart.exclude开始。所有属性键必须是唯一的。**

**从类路径中加载所有META-INF/spring-devtools.properties内容。您可以将文件打包到项目中，也可以打包在项目使用的库中。**

> 已知的限制

重新启动功能不适用于使用标准反序列化的对象**ObjectInputStream**。如果你需要反序列化的数据，你可能需要使用Spring的**ConfigurableObjectInputStream**结合 **Thread.currentThread().getContextClassLoader()**。

不幸的是，几个第三方库反序列化而没有考虑上下文类加载器。如果您发现此类问题，则需要向原始作者请求修复。

> LiveReload

**spring-boot-devtools**模块包括一个嵌入式LiveReload服务器，可用于在更改资源时触发浏览器刷新。LiveReload浏览器扩展程序可从[livereload.com](https://livereload.com/extensions/)免费用于Chrome，Firefox和Safari 。

如果您不想在应用程序运行时启动LiveReload服务器，则可以将**spring.devtools.livereload.enabled**属性设置为**false**。

**您一次只能运行一个LiveReload服务器。在启动应用程序之前，请确保没有其他LiveReload服务器正在运行。如果从IDE启动多个应用程序，则只有第一个具有LiveReload支持。**

> 全局设置

您可以通过添加名为**.spring-boot-devtools**的文件来配置全局devtools设置。你的$HOME文件夹(注意文件名以“.”开头)。任何添加到这个文件的属性都适用于使用devtools的计算机上的所有Spring引导应用程序。例如，要配置重新启动以始终使用触发器文件，您需要添加以下属性:

~/.spring-boot-devtools.properties

```bash
spring.devtools.reload.trigger-file=.reloadtrigger
```

> 远程应用程序

Spring Boot开发人员工具不仅限于本地开发。远程运行应用程序时，您还可以使用多个功能。远程支持是选择加入。要启用它，您需要确保devtools包含在重新打包的存档中，如下面的清单所示：

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<excludeDevtools>false</excludeDevtools>
			</configuration>
		</plugin>
	</plugins>
</build>
```

然后，您需要设置spring.devtools.remote.secret属性，如以下示例所示：

```bash
spring.devtools.remote.secret=mysecret
```

**spring-boot-devtools在远程应用程序上启用存在安全风险。您永远不应该在生产部署上启用支持。**

远程devtools支持由两部分组成：一个接受连接的服务器端端点和一个在IDE中运行的客户端应用程序。**spring.devtools.remote.secret**设置属性后，将自动启用服务器组件。必须手动启动客户端组件。

> 运行远程客户端应用程序

远程客户端应用程序设计为从您的IDE中运行。您需要运行**org.springframework.boot.devtools.RemoteSpringApplication**，其类路径与连接到的远程项目相同。应用程序的唯一必需参数是它连接到的远程URL。

例如，如果您使用的是Eclipse或STS，并且您有一个名为my-app已部署到Cloud Foundry的项目，那么您将执行以下操作：

- 选择Run Configurations…​从Run菜单。
- 创建一个新的Java Application“启动配置”。
- 浏览my-app项目。
- 用org.springframework.boot.devtools.RemoteSpringApplication作主类。
- 添加https://myapp.cfapps.io到Program arguments（或任何远程URL）。

正在运行的远程客户端可能类似于以下内容：

```
  .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote :: 2.0.4.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```

**因为远程客户端使用与实际应用程序相同的类路径，所以可以直接读取应用程序属性。这就是spring.devtools.remote.secret属性的读取方式，并将其传递给服务器进行身份验证。**

**使用https://作为连接协议总是明智的，这样就可以加密传输并不能截获密码。**

**如果需要使用代理访问远程应用程序，请配置spring.devtools.remote.proxy.host和spring.devtools.remote.proxy.port属性。**

> 远程更新

远程客户端以与本地重启相同的方式监视应用程序类路径的更改。将任何更新的资源推到远程应用程序，并(如果需要)触发重新启动。如果您对使用本地没有的云服务的特性进行迭代，这将非常有用。通常，远程更新和重新启动要比完整的重新构建和部署周期快得多。

**只有在远程客户端运行时才对文件进行监视。如果在启动远程客户端之前更改文件，则不会将其推到远程服务器。**

