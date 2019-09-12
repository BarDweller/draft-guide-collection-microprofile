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

the Appsody CLI will launch a local docker image containing a server which will host the microservice. After a period of time you will see messages similar to the following

```
TODO: Oz.
```

this indicates that the server is started and you are ready to begin development.

## Updating the application

We shall create a simple new REST endpoint and add it to the application.

First navigate to the endpoint with a browser to confirm that the endpoint does not exist currently. Open the following link in your browser:
http://localhost:8080/example
You should see an HTTP error TODO

```
TODO: http error goes here. 
```

Within your project folder navigate to the `src/main/java/application` folder. Within the folder create a file named `ExampleEndpoint.java`. Open the file in your editor and update to read

```
package java.application;

TODO.. add code.
```

when saved you should see the source being compiled and the application updated

```
TODO.. spring logs
```

Now if you browse http://localhost:8080/example you will no longer see the HTTP error. The endpoint response will be displayed.

which should show

```
TODO.. endpoint content
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
