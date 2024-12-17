# dynatrace-backstage-breakout
Get started with the Dynatrace Backstage plugin to enhance the developer experience. See how we used it as a one-stop shop for developers to see their applications and services and proactively gather performance information, such as error logs and quality gate validations.

**Table of contents:**
- [Overview](#overview)
- [Getting Started](#getting-started)

## Overview
The usage of the Dynatrace Backstage plugin spawned from wanting to alert on performance degradation from a Site Reliability Guardian (SRG). I wanted the proof of concept (PoC) to be easy to grasp so I decided to containerize an ASP.NET Core weather application. 

This application queried the weather for a given city and would display the results to the user. To showcase the SRG, two versions of this application were created, main & feature, with feature introducing latency to the API call to the weather service. The SRG would detect this given its set SLOs and alert the developer of the performance degradation they introduced to the source code.

I then thought to myself, “How cool would it be if the developer was able to view these quality gate validations alongside performing their daily tasks? This was the birth of selecting an Internal Developer Platform (IDP) and the Dynatrace Backstage plugin.

## Getting Started
Next, I will go through the series of tasks to get this working on your local machine.

### Find a Kubernetes distribution
I first needed a Kubernetes (K8s) environment to host my applications. For this task, I chose to use [K3s](https://k3s.io/) in Windows Subsystem for Linux (WSL). I ran into some issues with Minikube and OneAgent but you may have better luck than me if that’s your preferred distribution: https://community.dynatrace.com/t5/Container-platforms/OneAgent-in-Minikube-failing-to-deploy/m-p/192632

### Deploy Dynatrace Operator
Once you have selected and installed your distribution, you will want to deploy the [Dynatrace Operator](https://docs.dynatrace.com/docs/shortlink/quickstart-k8s#deploy-dynatrace-operator) so you can have observability of your cluster. Ensure the cluster shows up healthy in the Kubernetes app in Dynatrace before continuing to the next step.

### Deploying Backstage
This next session will be in a few parts since you must build the app locally first before deploying it to your cluster.
#### Create your backstage app
Follow the guide [here](https://backstage.io/docs/getting-started/) paying close attention to the prerequisites.
Once you launch your application, play around with the navigation pane and make sure you can deploy the example template.

![image](https://github.com/user-attachments/assets/25080c94-dd29-451b-8e2d-bdaa284bc2cd)

Note: If you're running Backstage with Node 20 or later, you'll need to pass the flag --no-node-snapshot to Node in order to use the templates feature. One way to do this is to specify the NODE_OPTIONS environment variable before starting Backstage: export NODE_OPTIONS=--no-node-snapshot

### Containerize your application
We are one step closer to seeing Backstage run on your cluster! Head over to this [link](https://backstage.io/docs/deployment/docker/) to learn how to build a Backstage App into a deployable Docker image (Host Build).

Note: I had to change the “ENV NODE_ENV=production” to “ENV NODE_ENV development” to run locally and kept it this way running on K3s for testing purposes. 
I would also swap the “CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]” line in the Dockerfile with “CMD ["node", "packages/backend", "--config", "app-config.yaml"]” when packaging and running the container locally so it wouldn’t try and use the PostgreSQL instance as its database.

You may choose to run the container locally or move forward to running it directly on the cluster.
