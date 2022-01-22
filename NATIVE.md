# Configuring app for Native build

## settings.gradle

Add releases repository and aot plugin:

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

1. Register aot plugin:

```groovy
plugins {
    ...
    //jhipster-needle-gradle-plugins - JHipster will add additional gradle plugins here
    id "org.springframework.experimental.aot"
}
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

5. set explicit version for all spring dependencies


6. Replace undertow with tomcat

```groovy
//implementation "org.springframework.boot:spring-boot-starter-undertow"
implementation "org.springframework.boot:spring-boot-starter-tomcat"
```

7. Replace

```groovy
//implementation ("org.springdoc:springdoc-openapi-webmvc-core")
implementation ("org.springdoc:springdoc-openapi-native:1.6.0")
```

8. Add Hibernate BytecodeEnhancement plugin (NOT SURE we need this?!)

```groovy


```

9. Fix logging
Delete `spring-logback.xml` and reduce logging in application-*.properties files with:

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
-[ ] Cache does not work
-[ ] EntityGraph does not work
-[ ] Mail service does not work
-[ ] 

## References:
https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/
https://github.com/mraible/spring-native-examples
https://docs.jboss.org/hibernate/orm/5.4/topical/html_single/bytecode/BytecodeEnhancement.html#_build_time_enhancement
https://www.graalvm.org/reference-manual/native-image/Options/
