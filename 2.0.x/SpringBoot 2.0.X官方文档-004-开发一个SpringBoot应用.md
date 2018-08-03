# SpringBoot 2.0.X官方文档-第一章-004-开发一个SpringBoot应用

本节介绍如何开发一个简单的“Hello World！”Web应用程序，该应用程序重点介绍Spring Boot的一些主要功能。我们使用Maven来构建这个项目，因为大多数IDE都支持它。

* [x] [Spring IO](https://spring.io/)该网站包含许多Spring Boot的入门指南。如果您正在寻求解决一些具体问题; 可以先看一下那里。
* [x] 您可以通过到[start.spring.io](https://start.spring.io/)网站中并从依赖关系搜索器中选择“Web”起始器来缩短以下步骤。
* [x] 这样做会产生一个新的项目结构,以便您可以立即开始编码。点击[Spring Initializr文档](https://github.com/spring-io/initializr)以获取更多详细信息。

在开始之前，请打开终端并运行以下命令以确保您已安装了Java和Maven的有效版本：

```bash
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

```bash
$ mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T16:41:47+00:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```

* [x] 这个示例需要在其自己的文件夹中创建。 后面我们假设您在当前目录已经创建了一个正确的文件夹。

> 创建POM

我们需要先创建一个Maven pom.xml文件。本pom.xml是用来构建项目的配置文件。打开您最喜欢的文本编辑器并添加以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.3.RELEASE</version>
	</parent>

	<!-- Additional lines to be added here... -->

</project>
```

这应该给你一个工作构建(working build)，您可以通过运行`mvn package`来测试它（现在，您可以忽略`jar will be empty - no content was marked for inclusion!`警告）。

* [x] 此时，您可以将项目导入IDE（大多数现代Java IDE包含对Maven的内置支持）。为了简单起见，我们在本例中继续使用纯文本编辑器。

> 添加类路径依赖关系

Spring Boot提供了一些“启动器(Starters)”，可以方便地将jar添加到类路径中。我们的示例应用程序已经在POM的父部分使用了spring-boot-starter-parent。spring-boot-starter-parent是一个特殊启动器，提供一些Maven的默认值。它还提供依赖管理`dependency-management`标签，以便您可以省略子模块依赖关系的版本标签。
其他“启动器(Starters)”只是提供您在开发特定类型的应用程序时可能需要的依赖关系。 由于我们正在开发Web应用程序，所以我们将添加一个spring-boot-starter-web依赖关系，但在此之前，我们来看看我们目前的依赖。

```bash
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

mvn dependency:tree：打印项目依赖关系的树形表示。 您可以看到spring-boot-starter-parent本身不在依赖关系中。 编辑pom.xml并在parent下添加spring-boot-starter-web依赖关系：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```

如果你再次运行mvn dependency:tree，你会发现现在有很多附加的依赖关系，包括Tomcat Web服务器和Spring Boot本身。

> 编写代码

要完成我们的应用程序，我们需要创建一个的Java文件。 默认情况下，Maven将从`src/main/java`编译源代码，因此您需要创建该文件夹结构，然后添加一个名为`src/main/java/Example.java`的文件：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

	@RequestMapping("/")
	String home() {
		return "Hello World!";
	}

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Example.class, args);
	}

}
```

虽然这里没有太多的代码，但是有一些重要的部分。

> @RestController和@RequestMapping注解

我们的Example类的第一个注解是@RestController。 这被称为 stereotype annotation。它为人们阅读代码提供了一些提示，对于Spring来说，这个类具有特定的作用。在这里，我们的类是一个web @Controller，所以Spring在处理传入的Web请求时会考虑这个类。

@RequestMapping注解提供“路由”信息。 告诉Spring，任何具有路径“/”的HTTP请求都应映射到home方法。

@RestController注解告诉Spring将生成的字符串直接返回给调用者。

* [x] @RestController和@RequestMapping注解是Spring MVC的注解（它们不是Spring Boot特有的）。 有关更多详细信息，请参阅[Spring参考文档中的MVC部分](http://docs.spring.io/spring/docs/4.3.7.RELEASE/spring-framework-reference/htmlsingle#mvc)。

> @EnableAutoConfiguration注解

第二个类级别的注释是@EnableAutoConfiguration。 这个注解告诉 Spring Boot 根据您添加的jar依赖关系来“猜(guess)”你将如何配置Spring。由于spring-boot-starter-web添加了Tomcat和Spring MVC，自动配置将假定您正在开发Web应用程序并相应地配置Spring。

* [x] 启动器和自动配置 : 自动配置旨在与“起动器”配合使用，但两个概念并不直接相关。 您可以自由选择启动器之外的jar依赖项，Spring Boot仍然会自动配置您的应用程序。

> “mian”方法

我们的应用程序的最后一部分是main()方法。 这只是一个遵循Java惯例的应用程序入口点的标准方法。 我们的main()方法通过调用run()委托(delegates)给Spring Boot的SpringApplication类。 SpringApplication将引导我们的应用程序，启动Spring，然后启动自动配置的Tomcat Web服务器。 我们需要将Example.class作为一个参数传递给run方法来告诉SpringApplication，它是主要的Spring组件。 还传递了args数组以传递命令行参数。

> 运行示例

由于我们使用了spring-boot-starter-parent POM，所以我们有一个可用的运行目标，我们可以使用它来启动应用程序。 键入mvn spring-boot(从根目录运行以启动应用程序),您应该看到类似于以下内容的输出：

```bash
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.0.3.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

如果您打开一个Web浏览器在网址中输入[localhost:8080](localhost:8080)，您应该看到以下输出：

```bash
Hello World!
```

要正常退出应用程序，请按`ctrl-c`。

> 创建可执行jar

我们通过创建一个完全独立的可执行jar文件来完成我们的示例，该文件可以在生产环境中运行。可执行jar（有时称为“fat jars”）是包含您的编译类以及您的代码需要运行的所有jar依赖项的归档文件。

* [x] 可执行jar和Java : Java不提供任何标准的方法来加载嵌套的jar文件（即本身包含在jar中的jar文件）。 如果您正在寻找可以发布自包含的应用程序，这可能是有问题的。
为了解决这个问题，许多开发人员使用“uber” jars。 一个uber jar简单地将所有类、jar包进行档案。 这种方法的问题是，很难看到您在应用程序中实际使用哪些库。 如果在多个jar中使用相同的文件名（但具有不同的内容），也可能会出现问题。Spring Boot采用[一个不同的方法](http://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#executable-jar)这样可以直接对jar进行嵌套。

要创建可执行的jar，我们需要将spring-boot-maven-plugin添加到我们的pom.xml中。 在 dependencies标签下方插入以下行：

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

* [x] 所述spring-boot-starter-parent POM包括<executions>配置以结合repackage指令进行打包。如果您不使用父POM，则需要自行声明此配置。有关详细信息，请参阅[插件文档](https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/maven-plugin/usage.html)。

保存pom.xml并从命令行运行mvn package，如下所示：

```bash
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.0.3.RELEASE:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

如果你看看target目录，你应该看到myproject-0.0.1-SNAPSHOT.jar。 该文件的大小约为10 MB。 如果你想查看里面，可以使用jar tvf，如下所示：

```bash
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

您还应该在target目录中看到一个名为myproject-0.0.1-SNAPSHOT.jar.original的较小文件。 这是Maven在Spring Boot重新打包之前创建的原始jar文件。

使用java -jar命令运行该应用程序：

```bash
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.0.3.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```

像之前一样，`ctrl+c`正常退出应用程序。

