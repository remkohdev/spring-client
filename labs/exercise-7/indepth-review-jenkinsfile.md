# Pipeline Walk-Through

## Syntax

See the [Pipeline Syntax](https://jenkins.io/doc/book/pipeline/syntax/) for a detailed explanation of the syntax of pipelines in Jenkins. You can use the Declarative or the Scripted Syntax, I used the Declarative Syntax in these pipeline, largely based on [Groovy](https://jenkins.io/doc/book/pipeline/syntax/#compare) syntax.

## OpenShift

The base image used to create a Jenkins slave container, includes the `oc` cli. I have mostly used the `oc` cli to communicate with the OpenShift cluster. You can also use the [Openshift Jenkins Client Plugin](https://github.com/openshift/jenkins-client-plugin), which uses an OpenShift plugin that is implemented as a Jenkins Global Variable, its step name, openshift, is a singleton within the context of a job run.

```script
openshift.withCluster() {

}
```

I will work on writing a version using the OpenShift Jenkins Plugin. See the TODOs.

## Jenkins Agent

The Jenkinsfile pipeline starts with defining which image must be used by the current pipeline build, e.g.

```script
pipeline {
  agent {
    label 'maven'
  }
...
}
```

## Setup

In the Setup stage, the pipeline logs into the OpenShift cluster using the credentials defined in the Jenkins Configuration named 'openshift-login-api-token'. The login API token is retrieved from the OpenShift web console. This is not the proper way to identify in the pipeline, because the API token will expire, so in the real environment, you will use the server certificate for instance. 

The [Credentials Binding Plugin](https://jenkins.io/doc/pipeline/steps/credentials-binding/) uses the syntaxt `withCredentials` to bind the token value to the `PASSWORD` variable. There is a long list of bindings available, here the `usernamePassword` binding is used.

```script
withCredentials([usernamePassword(
    credentialsId: 'openshift-login-api-token',
    usernameVariable: 'USERNAME',
    passwordVariable: 'PASSWORD',
  )]) {
        sh "oc login https://c100-e.us-south.containers.cloud.ibm.com:30403 --token=${PASSWORD}"
```

The OpenShift endpoint, using the cluster domain and port, is also hardcoded here, probably also not how you would do it in a real environment. This will be updated in a future version, but for a workshop it is not crucial.

I used to have another line here,

```script
sh 'oc import-image redhat-openjdk-18/openjdk18-openshift:1.6 --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift:1.6 --confirm'
```

But it was not working as expected, and I need to revisit. I was expecting it to pull the base image and store it in the local registry, but I think instead it only creates an imageStream for the image. (Like I said, revisiting this as well).

## Delete Project

I am using the `delete project` command to make sure all objects associated with the deployment are deleted. The problem with doing it this way, is that if there is no project with the specified name, this command will cause a pipeline error. 

Another TODO, add an `openshift.selector().exists()` condition.

## Maven Build

## Unit Tests

## Create Project

See also the `delete project`.

## Deploy

The `--strategy=source` flag runs a Source-to-Image (S2I) deployment strategy. The `oc new-app` looks for a `<image name>~<git repo>` template to build the application image.

The Source-to-Image (S2I) strategy uses a base image `redhat-openjdk-18/openjdk18-openshift`. The base image `Java S2I for OpenShift` uses the `/deployments` directory as the standard location of the deployments directory. In consequence, the Maven build artifect `hello.jar` is copied here. By default, S2I runs the openjdk18 image, which runs `https://github.com/fabric8io-images/java/blob/master/images/jboss/openjdk8/jdk/run-java.sh` to execute a Java application. To run a fat jar you have to specify the `JAVA_APP_JAR` environment var, or the deployment will break with a ClassNotFoundException for the main class. 

```script
sh 'oc new-app --name springclient \'registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift:1.6~https://github.com/remkohdev/spring-client\' --strategy=source --allow-missing-images --build-env=\'JAVA_APP_JAR=hello.jar\''
```

The `--allow-missing-images` flag will download the image if it is not found in the local registry.

## Expose

By default, an OpenShift service is not publically accessible. A route needs to be exposed to make a service available to external requests.

```script
sh 'oc expose svc/springclient'
```
