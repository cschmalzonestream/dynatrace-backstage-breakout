# dynatrace-backstage-breakout
Get started with the Dynatrace Backstage plugin to enhance the developer experience. See how we used it as a one-stop shop for developers to see their applications and services and proactively gather performance information, such as error logs and quality gate validations.

**Table of contents:**
- [Overview](#overview)
- [Getting Started](#getting-started)
  - [Find a Kubernetes distribution](#find-a-kubernetes-distribution)
  - [Deploy Dynatrace Operator](#deploy-dynatrace-operator)
  - [Deploying Backstage](#deploying-backstage)
    - [Containerize your application](#containerize-your-application)
    - [Deploy container to Kubernetes](#deploy-container-to-kubernetes)
- [Choosing your application to deploy and monitor](#choosing-your-application-to-deploy-and-monitor)
- [Install the Dynatrace Backstage plugin](#install-the-dynatrace-backstage-plugin)
- [Create a template to deploy your application as a component in Backstage](#create-a-template-to-deploy-your-application-as-a-component-in-backstage)
    -  [Create new software components using standard templates](#create-new-software-components-using-standard-template)

## Overview
The usage of the Dynatrace Backstage plugin spawned from wanting to alert on performance degradation from a Site Reliability Guardian (SRG). I wanted the proof of concept (PoC) to be easy to grasp so I decided to containerize an ASP.NET Core weather application. 

This application queried the weather for a given city and would display the results to the user. To showcase the SRG, two versions of this application were created, main & feature, with feature introducing latency to the API call to the weather service. The SRG would detect this given its set SLOs and alert the developer of the performance degradation they introduced to the source code.

I then thought to myself, “How cool would it be if the developer was able to view these quality gate validations alongside performing their daily tasks? This was the birth of selecting an Internal Developer Platform (IDP) and the Dynatrace Backstage plugin.

## Getting Started
Next, I will go through the series of tasks to get this working on your local machine.

### Find a Kubernetes distribution
I first needed a Kubernetes (K8s) environment to host my applications. For this task, I chose to use [K3s](https://k3s.io/) in Windows Subsystem for Linux (WSL). I ran into some [issues](https://community.dynatrace.com/t5/Container-platforms/OneAgent-in-Minikube-failing-to-deploy/m-p/192632) with Minikube and OneAgent but you may have better luck than me if that’s your preferred distribution.

### Deploy Dynatrace Operator
Once you have selected and installed your distribution, you will want to deploy the [Dynatrace Operator](https://docs.dynatrace.com/docs/shortlink/quickstart-k8s#deploy-dynatrace-operator) so you can have observability of your cluster. Ensure the cluster shows up healthy in the Kubernetes app in Dynatrace before continuing to the next step.

### Deploying Backstage
This next session will be in a few parts since you must build the app locally first before deploying it to your cluster.

#### Create your Backstage app
Follow the guide [here](https://backstage.io/docs/getting-started/) paying close attention to the prerequisites.
Once you launch your application, play around with the navigation pane and make sure you can deploy the example template.

![image](https://github.com/user-attachments/assets/25080c94-dd29-451b-8e2d-bdaa284bc2cd)

Note: If you're running Backstage with Node 20 or later, you'll need to pass the flag --no-node-snapshot to Node in order to use the templates feature. One way to do this is to specify the NODE_OPTIONS environment variable before starting Backstage: `export NODE_OPTIONS=--no-node-snapshot`

#### Containerize your application
We are one step closer to seeing Backstage run on your cluster! Head over to this [link](https://backstage.io/docs/deployment/docker/) to learn how to build a Backstage app into a deployable Docker image (Host Build).

Note: I had to change the `ENV NODE_ENV=production` to `ENV NODE_ENV development` to run locally and kept it this way running on K3s for testing purposes. 
I would also swap the `CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]` line in the Dockerfile with `CMD ["node", "packages/backend", "--config", "app-config.yaml"]` when packaging and running the container locally so it wouldn’t try and use the PostgreSQL instance as its database.

You  may choose to run the container locally to test that it’s in a good state before deploying or move forward to running it directly on the cluster.

#### Deploy container to Kubernetes
To allow K3s to pull from the local Docker registry I followed [this]( https://cwienczek.com/2020/06/import-images-to-k3s-without-docker-registry/) guide.

Once that is configured, please follow [this](https://backstage.io/docs/deployment/k8s/#creating-the-backstage-instance) documentation of deploying the Backstage app and needed dependencies to the cluster.

Note: Due to local restrictions I had to run the following command when forwarding the port: `kubectl port-forward --namespace=backstage svc/backstage 8080:80`

Verify that you can access the Backstage app and try deploying the example template. If this succeeds, you are now ready to select your application to deploy and monitor!

## Choosing your application to deploy and monitor
This part will be up to you depending on if you are looking to showcase your company’s application or just trying to put something together for a PoC. In my case, I decided to use the common subject of weather.

I followed [this](https://medium.com/@kmignaz/create-your-forecast-application-with-net-core-and-dockerize-it-945329e0fa83) guide to create the Forecast application and containerize it.

I then made an additional container with a tag of `degradation` where I added the following sleep method on line 17 of `Repositories/ForecastRepository.cs` to have an example of a developer introducing a degradation of service.
```csharp
System.Threading.Thread.Sleep(5000);
```

If you chose to use this app, I have provided some manifests in the resources folder [here](./resources/forecastapp) to deploy both versions to your cluster. If not, before moving on, you will want to make 2 containerized apps of your choice with 1 performing slower than the other and deploy both of those applications to your cluster.

Note: You will want to add the following to your deployment manifests so they can be displayed in the Kubernetes component of the Dynatrace Backstage plugin: `backstage.io/kubernetes-id: <application-name>` I named one `forecastapp` and the other `forecastappfeature`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: forecastapp
  namespace: forecastapp
  labels:
    backstage.io/kubernetes-id: forecastapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: forecastapp
  template:
    metadata:
      labels:
        app: forecastapp
        backstage.io/kubernetes-id: forecastapp
```

Verify you can access both versions of your application on the cluster

![image](https://github.com/user-attachments/assets/5cc1faef-f179-4b18-b891-d79c4ab0a20d) ![image](https://github.com/user-attachments/assets/4aaa5af1-2e4f-40be-b292-4fbe076e6f51)

## Install the Dynatrace Backstage plugin
We are now going to install the Dynatrace Backstage plugin before packaging the final Backstage application.

You can find instructions on how to install the Dynatrace Backstage plugin [here](https://github.com/Dynatrace/backstage-plugin?tab=readme-ov-file#getting-started). Return to this document once you have reached the **Additional Features** section.

You won't notice anything different until you complete the next step. This will unlock the following tab in the component view of your IDP.
![image](https://github.com/user-attachments/assets/10eb52d8-1190-481a-85eb-62f130649289)

## Create a template to deploy your application as a component in Backstage
This step will involve rebulding the container for the Backstage application. I followed [this](https://sagar-parmar.medium.com/creating-infra-using-backstage-templates-terraform-and-github-actions-15ca4a93b1a1) post to learn how to package the template and pull the component but if you are hosting your component on GitHub you will have to [configure]( https://backstage.io/docs/auth/github/provider/) the GitHub Authentication Provider to reach out to your component.

You will then have to edit `app-config.production.yaml`and  `app-config.local` (if testing locally) to point to the template like shown below (path is for deployed container). The path will be different if testing locally.

```yaml
# Local forecastapp template
    - type: file
      target: ./examples/forecastapp.yaml
      rules:
        - allow: [Template]
```

I have provided an example template for the forecast apps [here](./resources/forecastapp/template)

The components can be found [here](./resources/components)

You will have to replace the variables with your own in these files.

### Create new software components using standard template

Now that we have prepped the Backstage application, we can use it to deploy our components!

First, navigate to **Create...** in the navigation pane of the IDP and select **CHOOSE** on the **forecastapp** template.
![image](https://github.com/user-attachments/assets/842d95b8-851b-4558-ad90-051427bd94b7)

Next, let's deploy the main branch first. Input **release** click **NEXT**, check **main**, click **REVIEW** and **CREATE**
![image](https://github.com/user-attachments/assets/e329883f-0df0-4288-b7bb-1264a6c801dd)
![image](https://github.com/user-attachments/assets/3f2955ef-eb59-4a8e-895c-e02609b17a8e)
![image](https://github.com/user-attachments/assets/271fe74c-5b50-41c2-b403-a86fbc150c81)

Once the component is deployed, navigate to **Home** and click on your component.
![image](https://github.com/user-attachments/assets/f273f548-4f1c-4550-8b0d-a7904616b705)

Here you will see the general info of the application where you can choose to launch the existing deployed app on your cluster. This view also includes our star of the show, the Dynatrace Backstage plugin tab!
![image](https://github.com/user-attachments/assets/ea5bc8dd-daf0-4539-9068-a3d3c57510ec)

If you click the **DYNATRACE** tab you should see logs and a link to the kubernetes deployment in Dynatrace
![image](https://github.com/user-attachments/assets/89614b0e-8035-4d07-a8b8-b9d368f5b844)
![image](https://github.com/user-attachments/assets/746c4feb-9e7d-4362-b5bb-194dd98939c1)
![image](https://github.com/user-attachments/assets/642d651a-07e5-4c4b-bdb8-a27451ce9ede)

The one item you won't currently see any data for is the **Site Reliability Guardian**, this will require some additonal setup we will cover in the next section.

Repeat the steps above to deploy the feature branch so you have 2 components entries named **release** and **feature**.
