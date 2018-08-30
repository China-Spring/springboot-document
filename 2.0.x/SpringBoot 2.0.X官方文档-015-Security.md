# SpringBoot 2.0.X官方文档-015-Security

如果Spring Security位于类路径上，则默认情况下，Web应用程序将在所有HTTP端点上使用“basic”身份验证。 要向Web应用程序添加方法级安全性，您还可以使用所需的设置添加**@EnableGlobalMethodSecurity**。有关更多信息，请参见“[Spring Security Reference](https://docs.spring.io/spring-security/site/docs/5.0.7.RELEASE/reference/htmlsingle#jc-method)”。

默认的UserDetailsService只有一个用户。用户名是user，密码是随机的，在应用程序启动时在INFO级别打印，如下例所示:

```bash
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

**如果您对日志配置进行了微调，请确保org.springframework.boot.autoconfigure.security类别设置为记录信息级别的消息。否则，不会打印默认密码。**

您可以通过提供**spring.security.user.name**和**spring.security.user.password**更改用户名和密码。

您在Web应用程序中默认获得的基本功能包括：

- 一个UserDetailsService（在WebFlux应用程序的情况下是ReactiveUserDetailsService）具有内存存储的bean和具有生成密码的单个用户（参见 [SecurityProperties.User](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/api/org/springframework/boot/autoconfigure/security/SecurityProperties.User.html)用户的属性）。
- 基于表单的登录或HTTP基本安全性（取决于Content-Type），用于整个应用程序（如果执行器在类路径上，则包括执行器端点）。
- 一个**DefaultAuthenticationEventPublisher**用于发布身份验证事件。

你可以通过**AuthenticationEventPublisher**为它添加一个bean来提供不同的东西。

> MVC安全性

默认安全配置在**SecurityAutoConfiguration**和中**UserDetailsServiceAutoConfiguration**实现.
**SecurityAutoConfiguration**导入**SpringBootWebSecurityConfiguration**用于web安全，**UserDetailsServiceAutoConfiguration**配置身份验证，这在非web应用程序中也是相关的。要完全关闭默认的web应用程序安全配置，可以添加**WebSecurityConfigurerAdapter**类型的bean(这样做不会禁用**UserDetailsService**配置或执行器的安全)。

要关闭UserDetailsService配置，可以添加UserDetailsService、AuthenticationProvider或AuthenticationManager类型的bean。在[Spring Boot示例](https://github.com/spring-projects/spring-boot/tree/v2.0.4.RELEASE/spring-boot-samples/)中有几个安全的应用程序可以让您从常见的用例开始。

可以通过添加自定义来覆盖访问规则**WebSecurityConfigurerAdapter**。Spring Boot提供了便捷方法，可用于覆盖执行器端点和静态资源的访问规则。**EndpointRequest**可用于创建基于**management.endpoints.web.base-path**属性的**RequestMatcher**。 **PathRequest**可用于RequestMatcher在常用位置创建资源。

> WebFlux安全性

与Spring MVC应用程序类似，您可以通过添加**spring-boot-starter-security**依赖项来保护WebFlux应用程序。默认的安全配置在**ReactiveSecurityAutoConfiguration**和**UserDetailsServiceAutoConfiguration**中实现。**ReactiveSecurityAutoConfiguration**导入**WebFluxSecurityConfiguration**用于web安全，**UserDetailsServiceAutoConfiguration**配置身份验证，这在非web应用程序中也是相关的。要完全关闭默认的web应用程序安全配置，可以添加**WebFilterChainProxy**类型的bean(这样做不会禁用**UserDetailsService**配置或执行器的安全)。

为了关闭UserDetailsService配置，您可以添加一个类型为**ReactiveUserDetailsService**或**ReactiveAuthenticationManager**的bean。

可以通过添加自定义**SecurityWebFilterChain**来配置访问规则。Spring Boot提供了方便的方法，可用于覆盖执行器端点和静态资源的访问规则。可以使用EndpointRequest创建一个基于**management.endpoints.web**的**ServerWebExchangeMatcher**。基本路径属性。

PathRequest可用于为常用位置的资源创建ServerWebExchangeMatcher。

例如，您可以通过添加以下内容来自定义安全配置：

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
	return http
		.authorizeExchange()
			.matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
			.pathMatchers("/foo", "/bar")
				.authenticated().and()
			.formLogin().and()
		.build();
}
```

###  OAuth2

[OAuth2](https://oauth.net/2/)是Spring支持的一种广泛使用的授权框架。

> Client客户端

如果您的类路径中有**spring-security-oauth2-client**，那么可以利用一些自动配置来轻松地设置OAuth2客户机。这个配置使用**OAuth2ClientProperties**下的属性。

您可以在**spring.security.oauth2.client**前缀下注册多个OAuth2客户端和提供程序 ，如以下示例所示：

```bash
spring.security.oauth2.client.registration.my-client-1.client-id=abcd
spring.security.oauth2.client.registration.my-client-1.client-secret=password
spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-1.scope=user
spring.security.oauth2.client.registration.my-client-1.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code

spring.security.oauth2.client.registration.my-client-2.client-id=abcd
spring.security.oauth2.client.registration.my-client-2.client-secret=password
spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-2.scope=email
spring.security.oauth2.client.registration.my-client-2.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code

spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=http://my-auth-server/oauth/authorize
spring.security.oauth2.client.provider.my-oauth-provider.token-uri=http://my-auth-server/oauth/token
spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=http://my-auth-server/userinfo
spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=http://my-auth-server/token_keys
spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name
```

默认情况下，Spring Security的OAuth2LoginAuthenticationFilter只处理匹配/login/oauth2/code/*的url。如果您想定制redirect-uri-template来使用不同的模式，您需要提供配置来处理定制模式。例如，您可以添加类似于以下内容的WebSecurityConfigurerAdapter:

```java
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.oauth2Login()
				.redirectionEndpoint()
					.baseUri("/custom-callback");
	}
}
```

对于普通的OAuth2和OpenID提供者，包括谷歌、Github、Facebook和Okta，我们提供了一组提供者默认值(谷歌、Github、Facebook和Okta)。

如果您不需要定制这些提供者，您可以将提供者属性设置为您需要推断默认值的提供者属性。另外，如果客户机的ID与默认支持的提供程序匹配，Spring Boot也会推断出这一点。

换句话说，以下示例中的两个配置使用谷歌提供程序:

```java
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```

> Server服务器端

目前，Spring Security不支持实施OAuth 2.0授权服务器或资源服务器。但是，此功能可从[Spring Security OAuth](https://projects.spring.io/spring-security-oauth/)项目获得，该项目最终将完全被Spring Security取代。在此之前，您可以使用**spring-security-oauth2-autoconfigure**模块轻松设置OAuth 2.0服务器; 请参阅其[文档](https://docs.spring.io/spring-security-oauth2-boot)以获取说明

### 执行器安全性

出于安全考虑，默认情况下除了/health和/info禁用所有执行器。**management.endpoints.web.exposure.include**属性可用于启用执行器。

如果Spring Security位于类路径上且没有其他**WebSecurityConfigurerAdapter**存在，则执行程序由Spring Boot auto-config保护。如果您定义自定义 WebSecurityConfigurerAdapter，Spring Boot自动配置将退回，您将完全控制执行器访问规则。

**在设置management.endpoints.web.exposure.include之前，请确保暴露的执行器不包含敏感信息或通过将它们放在防火墙后面或通过Spring Security等方式进行保护。**

> 跨域请求伪造保护

由于Spring Boot依赖于Spring Security的默认值，因此默认情况下会启用CSRF保护。这意味着执行器端点需要POST（关闭和记录器端点），PUT或者DELETE在使用默认安全配置时将获得403禁止错误。

**我们建议仅在创建非浏览器客户端使用的服务时才完全禁用CSRF保护。**

有关CSRF保护的其他信息，请参阅“[Spring Security参考指南](https://docs.spring.io/spring-security/site/docs/5.0.7.RELEASE/reference/htmlsingle#csrf)”。


