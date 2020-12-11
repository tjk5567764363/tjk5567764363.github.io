# Spring Boot 2.4 更新日志

## 从Spring Boot 2.3升级

### 版本控制方案更改

从2.4开始，Spring Boot采用[新版Spring版本控制方案](https://spring.io/blog/2020/04/30/updates-to-spring-versions) - 这意味着你需要把你的`build.gradle`/`pom.xml`中的Spring Boot版本从`2.3.5.RELEASE`更新为`2.4.0`

### JUnit 5 的 Vintage 引擎从 `spring-boot-starter-test`中已去除

如果你升级到Spring Boot 2.4，看到一些JUnit类的测试编译错误，例如`org.junit.Test`，这可能是因为JUnit 5的vintage引擎已经从`spring-boot-starter-test`移除了。vintage引擎允许使用 JUit 4编写的测试代码运行在 JUit 5上。如果你不想迁移你的测试代码到 JUit 5且希望继续使用 JUit 4的话，添加 Vintage 引擎的依赖，如下图的 Maven 示例所示：

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

如果你使用的是 Gradle，如下所示了相同的配置

```groovy
testImplementation("org.junit.vintage:junit-vintage-engine") {
    exclude group: "org.hamcrest", module: "hamcrest-core"
}
```

### 配置文件处理（application.properties 和 application.yml 文件）

Spring Boot 2.4 更改了处理`application.properties`和 `application.yml`的方式，如果你只有简单的一个`application.properties`或`application.yml`文件，你可以无缝升级。但是如果，你的设置更为复杂（有profile分类properties，或profile active的properties），如果你想使用新的特性的话，你可能需要进行[一些修改](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-Config-Data-Migration-Guide)。

如果你只是想兼容Spring Boot 2.3的逻辑，你可以把`application.properties`和`application.yml`文件中的 `spring.config.use-legacy-processing`配置项设置为`true`

### 配置数据导入

配置位置指定取决于`spring.config.location`和`spring.config.import`（该次版本引入），如果文件或文件夹不存在，将不再默默的失败。如果你想从一个位置导入配置

如果你想从一个位置导入，就算没有找到，你希望它跳过，你现在可以加上optiona前缀

举个例子`spring.config.location=optional:/etc/config/application.properties` 将会从`/etc/config/` 导入`application.properties`，如果不存在，也会跳过

如果你想要把所有位置处理成可选的，你可以在`SpringApplication.setDefaultProperties(… )`设置`spring.config.on-not-found=ignore`，或者当作系统/环境变量设置

### 内置数据库侦测

内置数据库逻辑优化，考虑到内置数据库只是内存级别的。如果你使用的是基于文件持久化或服务器模式的H2，HSQL和Derby，这次修改有2个比较重要的地方

1. sa的用户名不再设置了，如果你依赖这个设置，你需要在你的配置里设置`spring.datasource.username=sa`
2. 这些数据库不在系统启动的时候初始化，他们不再被考虑到内置，你可以用`spring.datasource.initialization-mode`调整一些通常的配置

### logback配置参数

特定于Logback的日志记录属性已重命名以反映它们是特定于Logback的事实。以前的名字已被废弃。

以下Spring Boot属性已更改：

- `logging.pattern.rolling-file-name` → `logging.logback.rollingpolicy.file-name-pattern`
- `logging.file.clean-history-on-start` → `logging.logback.rollingpolicy.clean-history-on-start`
- `logging.file.max-size` → `logging.logback.rollingpolicy.max-file-size`
- `logging.file.total-size-cap` → `logging.logback.rollingpolicy.total-size-cap`
- `logging.file.max-history` → `logging.logback.rollingpolicy.max-history`

以及它们映射到的系统环境属性：

- `ROLLING_FILE_NAME_PATTERN` → `LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN`
- `LOG_FILE_CLEAN_HISTORY_ON_START` → `LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START`
- `LOG_FILE_MAX_SIZE` → `LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE`
- `LOG_FILE_TOTAL_SIZE_CAP` → `LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP`
- `LOG_FILE_MAX_HISTORY` → `LOGBACK_ROLLINGPOLICY_MAX_HISTORY`

### 默认servlet注册器

spring boot 2.4不再注册DefaultServlet进你的servlet容器