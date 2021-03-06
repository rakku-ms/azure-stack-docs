---
title: Deploy Azure Cognitive Services to Azure Stack Hub 
description: Learn how to deploy Azure Cognitive Services to Azure Stack Hub.
author: mattbriggs

ms.topic: article
ms.date: 05/13/2020
ms.author: mabrigg
ms.reviewer: guanghu
ms.lastreviewed: 05/13/2020

# Intent: As an Azure Stack user, I want to deploy cognitive services on Azure Stack using containers so I can use the benefits of Azure Cognitive Services. 
# Keyword: azure stack cognitive services

---

# Deploy Azure Cognitive Services to Azure Stack Hub

You can use Azure Cognitive Services with container support on Azure Stack Hub. Container support in Azure Cognitive Services allows you to use the same rich APIs that are available in Azure. Your use of containers enables flexibility in where to deploy and host the services delivered in [Docker containers](https://www.docker.com/what-container). 

Containerization is an approach to software distribution in which an app or service, including its dependencies and configuration, are packaged as a container image. With little or no modification, you can deploy an image to a container host. Each container is isolated from other containers and from the underlying operating system. The system itself only has the components needed to run your image. A container host has a smaller footprint than a virtual machine. You can also create containers from images for short-term tasks and they can be removed when no longer needed.

Container support is currently available for a subset of Azure Cognitive Services:

- Language Understanding
- Text Analytics (Sentiment 3.0)

> [!IMPORTANT]
> A subset of Azure Cognitive Services for Azure Stack Hub are currently in public preview.
> The review version is provided without a service level agreement, and it's not recommended for production workloads. Certain features might not be supported or might have constrained capabilities. 
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Container support is currently in public preview for a subset of Azure Cognitive Services:

- Read (optical character recognition \[OCR])
- Key phrase extraction
- Language detection
- Anomaly detector
- Form recognizer
- Speech-to-text (custom, standard)
- Text-to-speech (custom, standard)

## Use containers with Cognitive Services on Azure Stack Hub

- **Control over data**  
  Allow your app users to have control over their data while using Cognitive Services. You can deliver Cognitive Services to app users who can't send data to global Azure or the public cloud.

- **Control over model updates**  
  Provide app users version updates to the models deployed in their solution.

- **Portable architecture**  
  Enable the creation of a portable app architecture so that you can deploy your solution to the public cloud, to a private cloud on-premises, or the edge. You can deploy your container to Azure Kubernetes Service, Azure Container Instances, or to a Kubernetes cluster in Azure Stack Hub. For more information, see [Deploy Kubernetes to Azure Stack Hub](azure-stack-solution-template-kubernetes-deploy.md).

- **High throughput and low latency**  
   Provide your app users the ability to scale with spikes in traffic for high throughput and low latency. Enable Cognitive Services to run in Azure Kubernetes Service physically close to their app logic and data.

With Azure Stack Hub, deploy Cognitive Services containers in a Kubernetes cluster along with your app containers for high availability and elastic scaling. You can develop your app by combining Cognitive services with components built on App Services, Functions, Blob storage, SQL, or mySQL databases.

For more details on Cognitive Services containers, go to [Container support in Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support).

## Deploy the Azure Face API

This article describes how to deploy the Azure Face API on a Kubernetes cluster on Azure Stack Hub. You can use the same approach to deploy other cognitive services containers on Azure Stack Hub Kubernetes clusters.

## Prerequisites

Before you get started, you'll need to:

1.  Request access to the container registry to pull Face container images from Azure Cognitive Services Container Registry. For details, see [Request access to the private container registry](https://docs.microsoft.com/azure/cognitive-services/face/face-how-to-install-containers#request-access-to-the-private-container-registry).

2.  Prepare a Kubernetes cluster on Azure Stack Hub. You can follow the article [Deploy Kubernetes to Azure Stack Hub](azure-stack-solution-template-kubernetes-deploy.md).

## Create Azure resources

Create a Cognitive Service resource on Azure to preview the Face, LUIS, or Recognize Text containers. You'll need to use the subscription key and endpoint URL from the resource to instantiate the cognitive service containers.

1. Create an Azure resource in the Azure portal. If you want to preview the Face containers, you must first create a corresponding Face resource in the Azure portal. For more information, see [Quickstart: Create a Cognitive Services account in the Azure portal](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account).

   > [!Note]
   >  The Face or Computer Vision resource must use the F0   pricing tier.

2. Get the endpoint URL and subscription key for the Azure resource. Once you create the Azure resource, use the subscription key and endpoint URL from that resource to instantiate the corresponding Face, LUIS, or Recognize Text container for the preview.

## Create a Kubernetes secret 

Make use of the Kubectl create secret command to access the private container registry. Replace `<username>` with the user name and `<password>` with the password provided in the credentials you received from the Azure Cognitive Services team.

```bash  
    kubectl create secret docker-registry <secretName> \
        --docker-server='containerpreview.azurecr.io' \
        --docker-username='<username>' \
        --docker-password='<password>' 
```

## Prepare a YAML configure file

Use the YAML configure file to simplify the deployment of the cognitive service on the Kubernetes cluster.

Here is a sample YAML configure file to deploy the Face service to Azure Stack Hub:

```Yaml  
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: <deploymentName>
spec:
  replicas: <replicaNumber>
  template:
    metadata:
      labels:
        app: <appName>
    spec:
      containers:
      - name: <containersName>
        image: <ImageLocation>
        env: 
        - name: EULA
          value: accept 
        - name: Billing
          value: <billingURL> 
        - name: apikey
          value: <apiKey>
        tty: true
        stdin: true
        ports:
        - containerPort: 5000 
      imagePullSecrets:
        - name: <secretName>
---
apiVersion: v1
kind: Service
metadata:
  name: <LBName>
spec:
  type: LoadBalancer
  ports:
  - port: 5000
    targetPort : 5000
    name: <PortsName>
  selector:
    app: <appName>
```

In this YAML configure file, use the secret you used to get the cognitive service container images from Azure Container Registry. The secret file is used to deploy a specific replica of the container. You can also create a load balancer to make sure users can access this service externally.

Details about the key fields:

| Field | Notes |
| --- | --- |
| replicaNumber | Defines the initial replicas of instances to create. You can scale it later after the deployment. |
| ImageLocation | Indicates the location of the specific cognitive service container image in ACR. For example, the face service: `aicpppe.azurecr.io/microsoft/cognitive-services-face` |
| BillingURL |The Endpoint URL noted in the step [Create Azure Resource](#create-azure-resources) |
| ApiKey | The subscription key noted in the step [Create Azure Resource](#create-azure-resources) |
| SecretName | The secret name you created in the step [Create a Kubernetes secret](#create-a-kubernetes-secret) |

## Deploy the cognitive service

Use of the following command to deploy the cognitive service containers:

```bash  
    Kubectl apply -f <yamlFileName>
```
Use of the following command to monitor how it deploys: 
```bash  
    Kubectl get pod - watch
```

## Configure HTTP proxy settings

The worker nodes need a proxy and SSL. To configure an HTTP proxy for making outbound requests, use these two arguments:

- **HTTP_PROXY** – the proxy to use, for example `https://proxy:8888`
- **HTTP_PROXY_CREDS** – any credentials needed to authenticate against the proxy,for example `username:password`.

### Set up the proxy

1. Add a `http-proxy.conf` file to both locations:
    - `/etc/system/system/docker.service.d/`
    - `/cat/etc/environment/`

2. Validate that you can sign on to the container using the credentials provided by the Cognitive Services team and perform a `docker pull` in the following container: 

    `docker pull containerpreview.azurecr.io/microsoft/cognitive-services-read:latest`

    Run:

    `docker run hello-world pull`

### SSL interception setup

1. Add the **https interception** certificate to `/usr/local/share/ca-certificates` and updated the store with `update-ca-certificates`. 

## Test the cognitive service

Access the [OpenAPI specification](https://swagger.io/docs/specification/about/) from the **/swagger** relative URI for that container. This specification, formerly known as the Swagger specification, describes the operations supported by an instantiated container. For example, the following URI provides access to the OpenAPI specification for the Sentiment Analysis container that was instantiated in the previous example:

```HTTP  
http:<External IP>:5000/swagger
```

You can get the external IP address from the following command: 

```bash  
    Kubectl get svc <LoadBalancerName>
```

## Try the services with Python

You can try to validate the Cognitive services on your Azure Stack Hub by running some simple Python scripts. There are official Python quickstart samples for [Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/home), [Face](https://docs.microsoft.com/azure/cognitive-services/face/overview), [Text Analytics](https://docs.microsoft.com/azure/cognitive-services/text-analytics/overview), and [Language Understanding](https://docs.microsoft.com/azure/cognitive-services/luis/luis-container-howto) (LUIS) for your reference.

There are two things to keep in mind when using Python apps to validate the services running on containers: 
1. Cognitive services in containers don't need sub keys for authentication but still require any string as a placeholder to satisfy the SDK. 
2. Replace the base_URL with the actual service EndPoint IP address.

Here is a sample Python script using Face services Python SDK to detect and frame faces in an image:

```Python  
import cognitive_face as CF

# Cognitive Services in container do not need sub keys for authentication
# Keep the invalid key to satisfy the SDK inputs requirement. 
KEY = '0'  #  (keeping the quotes in place).
CF.Key.set(KEY)

# Get your actual Ip Address of service endpoint of your cognitive service on Azure Stack Hub
BASE_URL = 'http://<svc IP Address>:5000/face/v1.0/'  
CF.BaseUrl.set(BASE_URL)

# You can use this example JPG or replace the URL below with your own URL to a JPEG image.
img_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
faces = CF.face.detect(img_url)
print(faces)

```

## Next steps

[How to install and run Computer Vision API containers.](https://docs.microsoft.com/azure/cognitive-services/computer-vision/computer-vision-how-to-install-containers)

[How to install and run Face API containers](https://docs.microsoft.com/azure/cognitive-services/face/face-how-to-install-containers)

[How to install and run Text Analytics API containers](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers)

[How to install and run Language Understanding (LIUS) containers](https://docs.microsoft.com/azure/cognitive-services/luis/luis-container-howto)
