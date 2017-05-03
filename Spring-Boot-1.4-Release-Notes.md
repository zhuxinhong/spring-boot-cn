#	Spring Boot 1.4 Release Notes

##	从Spinrg Boot 1.3升级

###	Spring Boot 1.3 中废弃的

在 Spring Boot 1.3 被声明为 deprecated 的类、方法和属性在当前版本中被移除了。在升级前请确保这些被废弃的方法没有被调用。

按照 [Apache EOL announcement](https://blogs.apache.org/foundation/entry/apache_logging_services_project_announces)的说明，移除了 Log4j 1 的支持。

###	重命名 starters

以下的 starters 被重命名，老的那些会在 Spring Boot 1.5 中被移除。

* spring-boot-starter-ws → spring-boot-starter-web-services
* spring-boot-starter-redis → spring-boot-starter-data-redis

###	获取 DataSource 属性的方法

为了保持和其他使用了 @ConfigurationProperties 注解的类， DataSourceProperties 中的 一些get 方法做了一些调整。如果你之前在代码里调用了以下的任何方法，你需要迁移到以 determine 命名的新方法：

*	getDriverClassName() → determineDriverClassName()
*	getUrl() → determineUrl()
*	getUsername() → determineUsername()
*	getPassword() → determineUsername()


>get方法并没有被废弃，但是他们的实现已经变了，请确认在升级时检查这些使用情况。


###	DataSource 绑定

在 Spring Boot 1.4 之前，数据源的自动配置是绑定在 spring.datasource 的命名空间下。在1.4，只需在 spring.datasource 下做通用配置，同时针对以下4个连接池提供了新的命名空间：

*	spring.datasource.tomcat -- org.apache.tomcat.jdbc.pool.DataSource
*	spring.datasource.hikari -- com.zaxxer.hikari.HikariDataSource
*	spring.datasource.dbcp -- org.apache.commons.dbcp.BasicDataSource
*	spring.datasource.dbcp2 -- org.apache.commons.dbcp2.BasicDataSource

如果你使用了相关连接池实现的特定配置，需要把配置移动到对应的命名空间下。例如，如果你使用了 Tomcat 中的 testOnBorrow，你需要把它从 spring.datasource.test-on-borrow 移动到 spring.datasource.tomcat.test-on-borrow。

如果在 IDE 中使用了配置帮助工具，可以看见在 spring.datasource 命名空间下的配置有哪些还可用。这会让你更容易找到不同的实现都提供哪些配置。

### JTA 配置绑定

和 DataSource 的绑定略像。JTA 中对 Atomikos 和 Bitronix 的配置之前被绑定到 spring.jta，现在分别绑定到 spring.jta.atomikos.properties 和 spring.jta.bitronix.properties，同时升级了对这些实现的元数据配置。

###	Hibernate 5

Hibernate 5.0 现在是 JPA 的默认实现。如果是从 Spring Boot 1.3 升级，Hibernate 版本会从 4.3 升级到 5.0。具体升级可参考 [Hibernate 文档](https://github.com/hibernate/hibernate-orm/blob/5.0/migration-guide.adoc)。此外你需要了解以下内容：

####	命名策略

Hibernate 5.1 已经不再使用 SpringNamingStrategy，并且移除了对老 NamingStrategy 的支持。作为 Hibernate 默认 ImplicitNamingStrategy，自动配置提供了新的 SpringPhysicalNamingStrategy。这些和 Spring Boot 1.3 默认配置非常相似，升级时你得检查数据库 schema 是否正确。

如果升级前使用了 Hibernate 5，Hibernate 5 默认配置可能会生效。如果你想在升级后恢复他们，需要做如下配置：

```
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

#### Generator mappings

为了最小化升级代价，在 Hibernate5 中 hibernate.id.new_generator_mappings 为 false。Hibernate 团队通常不建议使用这个配置，如果你乐意使用 generator 新特性，你可能会需要在 application.properties 文件中把 spring.jpa.hibernate.use-new-id-generator-mappings 设为 true。

#### 降级到 Hibernate 4.3

如果升级到 Hibernate 5.0 之后出现了重大问题，可以通过在 pom.xml 中重写 hibernate.version 把 Hibernate 版本降级：

```
<properties>
	<hibernate.version>4.3.11.Final</hibernate.version>
</properties>
```

或者在 gradle 脚本中 重写 hibernate.version：

```
ext['hibernate.version'] = '4.3.11.Final'
```

>Spring Boot 1.4 将不再支持 Hibernate 1.4。如果你发现有 bug 影响到升级了，可以在[这](https://github.com/spring-projects/spring-boot/issues/new)提出问题。

#### @EntityScan

@org.springframework.boot.orm.jpa.EntityScan 注解被废弃，可使用 @org.springframework.boot.autoconfigure.domain.EntityScan 来代替或特定配置。

举个栗子，如果你有如下配置：

```Java
import org.springframework.boot.autoconfigure.SpringApplication;
import org.springframework.boot.orm.jpa.EntityScan;

@SpringBootApplication
@EntityScan(basePackageClasses=Customer.class)
public class MyApplication {

	// ....

}
```

如果你正使用 LocalContainerEntityManagerFactoryBean 的自动配置，切换到：

```Java
import org.springframework.boot.autoconfigure.SpringApplication;
import org.springframework.boot.autoconfigure.domain.EntityScan;

@SpringBootApplication
@EntityScan(basePackageClasses=Customer.class)
public class MyApplication {

	// ....

}
```

如果你定义了自己的 LocalContainerEntityManagerFactoryBean，可以完全抛弃 @EntityScan 注解或者调用 LocalContainerEntityManagerFactoryBean.setPackagesToScan(…​) 或使用 EntityManagerFactoryBuilder packages(…​) 方法：

```Java
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(
			EntityManagerFactoryBuilder builder) {
	return builder
		.dataSource(...)
		.properties(...)
		.packages(Customer.class)
		.build();
}
```

###	Test utilities and classes

Spring Boot 1.4 搭配了个新的 spring-boot-test 模块，其中包含了完全重构的 org.springframework.boot.test 包。从 Spring Boot 1.3 升级的时候，需要迁移那些在老版本中已经被弃用的类。如果你使用 Linux 或 OSX，可以使用以下命令来迁移代码：

```Shell
find . -type f -name '*.java' -exec sed -i '' \
-e s/org.springframework.boot.test.ConfigFileApplicationContextInitializer/org.springframework.boot.test.context.ConfigFileApplicationContextInitializer/g \
-e s/org.springframework.boot.test.EnvironmentTestUtils/org.springframework.boot.test.util.EnvironmentTestUtils/g \
-e s/org.springframework.boot.test.OutputCapture/org.springframework.boot.test.rule.OutputCapture/g \
-e s/org.springframework.boot.test.SpringApplicationContextLoader/org.springframework.boot.test.context.SpringApplicationContextLoader/g \
-e s/org.springframework.boot.test.SpringBootMockServletContext/org.springframework.boot.test.mock.web.SpringBootMockServletContext/g \
-e s/org.springframework.boot.test.TestRestTemplate/org.springframework.boot.test.web.client.TestRestTemplate/g \
{} \;
```

另外，Spring Boot 1.4 尝试简化 Spring Boot 的各种测试方式。你或许需要迁移以下注解到新的 @SpringBootTest 注解：

*	@SpringApplicationConfiguration(classes=MyConfig.class) -> @SpringBootTest(classes=MyConfig.class)

*	@ContextConfiguration(classes=MyConfig.class, loader=SpringApplicationContextLoader.class) ->  @SpringBootTest(classes=MyConfig.class)

*	@IntegrationTest to @SpringBootTest(webEnvironment=WebEnvironment.NONE)

*	@IntegrationTest with @WebAppConfiguration ->  @SpringBootTest(webEnvironment=WebEnvironment.DEFINED_PORT) (or RANDOM_PORT)

*	@WebIntegrationTest ->  @SpringBootTest(webEnvironment=WebEnvironment.DEFINED_PORT) (or RANDOM_PORT)

>
迁移测试时，你需要使用 Spring 4.3 更易读的 @RunWith(SpringRunner.class) 代替 @RunWith(SpringJUnit4ClassRunner.class)。

关于 @SpringBootTest 注解的更多详情，参考[文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-testing-spring-boot-applications)。

###	TestRestTemplate

TestRestTemplate 不再直接继承 RestTemplate （尽管他们提供同样的方法）。这可以将 TestRestTemplate 配置为 bean，而不会意外注入。如果您需要访问底层的 RestTemplate，请使用 getRestTemplate（）方法。

###	Maven spring-boot.version

spring-boot.version属性已从spring-boot-dependencies pom 文件中删除，详见 [issue 5104](https://github.com/spring-projects/spring-boot/issues/5014)。

###	Integration Starter

spring-boot-starter-integration 通过删除一些 Spring Integration 中不常用的四个模块进行了简化，分别是：

*	spring-integration-file

*	spring-integration-http

*	spring-integration-ip

*	spring-integration-stream

如果你的应用依赖于这四个模块中的任何一个，应该添加一个明确的依赖关系到你的 pom.xml 或build.gradle。

此外，spring-integration-java-dsl 和 spring-integration-jmx 现在已经添加到 starter。推荐在应用中配置 Spring Integration 来使用 DSL。

###	Spring Batch Starter

spring-boot-starter-batch 不再依赖于内置数据库。如果你在使用，请声明一个数据库，例如：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-batch</artifactId>
</dependency>
<dependency>
	<groupId>org.hsqldb</groupId>
	<artifactId>hsqldb</artifactId>
	<scope>runtime</scope>
</dependency>
```

如果在配置 hsqldb 上有排除权限，那么可以删除该项。

###	Tomcat 降级

从 Tomcat 8.5.4 起，tomcat-juli模块现在被打包为内置 tomcat 一部分。 大多数用户不会注意到此更改，但是，如果您手动降级到旧版本的Tomcat，需要自己添加 tomcat-juli 模块。具体参考[文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#howto-use-tomcat-7)。

###	Dispatch Options Request

默认的 spring.mvc.dispatch-options-request属性已从 false 变更为 true，以与Spring Framework的首选默认值对齐。如果您不希望将OPTIONS请求分派到FrameworkServlet.doService，则应将 spring.mvc.dispatch-options-request 显式设置为false。

###	Forced character encoding

强制字符编码现在只适用于 request（而不是response）。如果要强制设置 request 和 response 的编码，将 spring.http.encoding.force 设置为 true。

###	Multipart support

multipart 属性配置的命名空间从 multipart. 变更到了  spring.http.multipart. 。

###	Server header

Server 的 Http response header 需要在 server.server-header 配置。

###	@ConfigurationProperties default bean names

当 @ConfigurationProperties bean 通过@EnableConfigurationProperties（SomeBean.class）注册时，我们用于产生一个 <prefix>.CONFIGURATION_PROPERTIES 的 bean。从 Spring Boot 1.4 开始，我们已经改变了该模式用来避免两个 bean 使用相同前缀的冲突。

新的命名规则是 <prefix>-<fqn>，其中 <prefix> 是 @ConfigurationProperties 注释中指定的前缀，<fqn> 是 bean 的完全限定名称。如果注释不提供任何前缀，则仅使用该 bean 的完全限定名称。

###	Jetty JNDI support

spring-boot-starter-jetty 不再包括 org.eclipse.jetty：jetty-jndi。如果你使用 Jetty 和 JNDI，需要直接添加此依赖项。

###	Guava caches

建议使用 Guava 缓存的人迁移到 [Caffeine](https://github.com/ben-manes/caffeine)。

###	Remote Shell

CRaSH 属性的命名空间从 shell 移动到 manage.shell。此外，现在应通过manage.shell.auth.type 来定义身份验证类型。

###	Spring Session auto-configuration improvements

Spring Boot 支持了更多 Spring Session 的后端存储：Redis，JDBC，MongoDB，Hazelcast和内存并发哈希映射。引入了一个新的 spring.session.store 类型的必需属性来配置 Spring Session 的存储实现。

###	Launch script identity

当启动脚本确定应用程序的默认身份时，现在会使用包含该jar目录的规范名称。这之前，如果包含jar的目录是软连接，则使用了软链接的名称。如果您需要更多的控制应用程序的身份，应该使用 APP_NAME 环境变量。

###	MongoDB 3

Mongo 的 Java 驱动程序的默认版本现在为3.2.2（之前是2.14.2），并且 spring-boot-starter-data-mongodb 使用了新的 mongodb-driver。

Embedded MongoDB 的自动配置也更新为使用 3.2.2 作为其默认版本。

###	Thymeleaf 3

默认情况下，Spring Boot 使用 Thymeleaf 2.1，但它现在与 Thymeleaf 3 兼容，请查看更新的[文档](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-use-thymeleaf-3)以获取更多详细信息。

###	Executable jar layout

可执行 jar 结构已经改变。如果使用 Spring Boot 的 Maven，Gradle 或 Ant 来构建应用程序，则此更改不会影响。如果自己构建归档文件，请注意，应用程序的依赖项现在包装在 BOOT-INF/lib 而不是 lib 中，应用程序自己的类现在包装在 BOOT-INF/classes 中，而不是 jar 的根目录。

####	Jersey classpath scanning limitations

对可执行 jar 结构的改变意味着 [Jersey](https://java.net/jira/browse/JERSEY-2085) 类路径扫描会影响可执行的jar文件以及可执行的war文件。要解决此问题，由 Jersey 扫描的类应该打包在一个jar中，并作为 BOOT-INF/lib 中的依赖项包含。Spring Boot 启动时应该对这些解压以便Jersey 可以扫描其内容，具体参考[文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#howto-extract-specific-libraries-when-an-executable-jar-runs)。

###	maven-failsafe-plugin 集成测试

从 Failsafe 2.19 起，classpath 不再包含 target/classes，而是使用项目的内置 jar。由于可执行 jar 结构的更改，该插件将无法找到你的类。 有两种方法可以解决这个问题：

1.	降级到 2.18.1 就可以使用 target/classes。
2. 	使用 spring-boot-maven-plugin 配置项 classifier 来指定目标。 这样，原始的 jar 将被插件使用，例如：

```xml
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<configuration>
		<classifier>exec</classifier>
	</configuration>
</plugin>
```

>
如果使用 Spring Boot 依赖管理，则无需执行任何操作，默认情况下将使用 2.18.1。
具体参考[文档](https://issues.apache.org/jira/browse/SUREFIRE-1198)。

###	HornetQ

HornetQ 的支持已被弃用。使用 HornetQ 的用户应考虑迁移到 Artemis。

####	@Transactional 默认使用 cglib 代理

当 Spring Boot 自动配置事务管理时，proxyTargetClass 现在设置为 true（意味着创建cglib代理，而不是要求你的 bean 实现一个接口）。如果希望将该行为与未自动配置的其他配置一样，则需要立即显式启用该属性：

```Java
@EnableCaching(proxyTargetClass = true)
```

>
如果你正好在接口上使用 @Transactional，那么你必须是明确的，并将@EnableTransactionManagement 添加到你的配置中。这将保持和前版本一致。

##	新特性
>
具体可参考 [changelog](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-1.4-Configuration-Changelog)。

###	Spring Framework 4.3

Spring Boot 1.4 基于 Spring Framework 4.3。Spring Framework 4.3 中有一些很好的改进，包括新的 Spring MVC @RequestMapping注释。 有关详细信息，请参阅[Spring Framework参考文档](http://docs.spring.io/spring-framework/docs/4.3.x/spring-framework-reference/htmlsingle/#new-in-4.3)。

请注意，Spring Framework 4.3 中的测试框架需要JUnit 4.12。有关详细信息，请参阅[SPR-13275](https://jira.spring.io/browse/SPR-13275)。

###	第三方类库升级

许多第三方类库已升级到最新版本。更新包括Jetty 9.3，Tomcat 8.5，Jersey 2.23，Hibernate 5.0，Jackson 2.7，Solr 5.5，Spring Data Hopper，Spring Session 1.2，Hazelcast 3.6，Artemis 1.3，Ehcache 3.1，Elasticsearch 2.3，Spring REST Docs 1.1，Spring AMQP 1.6 和 Spring Integration 4.3。

少数几个 Maven 插件也有升级。

###	支持 Couchbase

现在提供了 Couchbase 的全量配置。你可以通过添加 spring-boot-starter-data-couchbase 依赖并提供一些轻量配置，轻松访问 Bucket 和 Cluster bean：

```
spring.couchbase.bootstrap-hosts=my-host-1,192.168.1.123
spring.couchbase.bucket.name=my-bucket
spring.couchbase.bucket.password=secret
```

同时可以把 Couchbase 当做 Spring Data Repository 的备用存储或者是缓存来使用。详情参考[升级文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-couchbase)。

###	支持 Neo4J

现在为 Neo4J 提供自动配置支持。你可以连接到远程服务器或运行嵌入式Neo4J服务器。同时可以像 Spring Data Repository 一样使用它，例如：

```Java
public interface CityRepository extends GraphRepository<City> {

	Page<City> findAll(Pageable pageable);

	City findByNameAndCountry(String name, String country);

}
```
具体细节参考[Neo4J文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-neo4j)。

###	Redis Spring Data repositories

Redis 现在可以直接用于 Spring Data repositories。详情参考[Spring Data Redis 文档](http://docs.spring.io/spring-data/redis/docs/current/reference/html/#redis.repositories)。

###	支持 Narayana 事务管理

现在提供 Narayana 事务管理器自动配置的支持。如果你需要 JTA 支持，可以选择Narayana，Bitronix 或 Atomkos。 有关详细信息，请参阅更新的[参考文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-jta-narayana)。

###	支持 Caffeine cache

提供 Caffeine v2.2（Java 8重写 Guava 缓存支持）的自动配置。现在的 Guava 缓存用户应该考虑迁移到 Caffeine，因为 Guava 缓存支持将在将来的版本中被删除。

###	支持 Elasticsearch Jest

如果 Jest 在 classpath 中，SpringBoot 会自动配置一个 JestClient 和专用的 HealthIndicator。当 spring-data-elasticsearch 不在 classpath 中同样允许你使用 Elasticsearch。

###	启动失败的分析

Spring Boot 现在将对常见的启动失败进行分析，并提供有用的诊断信息，而不是简单地记录异常及其堆栈跟踪。例如，由于嵌入式 servlet 容器的端口正在使用，启动失败在早期版本的 Spring Boot 中看起来是这样：

```
2016-02-16 17:46:14.334 ERROR 24753 --- [           main] o.s.boot.SpringApplication               : Application startup failed

java.lang.RuntimeException: java.net.BindException: Address already in use
    at io.undertow.Undertow.start(Undertow.java:181) ~[undertow-core-1.3.14.Final.jar:1.3.14.Final]
    at org.springframework.boot.context.embedded.undertow.UndertowEmbeddedServletContainer.start(UndertowEmbeddedServletContainer.java:121) ~[spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.startEmbeddedServletContainer(EmbeddedWebApplicationContext.java:293) ~[spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.finishRefresh(EmbeddedWebApplicationContext.java:141) ~[spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:541) ~[spring-context-4.2.4.RELEASE.jar:4.2.4.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:118) ~[spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:766) [spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.boot.SpringApplication.createAndRefreshContext(SpringApplication.java:361) [spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:307) [spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1191) [spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1180) [spring-boot-1.3.2.RELEASE.jar:1.3.2.RELEASE]
    at sample.undertow.SampleUndertowApplication.main(SampleUndertowApplication.java:26) [classes/:na]
Caused by: java.net.BindException: Address already in use
    at sun.nio.ch.Net.bind0(Native Method) ~[na:1.8.0_60]
    at sun.nio.ch.Net.bind(Net.java:433) ~[na:1.8.0_60]
    at sun.nio.ch.Net.bind(Net.java:425) ~[na:1.8.0_60]
    at sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:223) ~[na:1.8.0_60]
    at sun.nio.ch.ServerSocketAdaptor.bind(ServerSocketAdaptor.java:74) ~[na:1.8.0_60]
    at org.xnio.nio.NioXnioWorker.createTcpConnectionServer(NioXnioWorker.java:190) ~[xnio-nio-3.3.4.Final.jar:3.3.4.Final]
    at org.xnio.XnioWorker.createStreamConnectionServer(XnioWorker.java:243) ~[xnio-api-3.3.4.Final.jar:3.3.4.Final]
    at io.undertow.Undertow.start(Undertow.java:137) ~[undertow-core-1.3.14.Final.jar:1.3.14.Final]
    ... 11 common frames omitted
```

在 1.4，会是这样：

```
2016-02-16 17:44:49.179 ERROR 24745 --- [           main] o.s.b.d.LoggingFailureAnalysisReporter   :

***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

如果仍然希望查看原因并追踪堆栈，请为org.springframework.boot.diagnostics.Logging FailureAnalysisReporter 启用调试日志记录。

###	图片 Banner

现在可以使用图像文件来渲染 ASCII 编码的 banner。将一个banner.gif，banner.jpg或 banner.png 文件拖放到 src/main/resources 中，使其自动转换为 ASCII。 您可以使用 banner.image.width 和 banner.image.height 属性来调整大小，或者使用banner.image.invert 来反转颜色。

###	RestTemplate builder

一个新的 RestTemplateBuilder 可以用来轻松创建一个具有合理默认值的RestTemplate。默认情况下，构建的 RestTemplate 将尝试使用类路径中可用的最合适的 ClientHttpRequestFactory，并且将知道要使用的 MessageConverter 实例（包括Jackson）。构建器包含一些可用于快速配置RestTemplate的有用方法。 例如，要添加BASIC auth支持，可以使用：

```Java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
	return builder.basicAuthorization("user", "secret").build();
}
```

自动配置的 TestRestTemplate 现在也使用了 RestTemplateBuilder。

###	JSON 组件

现在为自定义的 Jackson JsonSerializer 和 JsonDeserializer 注册提供了一个新的 @JsonComponent 注释。 这可以是解析JSON序列化逻辑的有用方法：

```Java
@JsonComponent
public class Example {

	public static class Serializer extends JsonSerializer<SomeObject> {
		// ...
	}

	public static class Deserializer extends JsonDeserializer<SomeObject> {
		// ...
	}

}
```

此外，Spring Boot 还提供了 JsonObjectSerializer 和 JsonObjectDeserializer 基类，它们在序列化对象时为标准的 Jackson 版本提供了有用的替代方法。详情参考[升级文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-json-components)。

###	error pages 约定

现在可以通过遵循基于约定的方法创建给定状态代码的自定义错误页面。在 /public/error中创建静态HTML文件，或者使用状态代码作为文件名将模板添加到 /templates/error。 例如，要注册一个自定义的404文件，您可以添加 src/main/resource/public/error/404.html。详见[升级文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-error-handling-custom-error-pages)。

###	统一的 @EntityScan

现在可以使用 org.springframework.boot.autoconfigure.domain.EntityScan 来指定要用于 JPA，Neo4J，MongoDB，Cassandra 和 Couchbase 的包。因此，JPA特定的org.springframework.boot.orm.jpa.EntityScan现在已被弃用。

###	ErrorPageRegistry

新的 ErrorPageRegistry 和 ErrorPageRegistrar 接口允许以一致的方式注册错误页面，而不管是否使用嵌入式 servlet 容器。ErrorPageFilter 类已更新为现在是 ErrorPageRegistry，而不是伪造的 ConfigurableEmbeddedServletContainer。

###	PrincipalExtractor

如果需要使用自定义逻辑提取 OAuth2 Principal，那么现在可以使用 PrincipalExtractor 接口。

###	Test

Spring Boot 1.4 提供了详细测试检查的支持。测试类和实用程序现在提供 spring-boot-test 和 spring-boot-test-autoconfigure（尽管大多数用户将继续使用 spring-boot-starter-test）。 我们已经将 AssertJ，JSONassert 和 JsonPath 依赖关系添加到测试启动器。

####	@SpringBootTest

使用 Spring Boot 1.3，有多种写入 Spring Boot 测试的方式。 您可以使用@SpringApplicationConfiguration，@ContextConfiguration与SpringApplicationContextLoader，@IntegrationTest或@WebIntegrationTest。 使用 Spring Boot 1.4，单个 @SpringBootTest 注释替代了以上这些。

将 @SpringBootTest 与 @RunWith（SpringRunner.class）结合使用，并根据要编写的测试类型设置 webEnvironment 属性。

一个典型的集成 mocked servlet 测试：

```Java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MyTest {

	// ...

}
```

一个运行在 server 上并监听特定端口的 web 集成测试：

```Java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvionment.DEFINED_PORT)
public class MyTest {

	// ...

}
```

一个运行在 server 上并监听随机端口的 web 集成测试：

```Java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvionment.RANDOM_PORT)
public class MyTest {

	@LocalServerPort
	private int actualPort;

	// ...

}
```

详见[升级文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-testing-spring-boot-applications)。

####	测试配置的自动探测

大多数测试现在可以自动检测测试配置。如果您遵循 Spring Boot 建议的约定来构造代码，则在未定义显式配置时，将加载 @SpringBootApplication 类。如果需要加载一个不同的 @Configuration 类，可以将其作为嵌套的内部类包含在测试中，或者使用 @SpringBootTest 的 classes 属性。

详见[文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-testing-spring-boot-applications-detecting-config)。

####	Mocking and spying beans

想要将 ApplicationContext 中的单个 bean 替换为用于测试目的的 mock 是很常见的。使用Spring Boot 1.4，现在可以通过 @MockBean 在测试中注解一个字段：

```Java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MyTest {

	@MockBean
	private RemoteService remoteService;

	@Autowired
	private Reverser reverser;

	@Test
	public void exampleTest() {
		// RemoteService has been injected into the reverser bean
		given(this.remoteService.someCall()).willReturn("mock");
		String reverse = reverser.reverseSomeCall();
		assertThat(reverse).isEqualTo("kcom");
	}

}
```

如果你想监视一个已经存在的 bean 而不是 mock，可以使用 @SpyBean：

```Java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MyTest {

	@MockBean
	private RemoteService remoteService;

	@Autowired
	private Reverser reverser;

	@Test
	public void exampleTest() {
		// RemoteService has been injected into the reverser bean
		given(this.remoteService.someCall()).willReturn("mock");
		String reverse = reverser.reverseSomeCall();
		assertThat(reverse).isEqualTo("kcom");
	}

}
```

详见[升级文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-testing-spring-boot-applications-mocking-beans)。

####	tests 自动配置

完整的应用程序自动配置有时会过度的测试，通常只想自动配置应用程序的特定“切片”。 Spring Boot 1.4 引入了一些专门的测试注释，可用于测试应用程序的特定部分：

*	@JsonTest - 测试 JSON marshalling and unmarshalling.

*	@WebMvcTest - 测试 用 MockMVC 注解了的 Spring MVC @Controllers

*	@RestClientTest - 测试 RestTemplate 调用

*	@DataJpaTest - 测试 Spring Data JPA

许多注释提供了特定于测试额外的自动配置。例如，如果使用 @WebMvcTest，还可以使用 @Autowire 来配置 MockMvc 实例。

详见[升级文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)。

####	JSON AssertJ

新的 JacksonTester，GsonTester 和 BasicJsonTester 类可以与 AssertJ 结合使用来测试 JSON 序列化和反序列化。测试人员可以使用 @JsonTest 注释或直接在测试类上使用：

```Java
@RunWith(SpringRunner.class)
@JsonTest
public class MyJsonTests {

	private JacksonTester<VehicleDetails> json;

	@Test
	public void testSerialize() throws Exception {
		VehicleDetails details = new VehicleDetails("Honda", "Civic");
		assertThat(this.json.write(details)).isEqualToJson("expected.json");
		assertThat(this.json.write(details)).hasJsonPathStringValue("@.make");
	}

}
```

有关详情，参考[文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-json-tests)。

####	@RestClientTest

如果要测试 REST 客户端，可以使用 @RestClientTest 注释。默认情况下，它将自动配置Jackson 和 GSON，配置 RestTemplateBuilder 并添加对 MockRestServiceServer 的支持。

####	Spring TEST Docs 自动配置

结合上述对 MockMvc 自动配置的支持，Spring REST 文档的自动配置已经被引入。可以使用新的 @AutoConfigureRestDocs 注释启用 REST 文档。这将导致 MockMvc 实例被自动配置为使用 REST 文档，并且还消除了使用 REST 文档的JUnit规则的需要。有关详细信息，请参阅参考[文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-rest-docs)的相关部分。

####	测试应用

spring-boot-starter-test 现在提供 [AssertJ](http://joel-costigliola.github.io/assertj/) 支持。

org.springframework.boot.test 包中的测试工具已被移动到 spring-boot-test 中。

###	Actuator 改进

可以使用 InfoContributor 接口来注册需要被 /info 暴露的 bean。以下支持开箱即用：

*	通过 Maven git-commit-id-plugin 插件或 Gradle gradle-git-properties 插件展示 Git 信息。（配置 management.info.git.mode=full 暴露全部细节）

*	通过 Maven 或 Gradle 插件生成的构建信息。


详情参考[文档](http://docs.spring.io/spring-boot/docs/1.4.x/reference/htmlsingle/#production-ready-application-info)。

###	MetricsFilter 改进

MetricsFilter 现在可以以经典的方式提交，也可以按 HTTP 方法分组提交。通过 endpoints.metrics.filter 属性配置：

```
endpoints.metrics.filter.gauge-submissions=per-http-method
endpoints.metrics.filter.counter-submissions=per-http-method,merged
```

###	Spring Session JDBC Initializer

如果将 Spring Session 配置为使用JDBC存储，在启动时就会自动创建。

###	Artemis/HornetQ 安全连接

Spring Boot 现在允许连接一个安全的 Artemis / HornetQ 代理。

###	Annotation processing

Apache HttpCore 4.4.5 [删除了一些注解](https://github.com/apache/httpcore/commit/9e065bad07c9ca771c42e5b4f1dc12118c5e75c9)。如果你使用了注释并对其中一个已删除注释的类进行子类化，这会是一个不兼容的变更。例如，如果使用 @Immutable，会发现在编译时处理失败错误 [ERROR] diagnostic: error: cannot access org.apache.http.annotation.Immutable。

要避免这个问题，可以通过降级为  HttpCore 4.4.4，或者硬编码解决，以便有问题的类不受编译时注释的影响。

###	Miscellaneous

*	新增 server.jetty.acceptors 和 server.jetty.selectors  配置来指定 Jetty acceptors 和 selectors。

* 	server.max-http-header-size 和 server.max-http-post-size 可用来限制 HTTP headers 和 HTTP POST 最大值。在 Tomcat、Jetty 和 Undertow 上均生效。

*	Tomcat 线程池最小备用线程数可以通过 server.tomcat.min-spare-threads 配置。

*	application.yml 支持注释 profile。通过常见的 ! 前缀来设置 spring.profiles 的值。

*	actuator 提供了一个新的 headdump，返回一个经 GZip 压缩的 hprof 堆转储文件。

*	Spring Mobile 会在为所有模板引擎提供自动配置。

*	Spring Boot 的 Maven 插件允许使用新的 includeSystemScope 属性来绑定 system 作用域。

*	当一个异常被 HandlerExceptionResolver  捕获时，通过 spring.mvc.log-resolved-exception 配置可以自动记录警告日志。

*	在启动时可通过 spring.data.cassandra.schema-action 配置自定义的 schema 行为。

*	Spring Boot 臃肿的 jar 消耗更少内存。

*	语言字符集映射现在可通过 spring.http.encoding.mapping.<locale>=<charset> 配置。

*	默认情况下，使用 spring.mvc.locale 配置的区域设置现在被请求的 Accept-Language 头部覆盖。 要恢复 1.3 中忽略 header 特性，请将 spring.mvc.locale-resolver 设置为 fixed。

###	Spring Boot 1.4 中废弃的

*	自 Spring Framework 4.3 起，废弃了 Velocity 的支持。

*	废弃了 UndertowEmbeddedServletContainer 部分构造方法（大部分不受影响）。

*	@ConfigurationProperties 注解中的 locations 和 merge 被 Environment 替代。

*	不应再使用 protect 级别的 SpringApplication.printBanner 方法打印自定义 banner 。应改用 Banner 接口。

*	protect 级别的 InfoEndpoint.getAdditionalInfo 方法已被弃用，可使用 InfoContributor 接口替代。

*	org.springframework.boot.autoconfigure.test.ImportAutoConfiguration 移动到 org.springframework.boot.autoconfigure.

*	org.springframework.boot.test 包下的所有类都被弃用。可查阅 upgrading 来了解详情。

*	PropertiesConfigurationFactory.setProperties(Properties) 已被 PropertySources 替代。

*	org.springframework.boot.context.embedded 下的少数类被移动到 org.springframework.boot.web.servlet. 下。

*	org.springframework.boot.context.web 下所有类都被废弃或迁移。

*	spring-boot-starter-ws Starter 重命名为 spring-boot-starter-web-services。

*	spring-boot-starter-redis Starter 重命名为 spring-boot-starter-data-redis。

*	spring-boot-starter-hornetq Starter 和配置被 spring-boot-starter-artemis 取代。

*	management.security.role 被 management.security.roles 取代。

*	@org.springframework.boot.orm.jpa.EntityScan 注释已被弃用，可使用@ org.springframework.boot.autoconfigure.domain.EntityScan替代或显示配置。 

*	TomcatEmbeddedServletContainerFactory.getValves() 被 getContextValves() 取代。

*	org.springframework.boot.actuate.system.EmbeddedServerPortFileWriter 被 org.springframework.boot.system.EmbeddedServerPortFileWriter 替代。

*	org.springframework.boot.actuate.system.ApplicationPidFileWriter 被 org.springframework.boot.system.ApplicationPidFileWriter 替代。

###	配置重命名

*	spring.jackson.serialization-inclusion --> spring.jackson.default-property-inclusion.

*	spring.activemq.pooled --> spring.activemq.pool.enabled.

*	spring.jpa.hibernate.naming-strategy --> spring.jpa.hibernate.naming.strategy.

*	server.tomcat.max-http-header-size --> server.max-http-header-size.

