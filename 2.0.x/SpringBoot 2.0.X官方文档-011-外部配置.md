# SpringBoot 2.0.X官方文档-011-外部配置

Spring Boot允许您外部化配置，以便在不同的环境中使用相同的应用程序代码。您可以使用属性文件、YAML文件、环境变量和命令行参数来外部化配置。属性值可以通过使用@Value注解直接注入到bean中，可以通过Spring的环境抽象访问，也可以通过@ConfigurationProperties方法[绑定到结构化](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-external-config-typesafe-configuration-properties)对象。

Spring Boot使用非常特别的PropertySource命令，旨在允许合理地覆盖值。属性按以下顺序选择：

- 在你的主目录上的[Devtools全局属性](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#using-boot-devtools-globalsettings)。(**~/.spring-boot-devtools.properties**当devtools处于active时的属性)。
- 单元测试中的[@TestPropertySource](https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/org/springframework/test/context/TestPropertySource.html)注解。
- 单元测试中的[@SpringBootTest#properties](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/api/org/springframework/boot/test/context/SpringBootTest.html)注解属性
- 命令行参数。
- SPRING_APPLICATION_JSON中的属性值（内嵌JSON嵌入到环境变量或系统属性中）。
- ServletConfig初始化参数。
- ServletContext初始化参数。
- 来自java:comp/env的JNDI属性。
- Java系统属性（System.getProperties()）。
- OS环境变量。
- RandomValuePropertySource，只有随机的属性random.*中。
- jar包外面的[Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-external-config-profile-specific-properties)（application- {profile} .properties和YAML变体）
- jar包内的[Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-external-config-profile-specific-properties)（application-{profile}.properties和YAML变体）
- jar包外的应用属性文件（application.properties和YAML变体）。
- jar包内的应用属性文件（application.properties和YAML变体）。
- 在@Configuration上的**@PropertySource**注解。
- 默认属性（使用SpringApplication.setDefaultProperties设置）。

一个具体的例子，假设你开发一个使用name属性的@Component：

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

在应用程序类路径中(例如，在jar中)可以有一个应用程序。属性文件，为名称提供合理的默认属性值。在新环境中运行时，应用程序。属性文件可以在覆盖名称的jar之外提供。对于一次性测试，您可以使用特定的命令行开关启动(例如，java -jar app.jar—name="Spring")。

SPRING_APPLICATION_JSON属性可以在命令行中提供一个环境变量。 例如在UN*X shell中：

```bash
$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar
```

在前面的示例中，您最终会在Spring环境中得到acme.name=test。您还可以将JSON作为spring.application.json提供给系统属性中，如下例所示:

```bash
$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar
```

您还可以使用命令行参数提供JSON，如下例所示:

```bash
$ java -jar myapp.jar --spring.application.json='{"name":"test"}'
```

您还可以将JSON作为JNDI变量提供，如下所示:**java:comp/env/spring.application.json**。

> 配置随机值

RandomValuePropertySource对注入随机值(例如，到秘密或测试用例中)非常有用。它可以生成整数、longs、uuid或字符串，如下例所示:

```
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

random.int*语法是OPEN value (，max) CLOSE，其中OPEN、CLOSE是任何字符和值，max是整数。如果提供max，则value为最小值，max为最大值(互斥)。

> 访问命令行属性

默认情况下，SpringApplication将任何命令行选项参数(即以**server.port=9000**开头的参数)转换为属性，并将其添加到Spring环境中。如前所述，命令行属性总是优先于其他属性源。

如果不希望将命令行属性添加到环境中，可以使用**SpringApplication.setAddCommandLineProperties(false)**禁用它们。

> 应用程序属性文件

SpringApplication从应用程序加载属性。属性文件在以下位置，并将它们添加到Spring环境中:

- 当前目录的**/config**子目录
- 当前目录
- 类路径/配置方案
- 类路径的根路径

按优先级排序(在列表中较高位置定义的属性覆盖在较低位置定义的属性)。

**您还可以使用[YAML ('.yml')文件](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-external-config-yaml)作为'.properties'的替代。**

如果您不喜欢**application.properties**配置文件名，可以通过指定**spring.config.name**环境属性切换到另一个文件名。您还可以使用**spring.config.location** environment属性（以逗号分隔的目录位置或文件路径列表）来引用显式位置。以下示例显示如何指定其他文件名：

```bash
$ java -jar myproject.jar --spring.config.name=myproject
```

以下示例显示如何指定两个位置：

```bash
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```

**spring.config.name和spring.config.location很早就用来确定必须加载哪些文件，因此必须将它们定义为环境属性（通常是OS环境变量，系统属性或命令行参数）。**

如果**spring.config.location**包含目录（而不是文件），它们应该以/（并且在运行时，附加spring.config.name在加载之前生成的名称，包括特定于配置文件的文件名）结束。指定的文件 **spring.config.location**按原样使用，不支持特定于配置文件的变体，并且被任何特定于配置文件的属性覆盖。

以相反的顺序搜索配置位置。默认情况下，配置的位置是 **classpath:/,classpath:/config/,file:./,file:./config/**。生成的搜索顺序如下：

- file:./config/
- file:./
- classpath:/config/
- classpath:/

使用**spring.config.location**配置自定义配置位置时，它们会替换默认位置。例如，如果**spring.config.locatio**n配置了值**classpath:/custom-config/,file:./custom-config/**，则搜索顺序将变为以下内容：

- file:./custom-config/
- classpath:custom-config/

或者，当使用配置自定义配置位置时，除默认位置外，还会使用**spring.config.additional-location**。在默认位置之前搜索其他位置。例如，如果**classpath:/custom-config/,file:./custom-config/**配置了其他位置，则搜索顺序将变为以下内容：

- file:./custom-config/
- classpath:custom-config/
- file:./config/
- file:./
- classpath:/config/
- classpath:/

此搜索顺序允许您在一个配置文件中指定默认值，然后有选择地覆盖另一个配置文件中的值。您可以在其中一个默认位置为您的应用程序提供默认值**application.properties**（或您选择的任何其他基本名称**spring.config.name**）。然后，可以在运行时使用位于其中一个自定义位置的不同文件覆盖这些默认值。

**如果使用环境变量而不是系统属性，则大多数操作系统不允许使用句点分隔的键名，但您可以使用下划线（例如，SPRING_CONFIG_NAME而不是spring.config.name）。**

**如果应用程序在容器中运行，则可以使用JNDI属性（in java:comp/env）或servlet上下文初始化参数来代替环境变量或系统属性。**

> 特定于配置文件的属性

除**application.properties**文件外，还可以使用以下命名约定来定义特定于配置文件的属性：**application-{profile}.properties**。在 Environment具有一组默认的配置文件（默认[default]）如果没有活动的简档设置中使用。换句话说，如果没有显式激活配置文件，**application-default.properties**则会加载属性。

特定于配置文件的属性从标准的相同位置加载 **application.properties**，特定于配置文件的文件始终覆盖非特定的文件，无论特定于配置文件的文件是在打包的jar内部还是外部。

如果指定了多个配置文件，则应用last-wins策略。例如，**spring.profiles.activ**e属性指定的配置文件将在通过**SpringApplication** API配置的配置文件之后添加，因此优先。

**如果您在spring.config.location指定任何文件，则不考虑这些文件的特定于配置文件的变体。如果您还想使用特定于配置文件的属性，请使用spring.config.location目录 。**

> 属性中的占位符

使用时，值**application.properties**将通过现有值进行过滤Environment ，因此您可以返回先前定义的值（例如，从“系统”属性）。

```bash
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

### 使用YAML替代Properties

[YAML](http://yaml.org/)是JSON的超集，因此是用于指定分层配置数据的便捷格式。每当您的类路径中都有[SnakeYAML](http://www.snakeyaml.org/)库时，SpringApplication类将自动支持YAML作为properties的替代方法。

**如果您使用“Starters”，SnakeYAML将通过spring-boot-starter自动提供。**

> 加载YAML

Spring Framework提供了两个方便的类，可用于加载YAML文档。 **YamlPropertiesFactoryBean**将YAML作为Properties加载，**YamlMapFactoryBean**将YAML作为Map加载。

例如，下面YAML文档：

```yml
environments:
	dev:
		url: http://dev.example.com
		name: Developer Setup
	prod:
		url: http://another.example.com
		name: My Cool App
```

前面的示例将转换为以下属性：

```bash
environments.dev.url=http://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=http://another.example.com
environments.prod.name=My Cool App
```

YAML列表表示为具有[index] dereferencers的属性键，例如YAML：

```yml
my:
servers:
	- dev.example.com
	- another.example.com
```

将转化为属性：

```bash
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

要使用Spring DataBinder工具（@ConfigurationProperties做的）绑定到这样的属性，您需要有一个属性类型为java.util.List（或Set）的目标bean，并且您需要提供一个setter，或者 用可变值初始化它，例如 这将绑定到上面的属性

```java
@ConfigurationProperties(prefix="my")
public class Config {

	private List<String> servers = new ArrayList<String>();

	public List<String> getServers() {
		return this.servers;
	}
}
```

> 将YAML作为Spring环境中的属性文件

可以使用YamlPropertySourceLoader类在Spring环境中将YAML作为PropertySource暴露出来。 这允许您使用熟悉的**@Value**注解和占位符语法来访问YAML属性。

> 多个YAML文件

您可以使用**spring.profiles**键指定单个文件中的多个特定配置文件YAML文档，以指示文档何时应用。 例如：

```yml
server:
	address: 192.168.1.100
---
spring:
	profiles: development
server:
	address: 127.0.0.1
---
spring:
	profiles: production
server:
	address: 192.168.1.120
```

在上面的示例中，如果开发配置文件处于活动状态，则server.address属性将为127.0.0.1。 如果开发和生产配置文件未启用，则该属性的值将为192.168.1.100。

如果应用程序上下文启动时没有显式激活，默认配置文件将被激活。 所以在这个YAML中，我们为security.user.password设置一个仅在“默认”配置文件中可用的值：

```yml
server:
  port: 8000
---
spring:
  profiles: default
  security:
    user:
      password: weak
```

使用“spring.profiles”元素指定的Spring profiles 以选择使用！字符。 如果为单个文档指定了否定和非否定的配置文件，则至少有一个非否定配置文件必须匹配，没有否定配置文件可能匹配。

> YAML的缺点

使用**@PropertySource**注解无法加载YAML文件。因此，如果您需要以这种方式加载值，则需要使用属性文件。

### 类型安全的配置属性

使用**@Value(“${property}”)**注解来注入配置属性有时可能很麻烦，特别是如果您正在使用多个层次结构的属性或数据时。 Spring Boot提供了一种处理属性的替代方法，允许强类型Bean管理并验证应用程序的配置。

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

	private boolean enabled;

	private InetAddress remoteAddress;

	private final Security security = new Security();

	public boolean isEnabled() { ... }

	public void setEnabled(boolean enabled) { ... }

	public InetAddress getRemoteAddress() { ... }

	public void setRemoteAddress(InetAddress remoteAddress) { ... }

	public Security getSecurity() { ... }

	public static class Security {

		private String username;

		private String password;

		private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

		public String getUsername() { ... }

		public void setUsername(String username) { ... }

		public String getPassword() { ... }

		public void setPassword(String password) { ... }

		public List<String> getRoles() { ... }

		public void setRoles(List<String> roles) { ... }

	}
}
```

上述POJO定义了以下属性：

- **foo.enabled**，默认为false
- **foo.remote-address**，具有可以从String强转的类型
- **foo.security.username**，具有内置的“安全性(security)”，其名称由属性名称决定。 特别是返回类型并没有被使用，可能是SecurityProperties
- **foo.security.password**
- **foo.security.roles**，一个String集合

Getters和setter方法通常是必须要有的，因为绑定是通过标准的Java Beans属性描述符，就像在Spring MVC中一样。 在某些情况下可能会省略setter方法：

- Map 只要它们被初始化，需要一个getter，但不一定是一个setter，因为它们可以被binder修改。
- 集合和数组可以通过索引（通常使用YAML）或使用单个逗号分隔值（Properties中）来访问。 在后一种情况下，setter方法是强制性的。 我们建议总是为这样的类型添加一个设置器。 如果您初始化集合，请确保它不是不可变的（如上例所示）
- 如果已初始化嵌套POJO属性（如上例中的Security字段），则不需要setter方法。如果您希望binder使用其默认构造函数即时创建实例，则需要一个setter。

有些人使用Project Lombok自动添加getter和setter。 确保Lombok不会为这种类型生成任何特定的构造函数，因为构造函将被容器自动用于实例化对象。

另请参阅[@Value和@ConfigurationProperties之间的不同](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#boot-features-external-config-vs-value)。

您还需要列出在**@EnableConfigurationProperties**注解中注册的属性类：

```java
@Configuration
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

**当@ConfigurationProperties bean以这种方式注册时，该bean将具有常规名称：\<prefix\>-\<fqn\>，其中是@ConfigurationProperties注解中指定的环境密钥前缀，是bean的全名(fully qualified name)。 如果注解不提供任何前缀，则仅使用该bean的全名。上面示例中的bean名称将是foo-com.example.FooProperties。**

即使上述配置将为AcmeProperties创建一个常规bean，我们建议@ConfigurationProperties仅处理环境，特别是不从上下文中注入其他bean。话虽如此，**@EnableConfigurationProperties**注释也会自动应用于您的项目，以便使用@ConfigurationProperties注释的任何现有的bean都将从环境配置。 您可以通过确保AcmeProperties已经是一个bean来快速上面的MyConfiguration

```java
@Component
@ConfigurationProperties(prefix="acme")
public class AcmeProperties {

	// ... see the preceding example

}
```

这种配置方式与SpringApplication外部的YAML配置相当：

```yml
# application.yml

acme:
	remote-address: 192.168.1.1
	security:
		username: admin
		roles:
		  - USER
		  - ADMIN

# additional configuration as required
```

要使用**@ConfigurationProperties** bean，您可以使用与任何其他bean相同的方式注入它们，如以下示例所示：

```java
@Service
public class MyService {

	private final AcmeProperties properties;

	@Autowired
	public MyService(AcmeProperties properties) {
	    this.properties = properties;
	}

 	//...

	@PostConstruct
	public void openConnection() {
		Server server = new Server(this.properties.getRemoteAddress());
		// ...
	}

}
```

使用@ConfigurationProperties还可以生成IDE可以为自己的密钥提供自动完成的元数据文件，有关详细信息，请参见[附录B，配置元数据附录](https://docs.spring.io/spring-boot/docs/2.0.x/reference/htmlsingle/#configuration-metadata)。

> 第三方配置

除了使用@ConfigurationProperties来注解类，还可以在public @Bean方法中使用它。 当您希望将属性绑定到不受控制的第三方组件时，这可能特别有用。

要从Environment属性配置bean ，请添加@ConfigurationProperties到其bean注册，如以下示例所示：

```java
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
	...
}
```

使用另一个前缀定义的任何属性都以类似于前面**AcmeProperties**示例的方式映射到另一个组件bean。

> 宽松的绑定

Spring Boot使用一些宽松的规则来绑定bean的Environment属性(**@ConfigurationProperties**)，因此不需要在Environment属性名和bean属性名之间进行精确匹配 。这有用的常见示例包括破折号分隔的环境属性（例如，context-path绑定到contextPath）和大写环境属性（例如，PORT绑定到 port）。

例如，请考虑以下@ConfigurationProperties类：

```java
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

	private String firstName;

	public String getFirstName() {
		return this.firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

}
```

在前面的示例中，可以使用以下属性名称：

表格 24.1. relaxed binding

|Property|Note|
|---|---|
|acme.my-project.person.first-name|推荐在.properties和.yml文件中使用。|
|acme.myProject.person.firstName|推荐在.properties和.yml文件中使用。标准的驼峰式大小写语法。|
|acme.my_project.person.first_name|下划线符号，是.properties和.yml文件中使用的另一种格式。|
|ACME_MYPROJECT_PERSON_FIRSTNAME|大小写格式，在使用系统环境变量时推荐使用大小写格式。|

**prefix注解的值必须是kebab大小写（小写并且用-，例如acme.my-project.person）。**

表24.2. 放宽每个属性源的绑定规则

|Property Source|Simple|List|
|---|---|---|
|Properties Files|驼峰，下划线符号|使用[]或逗号分隔值的标准列表语法|
|YAML Files|驼峰，下划线符号|标准的YAML列表语法或逗号分隔的值|
|Environment Variables|以下划线作为分隔符的大小写格式。_不应该在属性名中使用|由下划线包围的数值，例如MY_ACME_1_OTHER = my.acme[1].other|
|System properties|驼峰，下划线符号|使用[]或逗号分隔值的标准列表语法|

**我们建议，在可能的情况下，属性以小写的kebab格式存储，例如my.property-name=acme。**

在绑定到Map映射属性时，如果键不包含小写字母数字字符或-，则需要使用括号符号，以便保留原始值。如果键没有被[]包围，任何非字母数字或-的字符都会被删除。例如，考虑将以下属性绑定到映射:

```yml
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```

上面的属性将绑定到**Map** with **/key1**，/**key2**, **key3**作为Map的键。

> 合并复杂类型

当列表配置在多个位置时，覆盖通过替换整个列表来工作。

例如，假设**MyPojo**对象**name**和**description**属性默认为**null**。下面的示例公开了AcmeProperties中的MyPojo对象列表:

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final List<MyPojo> list = new ArrayList<>();

	public List<MyPojo> getList() {
		return this.list;
	}

}
```

请考虑以下配置：

```yml
acme:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

如果dev配置文件未激活，则AcmeProperties.list包含一个MyPojo条目，如先前定义的那样。但是，如果启用了dev配置文件，则list仍然只包含一个条目。此配置不会MyPojo向列表添加第二个实例，也不会合并项目。

当**List**在多个配置文件中指定a时，将使用具有最高优先级（并且仅具有该优先级）的配置文件。请考虑以下示例：

```yml
acme:
  list:
    - name: my name
      description: my description
    - name: another name
      description: another description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

在前面的示例中，如果dev配置文件处于活动状态，则AcmeProperties.list包含一个MyPojo条目。对于YAML，逗号分隔列表和YAML列表都可用于完全覆盖列表的内容。

对于Map属性，您可以绑定从多个源中提取的属性值。但是，对于多个源中的相同属性，使用具有最高优先级的属性。以下示例公开了一个Map\<String, MyPojo\> from AcmeProperties：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final Map<String, MyPojo> map = new HashMap<>();

	public Map<String, MyPojo> getMap() {
		return this.map;
	}

}
```

请考虑以下配置：

```yml
acme:
  map:
    key1:
      name: my name 1
      description: my description 1
---
spring:
  profiles: dev
acme:
  map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2
```

如果dev配置文件未激活，则AcmeProperties.map包含一个带密钥的条目key1。但是，如果启用了dev配置文件，则map包含两个带有键的条目key1和key2。

**前面的合并规则适用于所有属性源的属性，而不仅仅是YAML文件。**

> 属性转换

Spring Boot尝试在绑定到**@ConfigurationProperties** bean时将外部应用程序属性强制转换为正确的类型。如果需要自定义类型转换，则可以提供ConversionServicebean（带有bean命名conversionService）或自定义属性编辑器（通过**CustomEditorConfigurerbean**）或自定义Converters（带有注解为bean的bean定义@ConfigurationPropertiesBinding）。

**由于在应用程序生命周期中很早就请求了此bean，因此请确保限制您ConversionService正在使用的依赖项。通常，您在创建时可能无法完全初始化所需的任何依赖项。ConversionService如果配置密钥强制不需要，您可能希望重命名自定义，并且只依赖于合格的自定义转换器@ConfigurationPropertiesBinding。**

Spring Boot专门支持表达持续时间。如果公开**java.time.Duration**属性，则可以使用应用程序属性中的以下格式：

- 常规Long表示(使用毫秒作为默认单位，除非指定了**@DurationUnit**)
- [java.util.Duration](https://docs.oracle.com/javase/8/docs/api//java/time/Duration.html#parse-java.lang.CharSequence-)使用的标准ISO-8601格式
- 一种更易于阅读的格式，其中值和单元是耦合的(例如，10秒表示10秒)

请考虑以下示例：

```java
@ConfigurationProperties("app.system")
public class AppSystemProperties {

	@DurationUnit(ChronoUnit.SECONDS)
	private Duration sessionTimeout = Duration.ofSeconds(30);

	private Duration readTimeout = Duration.ofMillis(1000);

	public Duration getSessionTimeout() {
		return this.sessionTimeout;
	}

	public void setSessionTimeout(Duration sessionTimeout) {
		this.sessionTimeout = sessionTimeout;
	}

	public Duration getReadTimeout() {
		return this.readTimeout;
	}

	public void setReadTimeout(Duration readTimeout) {
		this.readTimeout = readTimeout;
	}

}
```

要指定30秒的会话超时，30秒、PT30S和30s都是等效的。读取超时500ms可以用以下任何一种形式指定:500ms、pt0.5 5s和500ms。

您也可以使用任何支持的设备。这些是：

- **ns** 纳秒
- **us** 微秒
- **ms** 毫秒
- **s** 秒
- **m** 分钟
- **h** 小时
- **d** 天

默认单位是毫秒，可以使用**@DurationUnit**上面的示例中所示进行覆盖。

**如果要从仅Long用于表示持续时间的先前版本升级，请确保定义单位（使用@DurationUnit），如果它不是交换机旁边的毫秒数Duration。这样做可以提供透明的升级路径，同时支持更丰富的格式。**

> @ConfigurationProperties验证

Spring Boot尝试在**@ConfigurationProperties**使用Spring的**@Validated**注解时验证类。您可以**javax.validation**直接在配置类上使用JSR-303约束注解。为此，请确保符合条件的JSR-303实现位于类路径上，然后将约束注释添加到字段中，如以下示例所示：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

	@NotNull
	private InetAddress remoteAddress;

	// ... getters and setters

}
```

**您还可以通过注解@Bean创建配置属性的方法来触发验证@Validated。**

虽然嵌套属性也会在绑定时进行验证，但最好还是将关联字段注解为**@Valid**。这可确保即使未找到嵌套属性也会触发验证。以下示例基于前面的AcmeProperties示例构建 ：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

	@NotNull
	private InetAddress remoteAddress;

	@Valid
	private final Security security = new Security();

	// ... getters and setters

	public static class Security {

		@NotEmpty
		public String username;

		// ... getters and setters

	}

}
```

还可以通过创建一个名为**configurationPropertiesValidator**的bean定义来添加定制的Spring验证器。@Bean方法应该声明为静态方法。配置属性验证器是在应用程序生命周期的很早就创建的，并且将@Bean方法声明为静态，这样就可以创建bean，而不必实例化**@Configuration**类。这样做可以避免早期实例化可能导致的任何问题。有一个属性验证示例演示如何设置。

**spring-boot-actuator模块包括一个暴露所有@ConfigurationPropertiesbean的端点 。将Web浏览器指向 /actuator/configprops或使用等效的JMX端点。**

> @ConfigurationProperties vs @Value

**@Value**是核心容器功能，它不提供与类型安全配置属性相同的功能。 下表总结了**@ConfigurationProperties**和**@Value**支持的功能：

|Feature	|@ConfigurationProperties|@Value|
|---|---|---|
|Relaxed binding|Yes|No|
|Meta-data support|Yes|No|
|SpEL evaluation|No|Yes|

如果您为自己的组件定义了一组配置键，我们建议您将它们分组到带**@ConfigurationProperties**注解的POJO中。您还应该知道，因为**@Value**不支持宽松绑定，所以如果您需要使用环境变量来提供值，则它不是一个好的候选者。

最后，虽然您可以编写SpEL表达式@Value，但不会从应用程序属性文件处理此类表达式。

