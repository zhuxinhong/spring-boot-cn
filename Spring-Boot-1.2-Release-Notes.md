#	Spring Boot 1.2 Release Notes

##	从Spinrg Boot 1.1升级

###	Servlet 3.1, Tomcat 8 and Jetty 9

Spring Boot 现在使用Tomcat8和Jetty9作为内置servlet容器。同时提供了Servlet3.1和开箱即用的增强WebSocket。如果你坚持喜欢旧版本的话，仍然可以使用Tomcat7或者Jetty8。

###	日志输出

升级到Spring Boot1.2以后，默认日志配置不再写到日志文件。如果你想输出文件，可以使用 logging.path 或者 logging.file 配置。你也可以完全自定义添加你自己的logback.xml。

###	HTTP解码

现在会自动注册一个CharacterEncodingFilter来进行Http URI和body的解码。你可以使用 spring.http.encoding.charset 配置如果你需要utf-8以外的编码，或者可以设置 spring.http.encoding.enabled 属性为false来关闭CharacterEncodingFilter的注册。

###	Spring MVC中Redirect忽略Model属性

Spring MVC的初始配置中RequestMappingHandlerAdapter内的ignoreDefaultModelOnRedirect属性为true。如果你需要在redirect时带上model的属性需要在 application.properties 中配置：

```
spring.mvc.ignore-default-model-on-redirect=false
```

###	Jackson 默认配置

Jackson初始配置中 ObjectMapperMapperFeature.DEFAULT_VIEW_INCLUSION 和 DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES 被禁用。如果你需要恢复到之前的配置，需要在 application.properties 中配置：

```
spring.jackson.mapper.default-view-inclusion=true
spring.jackson.deserialization.fail-on-unknown-properties=true
```

###	Mongo 和 MongoDbFactory

如果你定义了你自己的 MongoDbFactory bean，MongoAutoConfiguration将不会在起作用。要注册 Mongo bean，请确人你声明了自己的 MongoDbFactory 或者使用 MongoDbFactory 接口去连接 Mongo。

###	变更 health.* properties 为 management.health.*

为了和其他管理相关配置保持一致，健康检查相关配置从 health 变为 management.health。

###	重命名 VanillaHealthIndicator

VanillaHealthIndicator 被重命名为 ApplicationHealthIndicator 。大部分用户不会受此影响，除非在代码里 你导入了 org.springframework.boot.actuate.health.VanillaHealthIndicator ，需要变更为 org.springframework.boot.actuate.health.ApplicationHealthIndicator 。

###	Hibernate

SpringNamingStrategy 被移动到 org.springframework.boot.orm.jpa.hibernate 包下，org.springframework.boot.orm.jpa.SpringNamingStrategy 仍然保留，但它已经过时并在未来的版本中被去除。

现在提供了 hibernate-envers, hibernate-jpamodelgen 和 hibernate-ehcache 的依赖管理。

#####	PersistenceExceptionTranslationPostProcessor

默认注册了 PersistenceExceptionTranslationPostProcessor。如果你不想处理异常，可以设置 spring.dao.exceptiontranslation.enabled 为 false。

###	Health JSON

通过 /health 路由访问的JSON被稍稍改动，现在只有一个 HealthIndicator 参与其中。如果你以前查询一个特定的JSON，需要升级下你的监控工具。

###	健康检查路由匿名访问限制

现在 /health 路由已经限制匿名访问。如果有匿名访问，只会显示当前服务是起着还是挂了。它同时能缓存指定期间的响应信息，通过 endpoints.health.time-to-live 来配置。这种限制可以被禁用，像1.1x版本一样，通过设置 endpoints.health.sensitive 为false即可。

###	Spring4.1

Spring Boot 1.2 需要Spring Framework4.1.5或更新的版本，不兼容Spring Framework4.0。

###	Hikari CP

Boot中使用 com.zaxxer:HikariCP 时，需要依赖Java8。如果你想在Java6或Java7使用，需要更新你的依赖为com.zaxxer:HikariCP-java6。

###	配置

spring.data.mongo.repositories.enabled 重命名为 spring.data.mongodb.repositories.enabled 。

###	过时

