# geodataSbRest
Spring Boot rest only version of **[geodataApp][]**.

This application was generated using JHipster 7.4.1, you can find documentation and help at [https://www.jhipster.tech/documentation-archive/v7.4.1](https://www.jhipster.tech/documentation-archive/v7.4.1).

![](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=java&logoColor=white&style=flat)
![](https://img.shields.io/badge/Bootstrap-563D7C?style=for-the-badge&logo=bootstrap&logoColor=white&style=flat)
![](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white&style=flat)

## Project Structure

Node is required for generation and recommended for development. `package.json` is always generated for a better development experience with prettier, commit hooks, scripts and so on.

In the project root, JHipster generates configuration files for tools like git, prettier, eslint, husk, and others that are well known and you can find references in the web.

`/src/*` structure follows default Java structure.

- `.yo-rc.json` - Yeoman configuration file
  JHipster configuration is stored in this file at `generator-jhipster` key. You may find `generator-jhipster-*` for specific blueprints configuration.
- `.yo-resolve` (optional) - Yeoman conflict resolver
  Allows to use a specific action when conflicts are found skipping prompts for files that matches a pattern. Each line should match `[pattern] [action]` with pattern been a [Minimatch](https://github.com/isaacs/minimatch#minimatch) pattern and action been one of skip (default if ommited) or force. Lines starting with `#` are considered comments and are ignored.
- `.jhipster/*.json` - JHipster entity configuration files
- `/src/main/docker` - Docker configurations for the application and services that the application depends on

## Development

### 1. Database setup (optional)
This step needs to be performed only if you have not already done it while preparing [geodataApp][].

If you already do not have it, create a database named: **ag04**.
Connect to database with user that has sufficient privileges and execute:

```sql
CREATE DATABASE ag04;
```

The next step is to create **geodata** user and his corresponding schema.
To do so execute the following sql commands:

```sql
CREATE ROLE geodata NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT LOGIN PASSWORD 'geodatapwd';
GRANT ALL PRIVILEGES ON DATABASE ag04 TO geodata;
```

Disconnect from "default" database, and connect to ag04 database using the same user as in the previous steps.

```sql
CREATE SCHEMA IF NOT EXISTS AUTHORIZATION "geodata";
```
### Running geodata-sb-rest application

For **geodata-sb-rest** application to be run you need to specify one runtime env variable **GSBR_LIQUIBASE_ENABLED** with value set to `true` or `false`.
This variable controls if **geodata-sb-rest** liquibase is enabled or not. 

**It should not be enabled if you have already created all database objects and populated them (for example while running [geodataApp][]).**

Once, you have this env var in place start your application in the dev profile by running:

```
./gradlew
```

For further instructions on how to develop with JHipster, have a look at [Using JHipster in development][].

### JHipster Control Center

JHipster Control Center can help you manage and control your application(s). You can start a local control center server (accessible on http://localhost:7419) with:

```
docker-compose -f src/main/docker/jhipster-control-center.yml up
```

## Building for production

### Packaging as jar

To build the final jar and optimize the geodataSbRest application for production, run:

```
./gradlew -Pprod clean bootJar
```

To ensure everything worked, run:

```
java -jar build/libs/*.jar
```

Refer to [Using JHipster in production][] for more details.

### Packaging as war

To package your application as a war in order to deploy it to an application server, run:

```
./gradlew -Pprod -Pwar clean bootWar
```

## Testing

To launch your application's tests, run:

```
./gradlew test integrationTest jacocoTestReport
```

For more information, refer to the [Running tests page][].

### Code quality

Sonar is used to analyse code quality. You can start a local Sonar server (accessible on http://localhost:9001) with:

```
docker-compose -f src/main/docker/sonar.yml up -d
```

Note: we have turned off authentication in [src/main/docker/sonar.yml](src/main/docker/sonar.yml) for out of the box experience while trying out SonarQube, for real use cases turn it back on.

You can run a Sonar analysis with using the [sonar-scanner](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner) or by using the gradle plugin.

Then, run a Sonar analysis:

```
./gradlew -Pprod clean check jacocoTestReport sonarqube
```

For more information, refer to the [Code quality page][].

## Using Docker to simplify development (optional)

You can use Docker to improve your JHipster development experience. A number of docker-compose configuration are available in the [src/main/docker](src/main/docker) folder to launch required third party services.

For example, to start a postgresql database in a docker container, run:

```
docker-compose -f src/main/docker/postgresql.yml up -d
```

To stop it and remove the container, run:

```
docker-compose -f src/main/docker/postgresql.yml down
```

You can also fully dockerize your application and all the services that it depends on.
To achieve this, you first need to build a docker image of **geodata-sb-rest** app (see section below).

Then run:

```
docker-compose -f src/main/docker/app.yml up -d
```

For more information refer to [Using Docker and Docker-Compose][], this page also contains information on the docker-compose sub-generator (`jhipster docker-compose`), which is able to generate docker configurations for one or several JHipster applications.


## Buidling docker image
JIB plugin is used to build docker image of the spotsie-rest subproject.

To build local image use the follwoing command:

```bash
./gradlew bootJar -Pprod jibDockerBuild
```
This will build image named ` ag04/geodata-sb-rest ` in your local registry with the tag equal to the project version.

On the other hand, this command:
```bash
./gradlew bootJar -Pprod jib
```
Will build image named `ag04/geodata-sb-res` in docker.io registry with the tag equal to the project version.

Image name, version and docker Registry to be used in buildcan be customized by passing these arguments:

| Argument name     | Description                    |
|-------------------|--------------------------------|
| imageName         | Name of the image to be built  |
| imageVersion      | Image tag to be used insted of project version  |
| dockerRegistryUrl | URL of the docker registry this image should be pushed to (applicable only for jib command) |

To build (local) image wiht the latest tag run:
```bash
./gradlew bootJar -Pprod jibDockerBuild -PimageVersion=latest
```

For all other parameters available (docker registry authentication etc) for jib plugin please see official plugin docs:
https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin

## Continuous Integration (optional)

To configure CI for your project, run the ci-cd sub-generator (`jhipster ci-cd`), this will let you generate configuration files for a number of Continuous Integration systems. Consult the [Setting up Continuous Integration][] page for more information.

[jhipster homepage and latest documentation]: https://www.jhipster.tech
[jhipster 7.4.1 archive]: https://www.jhipster.tech/documentation-archive/v7.4.1
[using jhipster in development]: https://www.jhipster.tech/documentation-archive/v7.4.1/development/
[using docker and docker-compose]: https://www.jhipster.tech/documentation-archive/v7.4.1/docker-compose
[using jhipster in production]: https://www.jhipster.tech/documentation-archive/v7.4.1/production/
[running tests page]: https://www.jhipster.tech/documentation-archive/v7.4.1/running-tests/
[code quality page]: https://www.jhipster.tech/documentation-archive/v7.4.1/code-quality/
[setting up continuous integration]: https://www.jhipster.tech/documentation-archive/v7.4.1/setting-up-ci/
[node.js]: https://nodejs.org/
[npm]: https://www.npmjs.com/
[geodataApp]: https://github.com/dmadunic/geodata-app
