# Configuring app for Native build

## settings.gradle

Add spring releases repository instead of mileston and add aot plugin:

```
pluginManagement {
    repositories {
        //maven { url 'https://repo.spring.io/milestone' }
        maven { url 'https://repo.spring.io/release' }
        gradlePluginPortal()
    }
    plugins {
        ...
        id 'org.springframework.experimental.aot' version '0.11.1'
    }
}
```

## gradle.properties

Upgrade the following two librarires:

```properites
springBootVersion=2.6.1
hibernateVersion=5.6.1.Final
```

## build.gragle

1. Register aot plugin and change spring `https://repo.spring.io/milestone` with `https://repo.spring.io/release`

2. Add Hibernate BytecodeEnhancement plugin (NOT SURE we need this?!)

```groovy
buildscript {
    repositories {
        gradlePluginPortal()
        maven { url 'https://repo.spring.io/release' }
    }
    dependencies {
        //jhipster-needle-gradle-buildscript-dependency - JHipster will add additional gradle build script plugins here
        classpath "org.hibernate:hibernate-gradle-plugin:5.6.3.Final"
    }
}

plugins {
    id "java"
    id "maven-publish"
    id "idea"
    id "eclipse"
    id "jacoco"
    id "org.springframework.boot"
    id "com.google.cloud.tools.jib"
    id "com.gorylenko.gradle-git-properties"
    id "org.liquibase.gradle"
    id "org.sonarqube"
    id "io.spring.nohttp"
    id "com.github.andygoossens.gradle-modernizer-plugin"
    //jhipster-needle-gradle-plugins - JHipster will add additional gradle plugins here
    id "org.springframework.experimental.aot"
}
apply plugin: 'org.hibernate.orm'
```

2. Configure buildpacks

```groovy
bootBuildImage {
    builder = "paketobuildpacks/builder:tiny"
    environment = [
        "BP_NATIVE_IMAGE" : "true"
    ]
}
```

3. Disable devtools and enable spring-boot-starter-tomcat

```groovy
configurations {
    providedRuntime
    //implementation.exclude module: "spring-boot-starter-tomcat"
    all {
        exclude group:"org.springframework.boot", module: "spring-boot-devtools"
        resolutionStrategy {
            // Inherited version from Spring Boot can't be used because of regressions:
            // To be removed as soon as spring-boot use the same version
            force 'org.liquibase:liquibase-core:4.6.1'
        }
    }
}
```

4. Add spring release repo to repositories:

```groovy
repositories {
    // Local maven repository is required for libraries built locally with maven like development jhipster-bom.
    // mavenLocal()
    mavenCentral()
    //jhipster-needle-gradle-repositories - JHipster will add additional repositories
    maven { url 'https://repo.spring.io/release' }
}
```

### Fix dependencies

5. Set explicit version for all spring dependencies

6. Replace undertow with tomcat

```groovy
/*
implementation ("org.springframework.boot:spring-boot-starter-web:${springBootVersion}") {
    exclude module: "spring-boot-starter-tomcat"
}
implementation "org.springframework.boot:spring-boot-starter-undertow"
*/
implementation "org.springframework.boot:spring-boot-starter-web:${springBootVersion}"
implementation "org.springframework.boot:spring-boot-starter-tomcat:${springBootVersion}"
```

7. Replace

```groovy
//implementation ("org.springdoc:springdoc-openapi-webmvc-core")
implementation ("org.springdoc:springdoc-openapi-native:1.6.0")
```

8. Disable spring-cloud-starter-bootstrap

```groovy
//implementation "org.springframework.cloud:spring-cloud-starter-bootstrap:${springBootVersion}"
```

9. Fix logging
   Delete `spring-logback.xml` and reduce logging in application-\*.properties files with:

```
logging:
  level:
    root: ERROR
    io.netty: ERROR
    org.springframework: INFO
```

## Fix User entity

Comment @EntityGraph annotations in UserRepository
Mofiy User entity by adding @EntityListeners(AuditingEntityListener.class) at the top of it

Change authorites property by adding FetchType.EAGER

```java
    @JsonIgnore
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "jhi_user_authority",
        joinColumns = { @JoinColumn(name = "user_id", referencedColumnName = "id") },
        inverseJoinColumns = { @JoinColumn(name = "authority_name", referencedColumnName = "name") }
    )
    @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
    @BatchSize(size = 20)
    private Set<Authority> authorities = new HashSet<>();
```

## Fix mail service

- Add mime.types to META-INF/native-image/resource-config.json include section
- Create MailPropertiesConfiguration
- Manually configure MailSender bean

## Fix thymeleaf template generation (used for mail)

Add Locale hint to GeodataRestSbApp. `@TypeHint` annotation is repeatable so you can add as many type hints as needed. This hint will instruct spring to update the `java.util.Locale` entry in the `reflect-config.json` passed over to native image builder, and the method used by the template will not get deleted in the final image.

```
@TypeHint(types = {java.util.Locale.class}, methods = {@MethodHint(name = "getLanguage")})
```

## Fix Task Executor configuration

This fix is still dubious, but it works. Without it the application behaves as if the Spring didn't invoke the `afterPropertiesSet` method on the `ExceptionHandlingAsyncTaskExecutor` - which it does if you enable trace and debug logs in the relevant classes and check the output.

Anyhow, the problem is solved by invoking the method manually when creating the bean in `AsyncConfiguration`.

```
try {
    taskExecutor.afterPropertiesSet();
    return taskExecutor;
} catch (Exception e) {
    throw new RuntimeException(e);
}
```

## Build native image

Now native image can be built by issueing the command:

```bash
./gradlew bootBuildImage
```

To test if all is working well run:

```bash
docker-compose -f src/main/docker/app-host.yml up
```

This will start both angular web frontend and spring-native powered Rest API.

Now, you can point your browser to: `http://localhost:9000/`

## Known issues

-[ ] Cache does not work -[x] @Async does not work -[ ] EntityGraph does not work -[x] Mail service does not work (workaround fix) -[ ] Logs don’t work (/management/loggers returns HTML instead of JSON) -[ ] Configuration doesn’t work (org.springframework.http.converter.HttpMessageNotWritableException: No converter for [class org.springframework.boot.actuate.context.properties.ConfigurationPropertiesReportEndpoint$ApplicationConfigurationProperties] with preset Content-Type 'null') -[ ] Metrics (JMS Support)

## References:

https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/
https://github.com/mraible/spring-native-examples
https://docs.jboss.org/hibernate/orm/5.4/topical/html_single/bytecode/BytecodeEnhancement.html#_build_time_enhancement
https://www.graalvm.org/reference-manual/native-image/Options/