*	ApplicationPidFileWriter 替代了 org.springframework.boot.actuate.system.ApplicationPidListener。
* 	spring-rabbit工程中， @EnableRabbit 替代了 @EnableRabbitMessaging。
*	spring.jackson.serialization. 配置替代了 http.mappers. 。
*	BasicJsonParser 替代 org.springframework.boot.json.SimpleJsonParser，以此避免“Json Simple”jar的冲突。

##	新特性

###	版本升级

Spring Boot1.2需要Spring Framework4.1，Jackson、Joda Time、Hibernate Validator等第三方依赖已经升级到新的release版本。内置了Tomcat8和Jetty9作为默认servlet容器，并提供servlet3.1支持。

###	@SpringBootApplication

增加了一个新的注解 @SpringBootApplication，等价于@Configuration + @EnableAutoConfiguration + @ComponentScan。如果发现频繁的使用了上述3个注解，你应该考虑换掉它们。

###	JTA

Spring Boot 1.2现在支持Atomkos、Bitronix分布式事务。JTA事务同时支持Java EE合适的应用服务器。

当处于JTA环境中时，Spring的 JtaTransactionManager 会接管事务。自动配置的JMS、DataSource和JPA的beans将被升级到支持XA事务。你可以使用标准的Spring方言，例如 @Transactional 参与到一个分布式事务中。

此外，对Atomkos和Bitronix的配置会更加简单，即使你不使用@EnableAutoConfiguration。详情请见[JTA section](http://docs.spring.io/spring-boot/docs/1.2.x/reference/htmlsingle/#boot-features-jta)。

###	JNDI

如果你在使用完整的JavaEE应用服务，你现在可以从JNDI找到DataSource和JMS ConnectionFactory的beans。在application.properties或application.yml中使用 spring.datasource.jndi-name 和 spring.jms.jndi-name 等配置即可。

###	自定义 Jackson

现在可以通过 spring.jackson 配置来自定义Jackson的ObjectMapper。Jackson的SerializationFeature, DeserializationFeature, MapperFeature, JsonParser 和JsonGenerator 现在可以通过 erialization, deserialization, mapper, parser 和 generator等配置来自定义。例如要开启DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES，在application.properties中配置：

```
spring.jackson.deserialization.fail-on-unknown-properties=true
```

此外，还增加了 spring.jackson.date-format 和 spring.jackson.property-naming-strategy 的配置。

###	Banner 接口

增加了一个新的Banner接口来和SpringApplication.setBanner(…​)一起使用，以输出自定义banner。对于简单的banner，仍然推荐使用src/main/resources/banner.txt。但如果你有什么奇怪的想法，新的接口将会很给力。

###	Banner 属性配置

可以在 src/main/resources/banner.txt 中使用 ${application.version}, ${spring-boot.version}, ${application.formatted-version} 和 ${spring-boot.formatted-version} 等变量来输出SpringBoot版本信息。formatted-version 配置显示在括号中，并以v作为前缀，例如 （1.2.0.RELEASE)。

###	JMS

在 Spring Framework 4.1中，提供了 @JmsListerner 的自动配置注解。如果在classpath下有spring-jms.jar，@EnableJms注解仍能自动配置。

###	AMQP

同样，@RabbitListener 也能自动配置 Spring AMQP1.4。当使用spring-rabbit.jar时，仍可使用@EnableRabbit注解。自动配置的Rabbit同时扩展并注册了RabbitMessagingTemplate。

###	Spring Cloud 连接器

新增了 spring-cloud-connectors 和 spring-boot-starter-cloud-connectors 的POM依赖。增加了 @CloudScan 的自动配置注解。

###	Email

新增了 spring-boot-starter-mail 的POM依赖。你可以在你的服务中注入一个 JavaMailSender 的Bean来发邮件。使用spring.mail.* 来配置你的邮箱信息。

###	内置Servlet容器 Undertow

除了Tomcat和Jetty，Spring Boot现在提供了Undertow作为内置Servlet容器。切换方法请查看相关文档获取。

###	升级 CLI

#### 创建新工程

spring的命令行工具新增了一个 init 选项来从start.spring.io创建一个新工程。例如，创建一个新的web应用，输入：

```
$ spring init -d=web myapp.zip
```

####	CLI 扩展

现在允许CLI客户端自行安装和卸载CLI扩展。spring install <maven coordinates> 会从远端下载jar并安装到CLI。spring uninstall 可以移除之前安装的扩展。

