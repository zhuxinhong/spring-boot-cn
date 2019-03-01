#	Spring Boot 2.0 Release Notes


在 Spring Boot 2.0 中，非常多配置被重命名、废弃以至于开发者需要更新 application.properties/application.yml 。为了帮你实现这一点，Spring Boot 提供了一个新的模块 spring-boot-properties-migrator。一旦添加为项目依赖，不仅可以在启动时分析应用的环境、输出诊断信息，还可以在运行时迁移临时属性。在项目迁移过程中必须具备这个模块

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-properties-migrator</artifactId>
	<scope>runtime</scope>
</dependency>
```

```
runtime("org.springframework.boot:spring-boot-properties-migrator")
```

>完成迁移后，请确保将此模块从项目中删除。

如果你想了解细节，这里有个列表，否则，请继续下面的部分：

*	[Spring Boot 2.0.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0.0-Release-Notes)
*	[Running Spring Boot on Java 9](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-with-Java-9)
*	[Upgrading to Spring Framework 5.0](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x#upgrading-to-version-50)

##	构建 Spring Boot 应用
###	Spring Boot Maven 插件
作为公开的插件配置现在都以 spring-boot 前缀开始，以保持一致性并避免与其他插件冲突。

例如，以下命令使用命令行启用 prod 配置文件:

```
mvn spring-boot:run -Dspring-boot.run.profiles=prod
```

####	Surefire
自定义包含/排除模式已与最新的Surefire默认设置对齐。如果依赖我们的插件，请相应地更新插件配置。过去是这样的:

```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-surefire-plugin</artifactId>
	<configuration>
		<includes>
			 <include>**/*Tests.java</include>
			 <include>**/*Test.java</include>
		</includes>
		<excludes>
			<exclude>**/Abstract*.java</exclude>
		</excludes>
	</configuration>
</plugin>

```

>如果你在使用 JUnit5，需要升级 Surefire 到 [2.22.0](http://maven.apache.org/surefire/maven-surefire-plugin/examples/junit-platform.html)。

###	Spring Boot Gradle 插件
Spring Boot Gradle 插件进行了大量重写，提供不少新的改进特性。你可以在插件的[文档](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/gradle-plugin/api/)中阅读更多关于插件功能的信息。

####	依赖管理
Spring Boot Gradle 插件不在自用应用依赖管理插件。相反，Spring Boot 通过引入正确的 Spring - Boot -dependencies BOM 来使用依赖管理插件。这使你能够更好地控制依赖项管理以及何时配置依赖项管理。

对大多数应用来说，应用依赖管理插件就足够了：

```
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management' // <-- build.gradle加上这行
```

>依赖项管理插件仍然是 spring-boot-gradl -plugin 的传递依赖项，所以不需要在构建脚本配置中将它作为类路径依赖项列出。