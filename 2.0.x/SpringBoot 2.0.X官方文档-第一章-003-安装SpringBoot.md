# SpringBoot 2.0.X官方文档-第一章-003-安装SpringBoot

Spring Boot可以与“经典(classic)”Java开发工具一起使用，也可以作为命令行工具安装。无论哪种方式，您都需要[Java SDK v1.8](https://www.java.com/)或更高版本。在开始之前，您应该使用以下命令检查当前的Java安装：


```bash
$ java -version
```

如果您对Java开发不熟悉，或者想要试验Spring Boot，则可能需要先尝试[Spring Boot CLI](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/getting-started-installing-spring-boot.html#getting-started-installing-the-cli)（命令行界面）。否则，请阅读“经典(classic)”安装说明。

> 针对Java开发程序员安装说明

Spring Boot的使用方式与标准Java库的使用相同，只需在类路径中包含适当的**spring-boot-*.jar**文件。Spring Boot不需要任何特殊的集成工具，所以可以使用任何IDE或文本编辑器进行开发；并且Spring Boot应用程序没有什么特殊的地方，因此您可以像其他Java程序一样运行和调试。虽然您可以直接复制Spring Boot的jar包，但我们通常建议您使用依赖关系管理的构建工具（如Maven或Gradle）。

> Maven安装

Spring Boot兼容Apache Maven 3.2或更高版本。如果您尚未安装Maven，则可以按照[maven.apache.org](https://maven.apache.org/)的说明进行操作。

```
在许多操作系统上，Maven可以通过软件包管理器进行安装。
如果您使用OSX Homebrew，请尝试brew install maven
Ubuntu用户可以运行 sudo apt-get install maven
Windows用户可以在拥有管理员权限下运行提示符窗口输入choco install maven
```

Spring Boot依赖关系使用**org.springframework.boot**和**groupId**。通常，您的Maven POM文件从**spring-boot-starter-parent**项目中继承并向一个或多个“[Starter](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/using-boot-build-systems.html#using-boot-starter)”声明依赖关系。Spring Boot还提供了一个可选的[Maven插件](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/build-tool-plugins-maven-plugin.html)来创建可执行的jar。

一个典型的pom.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<!-- Inherit defaults from Spring Boot -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.3.RELEASE</version>
	</parent>

	<!-- Add typical dependencies for a web application -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<!-- Package as an executable jar -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

**spring-boot-starter-parent**是使用Spring Boot的一个很好的方式，但它并不是所有的时候都适合。有时您可能需要从不同的父POM继承，或者您可能不喜欢我们的默认设置。 请参见第13.2.2节“使用不带父POM的Spring Boot”作为使用导入作用域(import scope)的替代解决方案。

> Gradle安装

Spring Boot与Gradle 4兼容。如果您还没有安装Gradle，可以按照[gradle.org](https://gradle.org/)上的说明进行操作。
可以使用org.springframework.boot 组(group)声明Spring Boot 的依赖项。 通常，您的项目将声明一个或多个“启动器(Starters)”的依赖。Spring Boot提供了一个有用的Gradle插件，可用于简化依赖关系声明和创建可执行 jar包。

- [x] 当您需要构建项目时，Gradle Wrapper提供了一种“获取”Gradle的好方法。这是一个小脚本和库，与代码一起提交以引导构建过程。有关详细信息，请参阅docs.gradle.org/4.2.1/userguide/gradle_wrapper.html。

以下示例显示了一个典型的build.gradle文件：

```
plugins {
	id 'org.springframework.boot' version '2.0.3.RELEASE'
	id 'java'
}


jar {
	baseName = 'myproject'
	version =  '0.0.1-SNAPSHOT'
}

repositories {
	jcenter()
}

dependencies {
	compile("org.springframework.boot:spring-boot-starter-web")
	testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

> 安装Spring Boot CLI

Spring Boot CLI(Command Line Interface) 是一个命令行工具，您可以使用它来快速使用Spring进行原型开发。它可以让你运行[Groovy](http://groovy-lang.org/)脚本，这意味着你有一个熟悉的类Java语法，没有太多的样板代码(boilerplate code)。
您不需要使用CLI来使用Spring Boot，但它绝对是让Spring应用程序实现最快的最快捷方式。

> 手动安装

您可以从Spring软件存储库下载Spring CLI分发版：

1. [spring-boot-cli-2.0.3.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.0.3.RELEASE/spring-boot-cli-2.0.3.RELEASE-bin.zip)
2. [spring-boot-cli-2.0.3.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.0.3.RELEASE/spring-boot-cli-2.0.3.RELEASE-bin.tar.gz)
3. 最新的[快照](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)也可用

下载完成后，请按照解压缩归档中的**INSTALL.txt**说明进行操作。总之，在.zip文件的bin/目录中有一个spring脚本（Windows的spring.bat），或者你可以使用java -jar（脚本可以帮助您确保类路径设置正确）。

> 使用SDKMAN安装！

SDKMAN！（软件开发工具包管理器）可用于管理各种二进制SDK的多个版本，包括Groovy和Spring Boot CLI。获取SDKMAN！从[sdkman.io](http://sdkman.io/)安装并使用以下命令安装Spring Boot：

```bash
$ sdk install springboot 
$ spring --version 
Spring Boot v2.0.3.RELEASE
```

如果您正在开发CLI的功能，并希望轻松访问刚创建的版本，请遵循以下额外说明。

```bash
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-2.0.3.RELEASE-bin/spring-2.0.3.RELEASE/ 
$ sdk default springboot dev 
$ spring --version 
Spring CLI v2.0.3.RELEASE
```

这将安装一个称为dev的spring的本地实例(instance)。 它指向您构建位置的target，所以每次重建(rebuild)Spring Boot时，Spring 将是最新的。您可以通过运行以下命令来查看它：

```bash
$ sdk ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 2.0.3.RELEASE

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```

> OSX Homebrew安装

如果您在Mac上使用[Homebrew](https://brew.sh/)，则可以使用以下命令安装Spring Boot CLI：

```bash
$ brew tap pivotal/tap
$ brew install springboot
```

Homebrew会将Spring安装到**/usr/local/bin**

- [x] 如果您没有看到公钥(formula)，您的安装可能会过期。 只需执行brew更新，然后重试。

> MacPorts安装

如果您在Mac上使用[MacPorts](https://www.macports.org/)，则可以使用以下命令安装Spring Boot CLI：

```bash
$ sudo port install spring-boot-cli
```

> 命令行自动完成

Spring Boot CLI为BASH和zsh shell提供命令提示的功能。
您可以在任何shell中引用脚本（也称为spring），或将其放在您的个人或系统范围的bash完成初始化中。 在Debian系统上，系统范围的脚本位于/shell-completion/bash中，当新的shell启动时，该目录中的所有脚本将被执行。

```bash
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
  grab  help  jar  run  test  version
```

如果您使用Homebrew或MacPorts安装Spring Boot CLI，则命令行完成脚本会自动在您的shell中注册

> Windows Scoop安装

如果您在Windows上使用[Scoop](http://scoop.sh/)，则可以使用以下命令安装Spring Boot CLI：

```bat
> scoop bucket add extras
> scoop install springboot
```

Scoop将spring安装在**~/scoop/apps/springboot/current/bin**目录.
如果您没有看到应用程序清单，那么您的安装密钥可能过时。在这种情况下，请运行**scoop update**并重试。

> 快速启动Spring Cli示例

您可以使用以下Web应用程序来测试您的安装。首先，创建一个名为的文件app.groovy，如下所示:

```java
@RestController
class ThisWillActuallyRun {

	@RequestMapping("/")
	String home() {
		"Hello World!"
	}

}
```

然后从shell运行它：

```bash
$ spring run app.groovy
```

由于需要下载依赖项，应用程序的第一次运行速度很慢。后续运行速度更快。
使用浏览器打开[localhost:8080](http://localhost:8080).您应该会看到以下输出：

```html
Hello World!
```

> 从早期版本的Spring Boot升级

如果您是从早期版本的Spring Boot进行升级，请查看[“migration guide” on the project wiki](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)其中提供了详细的升级说明。检查 “[release notes](https://github.com/spring-projects/spring-boot/wiki)”，以获取每个发行版的“新功能”和“值得注意”功能列表。
要升级现有的CLI，请使用相应的软件包管理器命令(例如**brew upgrade**),如果您手动安装了CLI，请按照[standard instructions](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/getting-started-installing-spring-boot.html#getting-started-manual-cli-installation)进行操作，并记住更新PATH环境变量以删除所有旧的引用。