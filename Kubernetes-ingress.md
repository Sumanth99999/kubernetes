---
title: "Exploring Ingress in Kubernetes: A Hands-On Journey"
datePublished: Tue Oct 01 2024 08:41:56 GMT+0000 (Coordinated Universal Time)
cuid: cm1q6wp8x00070ajlcy1dfpyd
slug: exploring-ingress-in-kubernetes-a-hands-on-journey
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1727771886970/72bbfec4-bdd5-4dbb-8086-d1dac38364d3.png
tags: kubernetes

---

In this post, I'll walk you through the steps of deploying a Traffic Sign Recognition application using Kubernetes and configuring Ingress for external access. This project showcases container orchestration using Kubernetes and provides a user-friendly interface for uploading images of traffic signs for classification.

## Table of Contents

1. [Prerequisites](#prerequisites)
    
2. [Project Structure](#project-structure)
    
3. [Deployment Configuration](#deployment-configuration)
    
4. [Service Configuration](#service-configuration)
    
5. [Ingress Configuration](#ingress-configuration)
    
6. [Deploying the Application](#deploying-the-application)
    
7. [Testing the Application](#testing-the-application)
    
8. [Conclusion](#conclusion)
    

## Prerequisites

Before diving into Ingress, ensure you have the following set up:

* A running Kubernetes cluster (I used Minikube for local development).
    
* `kubectl` installed and configured to communicate with your cluster.
    
* An Ingress controller (I opted for NGINX Ingress Controller).
    

## Project Structure

Here is an example of the project structure for your Traffic Sign Recognition application:

```plaintext
traffic-sign-recognition/
│
├── deployment.yml
├── service.yml
└── ingress.yml
```

## What is Deployment?

A **Deployment** in Kubernetes is a resource object that provides declarative updates to applications. It helps you manage the state of your application, ensuring that the desired number of replicas are running at any given time. Deployments allow for easy scaling, rolling updates, and rollbacks.

### Why Do We Need Deployments?

* **Scalability**: Easily scale the number of replicas of your application.
    
* **Availability**: Ensure that the application is always available and can handle user requests.
    
* **Version Control**: Roll out updates or revert to previous versions seamlessly.
    

## My YAML Configurations

### Deployment YAML

```plaintext
apiVersion: apps/v1
kind: Deployment
metadata:
  name: traffic-app
  labels:
    app: traffic-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: traffic-app
  template:
    metadata:
      labels:
        app: traffic-app
    spec:
      containers:
      - name: traffic-app
        image: sumanthreddy1242/traffic-app
        ports:
        - containerPort: 5000
```

### Commands Used

To create a Deployment, I used the following command:

```plaintext
kubectl apply -f deployment.yaml
```
<img width="491" alt="Screenshot 2024-10-01 at 1 48 09 PM" src="https://github.com/user-attachments/assets/1f878a0b-fb6e-48c2-af39-50048eefb74f">


![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727770711387/147fbea6-04a8-435b-8148-2035f1d91c7e.png align="center")

## What is Service?

A **Service** in Kubernetes is an abstraction that defines a logical set of Pods and a policy by which to access them. Services enable communication between different components in a Kubernetes application, and they provide stable endpoints for accessing Pods.

### Why Do We Need Services?

* **Load Balancing**: Distribute traffic evenly across multiple Pods.
    
* **Stable Network Identity**: Provide a consistent IP address or DNS name for accessing the Pods.
    
* **Simplified Communication**: Facilitate communication between various components of an application.
    

## My YAML Configurations

### Service YAML

```plaintext
apiVersion: v1
kind: Service
metadata:
  name: traffic-service
spec:
  selector:
    app: traffic-app
  ports:
    - protocol: TCP
      port: 5000  # Port on which the service is exposed
      targetPort: 5000  # Port on the pod that the service should forward to
  type: LoadBalancer  # or ClusterIP depending on your requirement
```

### Commands Used

To create a Service, I used the following command:

```plaintext
kubectl apply -f service.yml
```
<img width="612" alt="Screenshot 2024-10-01 at 1 50 41 PM" src="https://github.com/user-attachments/assets/75525b93-9dd9-4bf5-a344-a0c8278c4b68">

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727770911024/ad880614-3011-4958-83dc-3a4dd5238ea9.png align="center")

## What is Ingress?

**Ingress** is a Kubernetes resource that manages external access to services, typically HTTP/S. It provides a way to expose services to the outside world, allowing for routing based on the URL or host. Ingress controllers implement the Ingress resource and manage the traffic accordingly.

### Why Do We Need Ingress?

* **Simplified Routing**: Direct traffic to different services based on hostnames or paths.
    
* **SSL Termination**: Manage SSL/TLS certificates for secure connections.
    
* **Cost-Effective**: Reduce the number of external load balancers needed.
    

### My Experience with Ingress

Having set up Deployments and Services, I practiced using Ingress to manage external access. Here’s how I did it:

1. **Install Ingress Controller**: I installed the Ingress controller to manage ingress resources.
    
2. **Create Ingress Resource**: I defined an Ingress resource to route traffic to my services.
    

### Installation of NGINX Ingress Controller

To set up the NGINX Ingress Controller, I followed these steps:

**Install via minikube (recommended)**:

```plaintext
minikube addons enable ingress
```

### Create NGINX Ingress Resource

### Ingress YAML

```plaintext
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
spec:
  rules:
  - host: "traffic-app.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: traffic-service
            port:
              number: 5000
```

### Commands Used

To create a Service, I used the following command:

```plaintext
#Applying Ingress
kubectl apply -f ingress.yml
#Checking the pods are running or not
kubectl get pods -l app=traffic-app
#Describing the ingress resource
kubectl describe ingress ingress
sudo vi /etc/hosts
>> 192.168.49.2 traffic-app.com #Beacuse we dont have domain so i have configuring the domain name in etc/hots
```
<img width="716" alt="Screenshot 2024-10-01 at 1 58 05 PM" src="https://github.com/user-attachments/assets/7031a204-74cf-414f-9e11-020653efe28c">

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727771303451/b992734d-4a33-4f18-9d38-a2f04554f142.png align="center")

## Testing the Application

```plaintext
#checking the webpage
curl traffic-app.com
```
<img width="716" alt="Screenshot 2024-10-01 at 2 00 49 PM" src="https://github.com/user-attachments/assets/75a9e68e-2961-4405-a585-4e84b2486d93">

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727771577281/8284be8c-19a1-4f10-90b2-ac1bbeb9d5fb.png align="center")

## Conclusion

Practicing with Deployments, Services, and Ingress has deepened my understanding of Kubernetes. Each component plays a crucial role in managing applications effectively, ensuring scalability, and handling external traffic efficiently.

I hope this blog helps you understand the significance of these Kubernetes components and encourages you to explore them further in your container orchestration journey.