####	其他CLI变化

CLI现在能检测并支持Spring的@Cacheable注解。

###	Jetty、Tomcat使用SSL

[SSL](http://docs.spring.io/spring-boot/docs/1.2.x/reference/htmlsingle/#howto-configure-ssl)现在可以通过 server.ssl.* 来显示声明配置。同时支持Tomcat和Jetty，具体细节请查看相关文档。

###	端点执行器

新增了全局配置 endpoints.enabled 来控制端点是否需要被开启。这允许你从当前输出模型转换到输入模型。例如，除了健康检查禁用其他全部端点可在application.properties中如下配置：

```
endpoints.enabled=false
endpoints.health.enabled=true
```

###	指标

####	系统

系统指标包含堆、线程、类和GC信息。

####	数据源

数据源连接池指标通过 /metrics 路由获取。可获取Tomcat、Hikari和DBCP等连接池的活跃连接数和使用详情。

####	Tomcat session

如果使用Tomcat作为内置Servlet容器，可以获得当前活跃的最大支持的session数。

####	Dropwizard

通过 /metrics 能获取到Dropwizard的MetricRegistry。标准和计数器作为单一值体现。Timers、Meters和Histograms等指标都以数字展现。

###	健康检查

####	JSON格式化

/health 现在返回同HealthIndicators一致的JSON数据。这使单独查询某一个JSON更简单。

#### 数据源

DataSourceHealthIndicator 现在通过 spring.datasource.validation-query 配置来检查数据源健康状况。

####	磁盘空间

通过 /health 可以获得剩余磁盘空间，当它低于一个阈值(默认10Mb)会触发 Down 状态。可以通过 health.diskspace.path 和 health.diskspace.threshold 来自定义配置。

### Conditions

@ConditionalOnProperty增加了 havingValue 和 matchIfMissing 属性。现在您可以使用条件来创建更复杂的属性匹配条件。为了和@Contidions一起使用，增加了一个新类 AnyNestedCondition 。最终，@ConditionalOnBean 可被用于声明类就像声明字符串一样。

###	Gson

可使用Gson来代替Jackson来创建Json输出。Jackson仍然是默认被推荐的。如果你使用Gson，需要排除Jackson的依赖，除非你使用Spring Boot某些特定依赖Jackson的执行器。在Spring Boot 1.2.2中，当Gson和Jackson都存在时，你可以通过配置 spring.http.converters.preferred-json-mapper 值为 json 来管理你的应用。

###	EmbeddedServerPortWriter

spring-actuator 工程中新增了类 org.springframework.boot.actuate.system.EmbeddedServerPortFileWriter ，可以在应用启动时输出内置服务器的端口。


###	Log4j2

提供Log4J来替代系统日志，提供POM依赖 spring-boot-starter-log4j2 。Logback仍然是被推荐默认使用的。

###	Jersey

支持自动配置Jersey。详情请见相关文档。

###	Apache-Commons DBCP2

提供commons-dbcp2数据库连接池。不能使用于Tomcat、Hikari和DBCP(v1)。

###	Maven 插件

spring-boot-maven-plugin 中 repackage 现在可以被禁用。当你使用 spring-boot:run 但不需要庞大的Jar包时会很有用。

###	meta-data 配置

为了给工具开发者在 application.properties 中提供 'code completion'支持，spring-boot, spring-boot-autoconfigure 和 spring-boot-actuator 现在包含了额外的 meta-data 文件。同时允许你生成自己的文件，只要使用@ConfigurationProperties来标记执行器即可。

###	其他

Spring Boot 1.2中其他特性：

*	RedisProperties 新增 database 字段。

*	RelaxedDataBinder 支持 alais 属性。

*	正则表达式可被用于所有 keystosanitize 配置文件中。

*	可通过 spring.output.ansi.enabled 配置来使用 AnsiOutput。

*	可以在 /public, /static, /resources 等目录下放置 favicon.ico文件。

*	可使用spring.pidfile环境变量，来指定ApplicationPidFileWriter输出的PID文件。	
*	Tomcat数据源信息通过JMX自动暴露。

*	使用了@Configuration的SpringBootServletInitializer的子类不需要在重写 configure 方法来注册资源。

*	如果你这么固执，可以使用xml作为配置格式。



