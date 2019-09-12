## What you will learn

You will learn how to create, run, update, deploy and deliver a simple cloud native microservice application using the Kabanero java-spring-boot2 collection and Appsody.

## Prerequisites

To complete the guide you will need to:

1. Install Docker *TO DO - Do we need to include this step*
2. Have access to k8s cluster
3. Install the Appsdocy CLI*TO DO - Do we link to instructions or have to add them*
4. Have access to the Kabanero index file you will be using: *TO DO - This could be a direct link to out github release or may be enterprise specific*

## Getting Started

First you need to add your Kabanero index to the Appsody CLI, in the example below we use the public index for Kabanero 0.1.2:

First check the existing repositories

```
appsody repo list
```
You should see output similar to

```
appsody repo list
NAME        URL
*appsodyhub https://github.com/appsody/stacks/releases/latest/download/incubator-index.yaml
```

Now add the Kabanero index

```
appsody repo add kabanero https://github.com/kabanero-io/collections/releases/download/v0.1.2/kabanero-index.yaml
```
and check it has been added

```
appsody repo list
NAME        URL
*appsodyhub https://github.com/appsody/stacks/releases/latest/download/incubator-index.yaml
kabanero    https://github.com/kabanero-io/collections/releases/download/v0.1.2/kabanero-index.yaml
```

Set the Kabanero index to be the default

```
appsody repo set-default kabanero
```

Check it updated

```
appsody repo list
NAME        URL
appsodyhub  https://github.com/appsody/stacks/releases/latest/download/incubator-index.yaml
*kabanero   https://github.com/kabanero-io/collections/releases/download/v0.1.2/kabanero-index.yaml
```

You Appsody CLI is now configured to use the Kabanero collections.
Next you need to initialise you project, first create a directory to contain it

```
mkdir -p ~/projects/simple-spring-boot2
cd ~/projects/simple-spring-boot2
```

you can now initialise the project using the Appsody CLI

```
appsody init java-spring-boot2
```

the output from the command will vary depending on whether you have an installation of Java on your system, the output below is from a system that has Java.

```
Running appsody init...
Downloading java-spring-boot2 template project from https://github.com/kabanero-io/collections/releases/download/v0.1.2/incubator.java-spring-boot2.v0.3.9.templates.default.tar.gz
Download complete. Extracting files from java-spring-boot2.tar.gz
Setting up the development environment
Running command: docker[pull kabanero/java-spring-boot2:0.3]
Running command: docker[run --rm --entrypoint /bin/bash kabanero/java-spring-boot2:0.3 -c find /project -type f -name .appsody-init.sh]
Extracting project from development environment
Running command: docker[create --name my-project-extract -v /home/username/projects/simple-spring-boot2/.:/project/user-app -v /home/username/.m2/repository:/mvn/repository kabanero/java-spring-boot2:0.3]
Running command: docker[cp my-project-extract:/project /home/username/.appsody/extract/simple-spring-boot2]
Running command: docker[rm my-project-extract -f]
Project extracted to /home/username/projects/simple-spring-boot2/.appsody_init
Running command: ./.appsody-init.sh[]
Successfully initialized Appsody project

```
Your project is now initialised.

## Understanding the project layout

```
./.vscode
./.gitignore
./.appsody-config.yaml


./src/main/java/application/Main.java
./src/main/java/application/LivenessEndpoint.java

./src/test/java/application/MainTests.java

./src/main/resources/application.properties
./src/main/resources/public/index.html

./pom.xml
```

This is intentionally a 'bare-bones' project so as to avoid the need to delete unnecessary files. It contains a
Spring Application class called Main.java, an example Liveness Endpoint called LivenessEndpoint.java, a simple test class, 
some Spring configuration in application.properties, a static index.html and the project build file, pom.xml

## Running the Appsody development environment

Start the Appsody development environment in preparation for developing the microservice:

```
appsody run
```

the Appsody CLI will launch a local docker container that will compile and host the microservice. After a period of time you will see messages similar to the following

```
[Container] 2019-09-12 17:28:44.066  INFO 171 --- [  restartedMain] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 4 endpoint(s) beneath base path '/actuator'
[Container] 2019-09-12 17:28:44.205  INFO 171 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
[Container] 2019-09-12 17:28:44.209  INFO 171 --- [  restartedMain] application.Main                         : Started Main in 6.051 seconds (JVM running for 6.923)
```

this indicates that the server is started and you are ready to begin development.

## Updating the application

We shall create a simple new REST endpoint and add it to the application.

