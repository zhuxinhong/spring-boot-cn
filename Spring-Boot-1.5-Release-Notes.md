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

> 详情请参考[changelog](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-1.5-Configuration-Changelog)。

###	升级第三方依赖

一些第三方库升级到最新版本。包括 Spring Data Ingalls, Jetty 9.4, JooQ 3.9, AssertJ 2.6.0, Hikari 2.5 and Neo4J 2.1。少数几个 Maven 插件也进行了升级。

###	Loggers endpoint

新的 loggers actuator endpoint 允许即时查看和更改应用日志级别。有可用的 JMX 和 MVC endpoint。例如，要使用 MVC endpoint 更改日志级别，可以使用以下JSON POST到 /loggers/com.yourcorp.application

```json
{
  "configuredLevel": "DEBUG"
}
```

要使用 JMX endpoint 更新记录器，需使用 setLogLevel 操作。有关详细信息，请参阅[文档](http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#production-ready-loggers)。

###	支持 Apache Kafka

Spring Boot 1.5 新增了 spring-kafka 项目，为 Apache Kafka 提供自动配置。使用 Kafka 仅需要添加 spring-kafka 的依赖，并配置 `spring.kafka * 属性即可。

从卡夫卡接收消息与注释方法一样简单：

```Java
@Component
public class MyBean {

    @KafkaListener(topics = "someTopic")
    public void processMessage(String content) {
        // ...
    }

}
```

###	扩展 Cloud Foundry actuator

当你部署一个兼容的 Cloud Foundary 实时时，Spring Boot 的 actuator 模块提供了额外的支持。/cloudfoundryapplication 路径为所有 NamedMvcEndpoint beans 提供了一个替代的安全路由。

Cloud Foundry 的管理界面可以利用 endpoint 来显示附加的 actuator 信息。

有关 Cloud Foundry endpoint 更多信息，请参考[文档](http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#production-ready-cloudfoundry)。各种场景的的示例，可以阅读有关 PCF1.9 的[博客文章](https://blog.pivotal.io/pivotal-cloud-foundry/products/pivotal-cloud-foundry-1-9-sets-the-bar-on-massive-scale)。

###	支持 LDAP

Spring Boot 现在可以为任何兼容的 LDAP 服务器提供自动配置，以及从 Unbounded 支持嵌入式内存 LDAP 服务器。

详情参考[文档](http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#boot-features-ldap)。

###	支持 AuditEvents Endpoint 

现在可以用一个新的 AuditEventsJmxEndpoint bean 来记录 AuditEvents。MBean 通过 AuditEventRepository find 方法提供入口例如 getData。身份验证和授权事件会被自动记录，也可以使用 AuditEventRepository 记录自定义事件。该信息也可用新的 /auditevents MVC endpoint 查看。

###	事务管理配置

现在可以使用 spring.transaction.* 属性来配置 PlatformTransactionManager。目前支持 “default-timeout” 和 rollback-on-commit-failure 属性。

###	JmxEndpoint interface

引入了一个新的 JmxEndpoint 接口，在 JMX 暴露的 actuator endpoint 基础上进行开发。该接口和 MVC endpoint 提供的 MvcEndpoint 接口非常相似。

###	特殊迁移方式

现在可以定义特定数据库的迁移。要使用供应商特定的迁移，请按如下所示设置 flyway.locations 属性：

```
flyway.locations=db/migration/{vendor}
```

详见[文档](http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#howto-execute-flyway-database-migrations-on-startup)。

###	测试升级

现在可以排除由 @Test... 等注解导入的自动配置。现在所有现有的 @Test... 注释都包含一个 excludeAutoConfiguration 属性。 或者，可以直接将 @ImportAutoConfiguration（exclude = ...）添加到测试中。

Spring Boot 1.5 还引入了一个新的 @JdbcTest 注释，可以用来直接测试 JDBC。

### 自定义 fat jar 

Spring Boot Maven 和 Gradle 插件现在支持自定义 fat jar。此功能允许在Spring Boot之外开发诸如此类的 [layout](https://github.com/dsyer/spring-boot-thin-launcher)。 有关详细信息，请参阅更新的[文档](http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-gradle-configuration-custom-repackager)。

###	自定义 JmsTemplate

可以使用 spring.jms.template.* 命名空间中提供的其他自定义属性配置 JmsTemplate。

###	其他

*	Mockito 2.x 可以与 @MockBean 一起使用（与Mockito 1.9保持兼容）。
* 	内置启动脚本支持 forse-stop。
*  	新增 Cassandra 的健康检查 bean。
*	Cassandra 用户自定义的类型问题现已解决（例如 Spring Data 的 SimpleUserTypeResolver）。
* 	skip 属性现在适用于 Spring Boot Maven插件 run，stop 和 repackage 指令。
*  	如果找到多个 main 方法类，Maven 和 Gradle 插件现在将自动使用被 @SpringBootApplication 注释的 main 方法类。

##	Spring Boot 1.5 中废弃的

*	为了 setTldSkipPatterns，废弃了 TomcatEmbeddedServletContainerFactory.setTldSkip。
* 	ApplicationStartingEvent 代替了 ApplicationStartedEvent。
*  	LoggingApplicationListener 中的少数常量被 LogFile 版本替代。
*	由于 Guava 将在 Spring Framework 5 中删除，Guava 缓存已被弃用。请升级到 Caffeine。
* 	由于 CRaSH 不再积极维护，所以废弃了。
*  	引入 JmxEndpoint 后，EndpointMBeanExporter 中的一些 protected 方法已被弃用。
*  	SearchStrategy.ANCESTORS. 替代了 SearchStrategy.PARENTS 。
*  	引入了 DBCP 2，废弃了 Apache DBCP 的支持。
*  	server.undertow.buffers-per-region 因为[没有生效](https://issues.jboss.org/browse/UNDERTOW-587)，所以废弃该配置。
*  	@AutoConfigureTestDatabase 从 org.springframework.boot.test.autoconfigure.orm.jpa 迁移到了 org.springframework.boot.test.autoconfigure.jdbc 包中。

##	属性重命名

*	server.max-http-post-size 属性已被特定的配置替代（例如 server.tomcat.max-http-post-size）	
*	删除了 spring.data.neo4j.session.scope。