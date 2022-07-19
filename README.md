# geodataRestSb
Spring Boot **rest only** implementation of **[geodataApp][]**.

This application was generated using JHipster 7.5.0, you can find documentation and help at [https://www.jhipster.tech/documentation-archive/v7.5.0](https://www.jhipster.tech/documentation-archive/v7.5.0).

![](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=java&logoColor=white&style=flat)
![](https://img.shields.io/badge/Spring-6DB33F?style=for-the-badge&logo=spring&logoColor=white&style=flat)

## Project Structure

Node is required for generation and recommended for development. `package.json` is always generated for a better development experience with prettier, commit hooks, scripts and so on.

In the project root, JHipster generates configuration files for tools like git, prettier, eslint, husk, and others that are well known and you can find references in the web.

`/src/*` structure follows default Java structure.

- `.yo-rc.json` - Yeoman configuration file
  JHipster configuration is stored in this file at `generator-jhipster` key. You may find `generator-jhipster-*` for specific blueprints configuration.
- `.yo-resolve` (optional) - Yeoman conflict resolver
  Allows to use a specific action when conflicts are found skipping prompts for files that matches a pattern. Each line should match `[pattern] [action]` with pattern been a [Minimatch](https://github.com/isaacs/minimatch#minimatch) pattern and action been one of skip (default if ommited) or force. Lines starting with `#` are considered comments and are ignored.
- `.jhipster/*.json` - JHipster entity configuration files
- `npmw` - wrapper to use locally installed npm.
  JHipster installs Node and npm locally using the build tool by default. This wrapper makes sure npm is installed locally and uses it avoiding some differences different versions can cause. By using `./npmw` instead of the traditional `npm` you can configure a Node-less environment to develop or test your application.
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
### Running geodata-rest-sb application

For **geodata-rest-sb** application to be run you need to specify one runtime env variable **GRSB_LIQUIBASE_ENABLED** with value set to `true` or `false`.
This variable controls if **geodata-rest-sb** liquibase is enabled or not. 

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

You can fully dockerize your application and all the services that it depends on.
To achieve this, you first need to build a docker image of **geodata-rest-sb** app (for more see section below).

Then run:

```
docker-compose -f src/main/docker/app.yml up -d
```
This will start, inside the docker container, the following services:
- geodata-rest-sb
- geodata angular frontend
- postgres database

Now, you can access the application by openning `http://localhost:9000/` in your browser.

To start only geodata-rest-sb and geodata-ng applications, which connect to local host Postgres database, run:

```
docker-compose -f src/main/docker/app-host.yml up -d
```

For more information refer to [Using Docker and Docker-Compose][], this page also contains information on the docker-compose sub-generator (`jhipster docker-compose`), which is able to generate docker configurations for one or several JHipster applications.


## Buidling docker image
JIB plugin is used to build **geodata-rest-sb** application docker image.

To build local image use the following command:

```bash
./gradlew bootJar -Pprod jibDockerBuild
```
This will build image named ` ag04/geodata-rest-sb ` in your local registry with the tag same as the current project version.

On the other hand, the command:

```bash
./gradlew bootJar -Pprod jib
```
will build image named `ag04/geodata-rest-sb` in docker.io registry, also with the tag same as the project version.

Image name, version and docker Registry used in container image build can be customized by passing these arguments:

| Argument name     | Description                    |
|-------------------|--------------------------------|
| imageName         | Name of the image to be built  |
| imageVersion      | Image tag to be used insted of project version  |
| dockerRegistryUrl | URL of the docker registry this image should be pushed to (applicable only for jib command) |


For example, to build (local) image wiht the `latest` tag run:

```bash
./gradlew bootJar -Pprod jibDockerBuild -PimageVersion=latest
```

For all other parameters available (docker registry authentication etc) for jib plugin please see official plugin docs:
https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin

## Continuous Integration (optional)

To configure CI for your project, run the ci-cd sub-generator (`jhipster ci-cd`), this will let you generate configuration files for a number of Continuous Integration systems. Consult the [Setting up Continuous Integration][] page for more information.

[jhipster homepage and latest documentation]: https://www.jhipster.tech
[jhipster 7.5.0 archive]: https://www.jhipster.tech/documentation-archive/v7.5.0
[using jhipster in development]: https://www.jhipster.tech/documentation-archive/v7.5.0/development/
[using docker and docker-compose]: https://www.jhipster.tech/documentation-archive/v7.5.0/docker-compose
[using jhipster in production]: https://www.jhipster.tech/documentation-archive/v7.5.0/production/
[running tests page]: https://www.jhipster.tech/documentation-archive/v7.5.0/running-tests/
[code quality page]: https://www.jhipster.tech/documentation-archive/v7.5.0/code-quality/
[setting up continuous integration]: https://www.jhipster.tech/documentation-archive/v7.5.0/setting-up-ci/
[node.js]: https://nodejs.org/
[npm]: https://www.npmjs.com/
[geodataApp]: https://github.com/dmadunic/geodata-app

# Full Application Kubernetes Deployment (minikube)

## Prerequisites

### Installed tools
- docker
- minikube
- Virtualbox

## Setup minikube

Start minikube.

```sh
minikube start --driver=virtualbox --no-vtx-check
```

### Enable ingress addon

```sh
minikube addons enable ingress
```

## 1. Deploy postgres

First we need to create config and secret maps:
```bash
kubectl apply -f postgres/postgres-config.yml
kubectl apply -f postgres/postgres-secret.yml
```
Now persistent volume:

```bash
kubectl create -f postgres/postgres-pv.yml
kubectl create -f postgres/postgres-pvc.yml
```

Check if all went well by issuing:

```bash
kubectl get pvc
```
You should see the similar output:

```bash
NAME                STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pv-claim   Bound    postgres-pv   5Gi        RWX            manual         35s
```

Now lets deploy postgres to kubernetes

```bash
kubectl apply -f k8s/postgres/postgres-deployment.yml
```

We need to expose database to others as service, so execue the following command:

```bash
kubectl apply -f k8s/postgres/postgres-service.yml
```

To check if everything is working well, execute:

```bash
kubectl get service postgres-service
```

You shoud see something like:

```
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
postgres-service   ClusterIP   10.106.77.188   <none>        5432/TCP   7m25s
```

We can expose this service, to the outside world by using port forwarding:

```bash
kubectl port-forward service/postgres-service 5435:5432
```

**Now startup dbeaver and connect to databse on localhost:5435**

Once all is working as expected kill port-forwarding by pressing ^C inside the terminal.

# 2. Create geodata namespace and resources

Position yourself in terminal in the root folder of this repository.

```bash
kubectl create -f k8s/geodata/namespace.yml
```

## Deploy geodata-rest application

First we need to create config and secret maps that hold rest-api configuration parameteres.

```bash
kubectl apply -f k8s/geodata/geodata-config.yml
kubectl apply -f k8s/geodata/geodata-secret.yml
```
Now it is time to create geodata-rest deployment and service:

```bash
kubectl apply -f k8s/geodata/rest/geodata-rest-deployment.yml
kubectl apply -f k8s/geodata/rest/geodata-rest-service.yml
```

## Deploy geodata-web application

```bash
kubectl apply -f k8s/geodata/web/geodata-web-deployment.yml
kubectl apply -f k8s/geodata/web/geodata-web-service.yml
```

## Expose geodata app with NodePort

```bash
kubectl expose deployment geodata-web-deployment --type=NodePort -n geodata --name geodata-web-np
```

Fetch URL
```bash  
echo $(minikube ip):$(kubectl get service geodata-web-np -n geodata -o jsonpath='{.spec.ports[0].nodePort}')
```

## Expose geodata app with LoadBalancer

```bash 
kubectl expose deployment geodata-web-deployment --type=LoadBalancer -n geodata --name geodata-web-lb
```

```bash
minikube tunnel
```

Fetch LoadBalancer IP
```bash
echo $(kubectl get service geodata-web-lb -n geodata -o jsonpath='{.status.loadBalancer.ingress[].ip}')
```

Use LoadBalancer IP address in browser


## Create ingress and expose geodata app

```bash
kubectl apply -f k8s/geodata/ingress.yml
```

Executing the command above will create, in geodata namespace, an ingress service that exposes geodata-web application on url: `geodata.local-minikube.io` on port 80.

Add these entry to /etc/hosts file 
(replace MINIKUBE_IP with your minikube ip address)

```bash
MINIKUBE_IP    geodata.local-minikube.io
```

## Single ingress in the default namespace

1. Remove ingress service created in geodata namespace
```bash
kubectl delete ingress -n geodata geodata-ingress
```

2. Create external service for geodata-web-service
```bash
kubectl apply -f default/geodata-service-svc.yml
```

3. Create ingress in default namespace
```bash
kubectl apply -f default/ingress-ns-default.yml
```




