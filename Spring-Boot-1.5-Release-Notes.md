#	Spring Boot 1.5 Release Notes

##	从Spinrg Boot 1.4 升级

###	Spring Boot 1.4 中废弃的

在 Spring Boot 1.4 被声明为 deprecated 的类、方法和属性在当前版本中被移除了。在升级前请确保这些被废弃的方法没有被调用。尤其是，移除了 HornetQ 和 Velocity 的支持。

###	starters 重命名

以下在 Spring Boot 1.4 中已经重命名的现在被移除了，如果你遇到“不能解析的依赖”错误请确认你拉取的是正确命名的 starter：

*	spring-boot-starter-ws → spring-boot-starter-web-services

*	spring-boot-starter-redis → spring-boot-starter-data-redis

###	@ConfigurationProperties 校验

如果有在类上使用 @ConfigurationProperties 注解以遵循 JSR-303 约束，那么现在应该再配合使用 @Validated。目前的验证仍会生效，但是会发出警告。在将来，没有 @Validated 注解的类将不会被验证。

###	Spring Session 存储

以前，如果没有专门配置 Spring Session 和 Redis，Redis 会自动用于存储 session。现在需要指定存储类型; 使用 Redis 的 Spring Session 的现有用户应在其配置中添加以下内容：

```
spring.session.store-type=redis
```

###	Actuator 安全

Actuator 敏感信息的 endpoints 现在是安全的（并不需要 Spring Security 依赖）。如果你现有的 Spring Boot 1.4 工程使用了Spring Security（并且没有任何自定义安全配置），那么应该还是像以前一样。如果你现有的 Spring Boot 1.4 工程中有自定义的安全配置，并希望对敏感 endpoints 有开放的访问权限，则需要在安全配置中明确声明。如果你正在升级不依赖 Spring Security 的 Spring Boot 1.4 应用，并且希望保留对敏感端点的开放访问权限，则需要将 management.security.enabled 设置为 false。详情参考[文档](http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#production-ready-sensitive-endpoints)。

访问端点所需的默认角色也从 ADMIN 更改为 ACTUATOR。如果你碰巧将 ADMIN 角色用于其他目的，则可以防止端点的意外暴露。如果要恢复 Spring Boot 1.4 的行为，请将 management.security.roles属性设置为 ADMIN。

###	InMemoryMetricRepository

InMemoryMetricRepository 不再直接实现 MultiMetricRepository。现在注册了一个新的InMemoryMultiMetricRepository bean，满足 MultiMetricRepository 接口并由常规 InMemoryMetricRepository 支持。由于大多数用户将与 MetricRepository 或 MultiMetricRepository 接口进行交互（而不是内存中的实现），因此此更改应该是透明的。

###	spring.jpa.database

现在可以从 spring.datasource.url 属性自动检测 spring.jpa.database 的常用数据库类型。 如果你已手动定义 spring.jpa.database，并且使用常见数据库，则可以尝试删除该属性。

少数几个数据库有多个方言（例如，Microsoft SQL Server 有3个），因此我们可能会配置一个与你正在使用数据库版本不匹配的配置。如果以前有一个配置，并希望依靠 Hibernate 自动检测方言，请设置 spring.jpa.database=default。 或者可以随时使用 spring.jpa.database-platform 属性设置方言。

###	@IntegrationComponentScan

Spring 集成用的 @IntegrationComponentScan 注解现在会自动配置。如果你遵循了[推荐的工程结构](http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#using-boot-structuring-your-code)，就可以删除它。

###	ApplicationStartedEvent

如果在当前代码中监听了 ApplicationStartedEvent，则应重构使用ApplicationStartingEvent。我们更名为这个类，让语义更加明显。

###	Spring Integration Starter

spring-boot-starter-integration POM 不再包含 spring-integration-jmx。如果需要 Spring Integration JMX 支持，应该自己添加一个 spring-integration-jmx 依赖。

###	Devtools excluded by default

现在默认的 Maven 和 Gradle 插件都不包括 spring-boot-devtools jar 在 “fat”jar 中打包。如果正在使用 devtools 远程支持，现在需要在 build.gradle 或 pom.xml 文件中显式设置 excludeDevtools 属性。

###	Gradle 1.x

Spring Boot Gradle 插件不再兼容 Gradle 1.x 和 Gradle 2.x 或更早版本。请确认你使用了 Gradle 2.9 或更高版本。

###	Remote CRaSH shell

不幸的是，Spring Boot 提供的远程SSH支持项目 [CRaSH](http://www.crashub.org/) 将不再维护。遗憾的是，我们也不赞成使用远程SSH，并计划在 Spring Boot 2.0 中完全删除。

###	OAuth 2 Resource Filter

OAuth2 资源过滤器的默认顺序从3更改为 SecurityProperties.ACCESS_OVERRIDE_ORDER - 1。它将其放置在执行器端点之后，但位于基本认证过滤器链之前。可以通过设置 security.oauth2.resource.filter-order=3 来恢复默认值。

###	JSP servlet

默认情况下，JSP servlet 不再处于开发模式。开发模式在使用 DevTools 时自动启用。也可以通过设置 server.jsp-servlet.init-parameters.development=true 来显式启用它。

###	Ignored paths and @EnableWebSecurity

在 Spring Boot 1.4 及更早版本中，执行器将始终配置一些忽略的路径，而忽略掉 @EnableWebSecurity。这在1.5中得到纠正，以便使用 @EnableWebSecurity 将关闭Web安全性的所有自动配置，从而遵循配置。

##	新特性