First navigate to the endpoint with a browser to confirm that the endpoint does not exist currently. Open the following link in your browser:
http://localhost:8080/example
You should see an HTTP error 404 with Spring's default "Whitelabel Error Page".

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Thu Sep 12 17:29:43 UTC 2019
There was an unexpected error (type=Not Found, status=404).
No message available
```

With the container still running, navigate enter your project folder and navigate to the `src/main/java/application` directory. Within the directory create a file named `ExampleEndpoint.java`. Open the file in your editor and update to read

```
package application;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ExampleEndpoint {

    @RequestMapping("/example")
    public String example() {
        return "This is an example";
    }
}
```

when saved you should see the source being compiled and the application updated

```
[Container] Running: /project/java-spring-boot2-build.sh recompile
[Container] Compile project in the foreground
[Container] > mvn compile
[Container] [INFO] Scanning for projects...
[Container] [INFO] 
[Container] [INFO] ----------------------< dev.appsody:application >-----------------------
[Container] [INFO] Building application 0.0.1-SNAPSHOT
[Container] [INFO] --------------------------------[ jar ]---------------------------------
[Container] [INFO] 
[Container] [INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ application ---
[Container] [INFO] Using 'UTF-8' encoding to copy filtered resources.
[Container] [INFO] Copying 2 resources
[Container] [INFO] 
[Container] [INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ application ---
[Container] [INFO] Changes detected - recompiling the module!
[Container] [INFO] Compiling 3 source files to /project/user-app/target/classes
[Container] [INFO] 
[Container] [INFO] --- maven-antrun-plugin:1.1:run (trigger-spring-restart) @ application ---
[Container] [INFO] Executing tasks
[Container]      [echo] Triggering Spring app restart.
[Container] [INFO] Executed tasks
[Container] [INFO] ------------------------------------------------------------------------
[Container] [INFO] BUILD SUCCESS
[Container] [INFO] ------------------------------------------------------------------------
[Container] [INFO] Total time:  3.585 s
[Container] [INFO] Finished at: 2019-09-12T17:34:37Z
[Container] [INFO] ------------------------------------------------------------------------
[Container] 2019-09-12 17:34:38.316  INFO 171 --- [      Thread-15] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
[Container] 
[Container]   .   ____          _            __ _ _
[Container]  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
[Container] ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
[Container]  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
[Container]   '  |____| .__|_| |_|_| |_\__, | / / / /
[Container]  =========|_|==============|___/=/_/_/_/
[Container]  :: Spring Boot ::        (v2.1.6.RELEASE)
...
[Container] 2019-09-12 17:34:39.711  INFO 171 --- [  restartedMain] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 4 endpoint(s) beneath base path '/actuator'
[Container] 2019-09-12 17:34:39.772  INFO 171 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
[Container] 2019-09-12 17:34:39.773  INFO 171 --- [  restartedMain] application.Main                         : Started Main in 1.403 seconds (JVM running for 362.487)
[Container] 2019-09-12 17:34:39.788  INFO 171 --- [  restartedMain] .ConditionEvaluationDeltaLoggingListener : Condition evaluation unchanged
```

Now if you browse http://localhost:8080/example you will no longer see the HTTP error. The endpoint response will be displayed.

This should show;

```
This is an example
```

Try changing the message in `ExampleEndpoint.java` saving and refreshing the page. You'll see it only takes a few seconds for the change to take effect.

## Stopping the Appsody development environment

To stop the Appsody development environment issue the following in the same window you issued `appsody run`
```
Ctrl-C
```

## Deploying to K8s

Once you have finished writing your application code the Appsody CLI makes it easy to deploy to a kubernetes cluster for further tesing. So long as your `kubectl` command is configured with cluster details you can deploy the application using

```
appsody deploy
```

this command will build a new docker image optimised for production deployment and deploy it to your k8s cluster, after a period of time you will see a message similar to

```
Deployed project running at http://localhost:30262
```

you can check the status of the application pods using

```
kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
appsody-operator-859b97bb98-xm8nl      1/1     Running   1          8d
simple-spring-boot2-77d6868765-bhd8x   1/1     Running   0          3m21s
```

once the simple-spring-boot2 pod is started go to the URL from the deploy step and you should see the Appsody microservice splash screen. Got to http://localhost:30262/example and you will see your deployed application response.
To stop the deployed application use

```
appsody deploy delete
```

this will remove the deployment and display a message

```
Deployment deleted
```

## Deploying to KNative

You can also choose to deploy the application vi KNative serving. To do this you must first install KNative in your kubernetes cluster, see https://knative.dev/docs/install/.
Once installed generate a `app-deploy.yaml` file using

```
appsody deploy —generate-only
```

open the file in your editor and add the following to the spec definition

```
createKnativeService: true
```

to deploy from your local image registry use

```
appsody deploy --tag dev.local/simple-spring-boot2 --namespace <namespace>
```

or to deploy from docker hub use

```
appsody deploy --push -—tag <my-account>/simple-spring-boot2 --namespace <namespace>
```

once deployment completes you should see a message detailing the serving url

```
Deployed project running at "http://simple-spring-boot2.knative-serving.192.168.1.10.nip.io"
```

browse to `http://simple-spring-boot2.knative-serving.192.168.1.10.nip.io/example` to see the response from your application.


## Deliver to enterprise pipelines

Once you have completed you development and testing it is time to deliver the application to your enterprise Kabanero pipelines.
This is a simple step of pushing the project to a pre-configured git repository (public or enterprise). Your operations team should have configured the necessary hooks on the repository to trigger the pipelines.
