# SpringBoot 2.0.X官方文档-第一章-005-构建系统

强烈建议您选择支持依赖关系管理且可以使用发布到“Maven Central”存储库的构建系统 。我们建议您选择Maven或Gradle。可以让Spring Boot与其他构建系统（例如Ant）一起工作，但是它们不会得到很好的支持。

> 依赖管理

每个版本的Spring Boot都提供了它支持的依赖项的关系列表。实际上，您不需要为构建配置中的任何这些依赖项提供版本，因为Spring Boot会为您管理这些依赖项。当您升级Spring Boot时，这些依赖项也会以一致的方式升级。

**如果需要，您仍然可以指定版本并覆盖Spring Boot的建议版本。**

关系列表包含可以与Spring Boot一起使用的所有spring模块以及精确的第三方库列表。该列表可作为标准的物料(Materials)清单（spring-boot-dependencies）使用 ，可与Maven和 Gradle一起使用。

**每个版本的Spring Boot都与Spring Framework的基本版本相关联。我们强烈建议您不要指定其版本。**

> Maven

Maven用户可以从**spring-boot-starter-parent**项目继承以获得合理的默认值。父项目提供以下功能：

 - Java 1.8作为默认编译器级别。
 - UTF-8源编码。
 - 一个依赖管理部分，从Spring启动依赖性继承POM，管理公共依赖的版本。此依赖关系管理允许您在自己的pom中使用时省略这些依赖项的<version>标记。
 - 更合理的[资源过滤](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)。
 - 更合理的插件配置（[exec plugin](http://www.mojohaus.org/exec-maven-plugin/)，[Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin)，[shade](https://maven.apache.org/plugins/maven-shade-plugin/)）。
 - 针对**application.properties**和**application.yml**的更合理的资源过滤，包括特定的文件（例如，**application-dev.properties**和 **application-dev.yml**）
 
**请注意，由于application.properties和application.yml文件接受Spring样式占位符（${…​}），因此Maven过滤更改为使用@..@占位符。（您可以通过设置调用的Maven属性来覆盖它resource.delimiter。）**

> 继承Starter Parent

要将项目配置为继承**spring-boot-starter-parent**，请将parent按如下所示进行设置 ：

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.4.RELEASE</version>
</parent>
```

**您应该只需要在此依赖项上指定Spring Boot版本号。如果导入其他启动器，则可以安全地省略版本号。**

通过该设置，您还可以通过覆盖自己的项目中的属性来覆盖单个依赖。 例如，要升级到另一个 Spring Data 版本序列，您需要将以下内容添加到您的**pom.xml**中。

```xml
<properties>
	<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

**检查[spring-boot-dependencies pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)以获取支持的属性列表。**

> 在没有父POM的情况下使用Spring Boot

不是每个人都喜欢从**spring-boot-starter-parent** POM继承。您可能拥有自己需要使用的公司标准父级，或者您可能更愿意明确声明所有Maven配置。

如果您不想使用**spring-boot-starter-paren**t，您仍然可以通过使用**scope=import**依赖关系来保持依赖关系管理（但不是插件管理），如下所示：

```xml
<dependencyManagement>
		<dependencies>
		<dependency>
			<!-- Import dependency management from Spring Boot -->
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.0.4.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

如上所述，前面的示例设置不允许您使用属性覆盖单个依赖项。要获得相同的结果，您需要在输入之前在dependencyManagement项目中添加一个条目。例如，要升级到另一个Spring Data版本系列，您可以将以下元素添加到：spring-boot-dependencies pom.xml

```xml
<dependencyManagement>
	<dependencies>
		<!-- Override Spring Data release train provided by Spring Boot -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-releasetrain</artifactId>
			<version>Fowler-SR2</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.0.4.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

**在前面的示例中，我们指定了BOM，但是可以以相同的方式覆盖任何依赖关系类型。**

> 使用Spring Boot Maven插件

Spring Boot包含一个[Maven插件](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/build-tool-plugins-maven-plugin.html)，可以将项目打包为可执行jar。如果要使用它，请将插件添加到您的<plugins>部分，如以下示例所示：

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

**如果您使用Spring Boot父pom，则只需添加插件。除非您要更改父级中定义的设置，否则无需对其进行配置。**

> Gradle

要了解如何将Spring Boot与Gradle一起使用，请参阅Spring Boot的Gradle插件的文档：

- [HTML](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/gradle-plugin/reference/html)和[PDF](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf)
- [API](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/gradle-plugin/api)

> Ant

可以使用Apache Ant + Ivy构建Spring Boot项目。该**spring-boot-antlib**的antlib模块还可以帮助ant创建可执行的JAR文件。

要声明依赖项，典型**ivy.xml**文件类似于以下示例：

```xml
<ivy-module version="2.0">
	<info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
	<configurations>
		<conf name="compile" description="everything needed to compile this module" />
		<conf name="runtime" extends="compile" description="everything needed to run this module" />
	</configurations>
	<dependencies>
		<dependency org="org.springframework.boot" name="spring-boot-starter"
			rev="${spring-boot.version}" conf="compile" />
	</dependencies>
</ivy-module>
```

典型**build.xml**示例：

```xml
<project
	xmlns:ivy="antlib:org.apache.ivy.ant"
	xmlns:spring-boot="antlib:org.springframework.boot.ant"
	name="myapp" default="build">

	<property name="spring-boot.version" value="2.0.4.RELEASE" />

	<target name="resolve" description="--> retrieve dependencies with ivy">
		<ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
	</target>

	<target name="classpaths" depends="resolve">
		<path id="compile.classpath">
			<fileset dir="lib/compile" includes="*.jar" />
		</path>
	</target>

	<target name="init" depends="classpaths">
		<mkdir dir="build/classes" />
	</target>

	<target name="compile" depends="init" description="compile">
		<javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
	</target>

	<target name="build" depends="compile">
		<spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
			<spring-boot:lib>
				<fileset dir="lib/runtime" />
			</spring-boot:lib>
		</spring-boot:exejar>
	</target>
</project>
```

**如果您不想使用该spring-boot-antlib模块，请参见 [第87.9节“从Ant构建可执行文件而不使用spring-boot-antlib”](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/howto-build.html#howto-build-an-executable-archive-with-ant)操作方法”。**

> 启动器

启动器是一组方便的依赖关系描述符，可以包含在应用程序中。 您可以获得所需的所有Spring和相关技术的一站式服务，无需通过示例代码搜索和复制粘贴依赖配置。 例如，如果要开始使用Spring和JPA进行数据库访问，那么只需在项目中包含spring-boot-starter-data-jpa依赖关系即可。

启动器包含许多依赖关系，包括您需要使项目快速启动并运行，并具有一致的受支持的依赖传递关系。

What’s in a name

所有正式起动器都遵循类似的命名模式： spring-boot-starter- ，其中 是特定类型的应用程序。 这个命名结构旨在帮助你快速找到一个启动器。 许多IDE中的Maven插件允许您按名称搜索依赖项。 例如，安装Eclipse或STS的Maven插件后，您可以简单地在POM编辑器中点击 Dependency Hierarchy，并在filter输入“spring-boot-starter”来获取完整的列表。
如[创建自己的启动器](http://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#boot-features-custom-starter)部分所述，第三方启动程序不应该从Spring-boot开始，因为它是为正式的Spring Boot artifacts 保留的。 acme的第三方启动器通常被命名为acme-spring-boot-starter。

Spring Boot在org.springframework.boot组下提供了以下应用程序启动器：

Spring Boot应用程序启动器

|名称|描述|Pom|
|---|---|---|
|spring-boot-starter|核心启动器，包括自动配置支持，日志记录和YAML|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml)|
|spring-boot-starter-activemq|使用Apache ActiveMQ的JMS启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml)|
|spring-boot-starter-amqp|使用Spring AMQP和Rabbit MQ的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml)|
|spring-boot-starter-aop|使用Spring AOP和AspectJ进行面向切面编程的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml)|
|spring-boot-starter-artemis|使用Apache Artemis的JMS启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml)|
|spring-boot-starter-batch|使用Spring Batch的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml)|
|spring-boot-starter-cache|使用Spring Framework缓存支持的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml)|
|spring-boot-starter-cloud-connectors|使用Spring Cloud连接器，简化了与Cloud Foundry和Heroku等云平台中的服务连接的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml)|
|spring-boot-starter-data-cassandra|使用Cassandra分布式数据库和Spring Data Cassandra的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml)|
|spring-boot-starter-data-cassandra-reactive|使用Cassandra分布式数据库和Spring Data Cassandra Reactive的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml)|
|spring-boot-starter-data-couchbase|使用Couchbase面向文档的数据库和Spring Data Couchbase的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml)|
|spring-boot-starter-data-couchbase-reactive|使用Couchbase面向文档的数据库和Spring Data Couchbase Reactive的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml)|
|spring-boot-starter-data-elasticsearch|使用Elasticsearch搜索和分析引擎以及Spring Data Elasticsearch的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml)|
|spring-boot-starter-data-jpa|将Spring Data JPA与Hibernate一起使用的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml)|
|spring-boot-starter-data-ldap|使用Spring Data LDAP的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml)|
|spring-boot-starter-data-mongodb|使用MongoDB面向文档的数据库和Spring Data MongoDB的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml)|
|spring-boot-starter-data-mongodb-reactive|使用MongoDB面向文档的数据库和Spring Data MongoDB Reactive的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml)|
|spring-boot-starter-data-neo4j|使用Neo4j图形数据库和Spring Data Neo4j的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml)|
|spring-boot-starter-data-redis|与Spring Data Redis和Lettuce客户端一起使用Redis键值数据存储的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml)|
|spring-boot-starter-data-redis-reactive|使用Redis键值数据存储与Spring Data Redis被动和Lettuce客户端的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml)|
|spring-boot-starter-data-rest|使用Spring Data REST通过REST公开Spring Data存储库的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml)|
|spring-boot-starter-data-solr|使用Apache Solr搜索平台和Spring Data Solr的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml)|
|spring-boot-starter-freemarker|使用FreeMarker视图构建MVC Web应用程序的|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml)|
|spring-boot-starter-groovy-templates|使用Groovy模板视图构建MVC Web应用程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml)|
|spring-boot-starter-hateoas|使用Spring MVC和Spring HATEOAS构建基于超媒体的RESTful Web应用程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml)|
|spring-boot-starter-integration|使用Spring Integration的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml)|
|spring-boot-starter-jdbc|将JDBC与HikariCP连接池一起使用的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml)|
|spring-boot-starter-jersey|使用JAX-RS和Jersey构建RESTful Web应用程序的启动器。替代**spring-boot-starter-web**|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml)|
|spring-boot-starter-jooq|使用jOOQ访问SQL数据库的启动器。替代**spring-boot-starter-data-jpa**或**spring-boot-starter-jdbc**|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml)
|spring-boot-starter-json|阅读和写作json的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml)|
|spring-boot-starter-jta-atomikos|使用Atomikos进行JTA交易的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml)|
|spring-boot-starter-jta-bitronix|使用Bitronix进行JTA事务的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml)|
|spring-boot-starter-jta-narayana|使用Narayana进行JTA交易的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-narayana/pom.xml)|
|spring-boot-starter-mail|使用Java Mail和Spring Framework的电子邮件发送支持的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml)|
|spring-boot-starter-mustache|使用Mustache视图构建Web应用程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml)|
|spring-boot-starter-quartz|使用Quartz调度程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml)|
|spring-boot-starter-security|使用Spring Security的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml)|
|spring-boot-starter-test|使用JUnit，Hamcrest和Mockito等库来测试Spring Boot应用程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml)|
|spring-boot-starter-thymeleaf|使用Thymeleaf视图构建MVC Web应用程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml)|
|spring-boot-starter-validation|使用Java Bean Validation和Hibernate Validator的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml)|
|spring-boot-starter-web|使用Spring MVC构建Web（包括RESTful）应用程序的启动器。使用Tomcat作为默认嵌入式容器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml)|
|spring-boot-starter-web-services|使用Spring Web Services的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml)
|spring-boot-starter-webflux|使用Spring Framework的Reactive Web支持构建WebFlux应用程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml)
|spring-boot-starter-websocket|使用Spring Framework的WebSocket支持构建WebSocket应用程序的启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml)

