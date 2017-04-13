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
