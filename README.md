# SpringClient - A Simple Spring Application

This application demonstrates simple examples for how to use Spring Boot, Spring MVC and Spring for REST APIs and Spring Service.

## Lab

A Lab or Workshop accompanies this application:

* Setup and configuration instructions, see [labs/exercise-7/README0.md](labs/exercise-7/README0.md),
* Lab: Deploying a Spring Boot App using with Jenkinsfile Using Source-to-Image (S2I) in Jenkings on Openshift, see [labs/exercise-7/README.md](labs/exercise-7/README.md), 
* Extra: [Indepth Review of the Jenkinsfile](labs/exercise-7/indepth-review-jenkinsfile.md),

## Run

Using the Maven Wrapper,

```console
user@host:~$ ./mvnw clean install
user@host:~$ ./mvnw spring-boot:run
```

Without Maven Wrapper,

```console
user@host:~$ mvn clean install
user@host:~$ mvn spring-boot:run
```

## Test

```console
user@host:~$ curl -X GET 'http://localhost:8080/api/hello?name=pluto'
```

## Maven

To download all dependencies for development, run 

```console
user@host:~$ mvn dependency:copy-dependencies
```

## Deployment

The `Jenkinsfile` is written for a Source-to-Image (S2I) deployment to OpenShift 3.11.

Lab coming soon...

## Docs

See [Java: Get Started with Spring Boot](https://medium.com/nycdev/big-java-get-booted-with-spring-1896055c3803) and [Java: Create a Spring MVC App with Tomcat and Maven](https://medium.com/nycdev/java-build-a-tomcat-web-app-with-maven-and-spring-fbc823fa9a37) for a tutorial on Spring Boot and Spring MVC.

## TODO

* Replace the 'oc' syntax for the [OpenShift command line interface (CLI)](https://docs.openshift.com/container-platform/3.11/cli_reference/index.html) by the syntax used for the [OpenShift Jenkins Pipeline (DSL) Plugin](https://github.com/openshift/jenkins-client-plugin).
* Another TODO, add an `openshift.selector().exists()` condition, before deleting. Currently there is manual step required to make sure the project exists. In addition, if deleting the project resources has not completed fully, the create project will also become an issue. In other words, not fool proof right now.
* The `oc new-app` takes a full path URI to the base image, but there has to be a way to reference this by tag and perhaps use the `oc import-image` as a separate step to make sure it exists. Didn't have time to fix this part.