除应用程序启动器外，还可以使用以下启动器添加[生产环境](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/production-ready.html)功能：

Spring Boot生产启动器

|名称|描述|Pom|
|---|---|---|
|spring-boot-starter-actuator|使用Spring Boot的Actuator的启动器，它提供生产就绪功能，帮助您监控和管理您的应用程序|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml)

最后，Spring Boot还包括一些启动器，如果要排除或替换特定的技术，可以使用它们：

|名称|描述|Pom|
|---|---|---|
|spring-boot-starter-jetty|使用Jetty作为嵌入式servlet容器的启动器。替代**spring-boot-starter-tomcat**|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml)|
|spring-boot-starter-log4j2|使用Log4j2进行日志记录的启动器。替代**spring-boot-starter-logging**|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml)|
|spring-boot-starter-logging|使用Logback进行日志记录的启动器。默认日志启动器|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml)|
|spring-boot-starter-reactor-netty|使用Reactor Netty作为嵌入式响应式HTTP服务器的启动器。|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml)|
|spring-boot-starter-tomcat|使用Tomcat作为嵌入式servlet容器的启动器。使用的默认servlet容器启动器**spring-boot-starter-web**|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml)|
|spring-boot-starter-undertow|使用Undertow作为嵌入式servlet容器的启动器。替代spring-boot-starter-tomcat|[Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml)|

有关其他社区提供的启动程序的列表，请参阅GitHub上的**spring-boot-starters**模块中的[README](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)文件。

