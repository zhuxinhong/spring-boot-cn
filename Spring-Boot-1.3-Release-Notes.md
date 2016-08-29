#	Spring Boot 1.3 Release Notes

##	从Spinrg Boot 1.2升级

###	Spring Boot 1.2 中过时的

当前版本移除了在1.2中过时的类、方法和属性配置。升级前请确保你没有调用过时的方法。

###	Jackson

在1.2的application context中，针对每一个ObjectMapper都注册了Jackson Module。这使得无法控制全局的ObjectMapper。SpringBoot1.3只会对创建或者配置了Jackson2ObjectMapperBuilder的ObjectMappers注册Jackson Module。这使得模块配置的行为符合Jackson的其他配置。

###	Logging

#####	声明Spring配置

为了防止Spring日志配置文件被初始化2次。强烈推荐(并不是必须的)你用 -spring 后缀新命名日志配置文件。例如 logback.xml 修改成 logback-spring.xml。

#####	初始化失败

在1.2中，如果你配置了 logging.config 自定义日志文件并且该文件不存在时，它会使用默认配置。Spring Boot 1.3会因为文件不存在而失败。相同的，如果你错误的配置了日志，1.2会使用默认日志配置。1.3会失败并通过 System.err 的配置来报错。

###	Spring HATEOAS

移除了 spring.hateoas.apply-to-primary-object-mapper 配置，这使得  Spring HATEOAS 相关配置不再影响主要的 ObjectMapper 。为了处理 application/hal+json 的请求，一个新的配置 spring.hateoas.use-hal-as-default-json-media-type 控制了  Spring HATEOAS HTTP 的消息转换器是否会处理 application/json 的请求。

###	/health 安全

控制 /health 暴露哪些信息的安全配置被微微调整了。详情可查看文档[HTTP health endpoint access restrictions](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready-health-access-restrictions)。

###	Http response 压缩

Spring Boot 1.2为Tomcat、使用了GZipFilter的Jetty、Undertow 提供了原生的response压缩。然而Jetty团队并不赞同他们的gzip filter，受此影响，Spring Boot 1.3 替换了所有内置容器的response压缩方案。server.tomcat.compression. 和 spring.http.gzip. 已不能再使用。可使用新的 server.compression.* 配置。

###	Tomcat session 存储

Tomcat默认已不再保存session数据到/tmp文件夹。如果你想使用Tomcat的session持久化，需设置 server.session.persistent 为 true。server.session.store-dir 可以指定保存文件的路径。


###	Jetty JSP

spring-boot-starter-jetty 工程 Starter POM 文件不再包含 org.eclipse.jetty:jetty-jsp 。如果你要使用Jetty的JSP功能，需要自行添加依赖。

###	MVC堆栈跟踪

Spring MVC 渲染错误响应时候不再包括堆栈信息。如果想要Spring Boot 1.2中的效果，请设置 error.include-stacktrace 为 on-trace-param 。

###	Thymeleaf’s 整合 Spring Security

鉴于升级到 Spring Security 4, Spring Boot 1.3 同时升级了 Thymeleaf’s Spring Security 的支持。相关组件是 org.thymeleaf.extras:thymeleaf-extras-springsecurity4 。请更新 pom.xml 或 build.gradle 。

###	Groovy 模板

GroovyTemplateProperties 继承 AbstractTemplateViewResolverProperties 并提供配置选项。如果你通过 prefix.spring.groovy.template.prefix 配置了自定义的资源，需要重命名为 prefix.spring.groovy.resource-loader-location 。

###	配置顺序

Spring Boot 不再使用 @Order 来管理 自动配置类的顺序，请使用 @AutoconfigureOrder 。同时也可以使用 @AutoconfigureBefore 和 @AutoconfigureAfter 来声明自动配置的类。

###	Gradle 插件

##### bootrun

使用 bootrun 时，Spring Boot 中Gradle插件不再加载 src/main/resources 到classpath。如果你想要保持之前功能的话，建议使用Devtools。Spring Boot 1.2 中使用 gradle 构建的话，可以使用 addResources 属性。

#####	依赖管理

Spring Boot gradle[插件已升级](https://github.com/spring-gradle-plugins/dependency-management-plugin)。
大多数用户都不受此影响，但使用了 versionManagement 配置的需要更新他们的构建脚本。

相较依赖一个配置文件来管理版本，新的插件允许你使用 Maven bom。例如：

```
dependencyManagement {
    imports {
        mavenBom 'com.example:example-bom:1.0'
    }
}
```

#####	应用插件

Spring Boot Gradle 插件默认不再使用Gradle的[应用插件](https://docs.gradle.org/current/userguide/application_plugin.html)。如果你希望使用应用插件，你需要在 build.gradle 中声明它。

如果不需要应用插件提供的功能，但是使用了它的 mainClassName 或 applicationDefaultJvmArgs 配置，你需要稍稍的更新你的 build.gradle 。

在springBoot配置中， main 类需要被 mainClass属性来指定。例如：

```
springBoot {
    mainClass = 'com.example.YourApplication'
}
```

applicationDefaultJvmArgs 需要在工程中的 ext 块中配置，例如：

```
ext {
    applicationDefaultJvmArgs = [ '-Dcom.example.property=true' ]
}
```

使用应用插件的run任务时，如果通过main属性配置了工程的main类，需要把它移动到 bootRun 任务下，例如：

```
bootRun {
    main = com.example.YourApplication
}
```

###	Maven插件

如果使用了 spring-boot-starter-parent，Maven 属性只能使用 @ 来标记。这阻止了Spring配置文件中的占位符在构建中被扩大。

具体的说，如果你使用表示格式（例如，${project.version}），请更改至 （@project.version@)来覆盖 maven-resources-plugin 的配置。

###	CLI依赖管理

Spring Boot 1.3 现在支持在metadata中使用 Maven boms 来管理依赖。@DependencyManagementBom 需要和 @GrabMetadata 一起使用来提供bom，例如：
@DependencyManagementBom("io.spring.platform:platform-bom:1.1.2.RELEASE")。

###	属性重命名

为了命名一致，application.properties 中下列配置key被修改：

*	spring.view. ---> spring.mvc.view.

*	spring.pidfile ---> spring.pid.file

*	error.path ---> server.error.path

*	server.session-timeout ---> server.session.timeout

*	servet.tomcat.accessLogEnabled ---> server.tomcat.accesslog.enabled

*	servet.tomcat.accessLogPattern ---> server.tomcat.accesslog.pattern

*	servet.undertow.accessLogDir ---> server.undertow.accesslog.dir

*	servet.undertow.accessLogEnabled ---> server.undertow.accesslog.enabled

*	servet.undertow.accessLogPattern ---> server.undertow.accesslog.pattern

*	spring.oauth2. ---> security.oauth2.

*	server.tomcat.compression and spring.http.gzip ---> server.compression.*

*	prefix.spring.groovy.template.prefix ---> prefix.spring.groovy.resource-loader-location


