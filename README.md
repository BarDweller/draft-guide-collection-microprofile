## What you will learn

You will learn how to create, run, update, deploy and deliver a simple cloud native microservice application using the Kabanero java-microprofile collection and Appsody.

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
appsody appsody repo set-default kabanero
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
mkdir -p ~/projects/simple-microprofile
cd ~/projects/simple-microprofile
```

you can now initialise the project using the Appsody CLI

```
appsody init java-microprofile
```

the output from the command will vary depending on whether you have an installation of Java and Maven on your system. The example below is from a system with neither installed.

```
[InitScript] Unable to find any JVMs matching version "(null)".
[InitScript] No Java runtime present, try --request to install.
[InitScript] Unable to find a $JAVA_HOME at "/usr", continuing with system-provided Java...
[InitScript] No Java runtime present, requesting install.
[Warning] The stack init script failed: exit status 1
[Warning] Your local IDE may not build properly, but the Appsody container should still work.
[Warning] To try again, resolve the issue then run `appsody init` with no arguments.
```
Your project is now initialised.

## Understanding the project layout

*TO DO - Lift from Graham Charter workshop*
<Screenshot>

This is intentionally a 'bare-bones' project so as to avoid the need to delete unnecessary files. It contains a
JAX-RS Application class called StarterApplication.java, and Liberty server configuration, server.xml, and static
html file, index.html and the project build file, pom.xml

## Running the Appsody development environment

Start the Appsody development environment in preparation for developing the microservice:

```
appsody run
```

the Appsody CLI will launch a local docker image containing a Open Liberty server which will host the microservice. After a period of time you will see messages similar to the following

```
[Container] [INFO] [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 20.235 seconds.
```

this indicates that the server is started and you are ready to begin development.

## Updating the application

First navigate to the JAX-RS application endpoint to confirm that there are no JAX-RS resources available. Open the following link in your browser:
http://localhost:9080/starter
You should see an HTTP 500 error with the following message stating that there are no provider or resource classes associated with the application:

```
Error 500: javax.servlet.ServletException: At least one provider or resource class should be specified for application class "dev.appsody.starter.StarterApplication
```

Within your project folder navigate to the `src/main/java/dev/appsody/starter` folder. Within the folder create a file named `StarterResource.java`. Open the file in your editor and update to read

```
package dev.appsody.starter;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
@Path("/resource")
public class StarterResource {
    @GET
    public String getRequest() {
        return "StarterResource response";
    }
}
```

when saved you should see the source being compiled and the application updated

```
[Container] [INFO] [AUDIT   ] CWWKT0017I: Web application removed (default_host): http://85862d8696be:9080/
[Container] [INFO] [AUDIT   ] CWWKZ0009I: The application starter-app has stopped successfully.
[Container] [INFO] [AUDIT   ] CWWKT0016I: Web application available (default_host): http://85862d8696be:9080/
[Container] [INFO] [AUDIT   ] CWWKZ0003I: The application starter-app updated in 0.988 seconds.
```

Now if you browse http://localhost:9080/starter you will no longer see the HTTP 500 error. The resource we just added is actually available at a location under `starter`. Browse the following URL to see the resource response:

```
http://localhost:9080/starter/resource
```

which should show

```
StarterResource response
```

Try changing the message in `StarterResource.java` saving and refreshing the page. You'll see it only takes a few seconds for the change to take effect.

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
simple-microprofile-77d6868765-bhd8x   1/1     Running   0          3m21s
```

once the simple-microprofile pod is started go to the URL from the deploy step and you should see the Appsody microservice splash screen. Got to http://localhost:30262/starter/resource and you will see your deployed application response.
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
appsody deploy --tag dev.local/simple-microprofile --namespace <namespace>
```

or to deploy from docker hub use

```
appsody deploy --push -—tag <my-account>/simple-microprofile --namespace <namespace>
```

once deployment completes you should see a message detailing the serving url

```
Deployed project running at "http://simple-microprofile.knative-serving.192.168.1.10.nip.io"
```

browse to `http://simple-microprofile.knative-serving.192.168.1.10.nip.io/starter/resource` to see the response from your application.


## Deliver to enterprise pipelines

Once you have completed you development and testing it is time to deliver the application to your enterprise Kabanero pipelines.
This is a simple step of pushing the project to a pre-configured git repository (public or enterprise). Your operations team should have configured the necessary hooks on the repository to trigger the pipelines.
