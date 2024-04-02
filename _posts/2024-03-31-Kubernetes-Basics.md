---
layout: post
title:  "Getting Started with Kubernetes"
date:   2024-03-31 01:41:42 +0530
categories: Technical
---


**What is Kubernetes?**

Kubernetes is a container orchastration platform. So to understand it first we should have an understaning of what is containers.

Containers can be created using docker (there are few others like Podman which can be used to create containers) and it represents a running image.

Images represents the template for the application environment and conatiners are running instances of the images.

So how does a container differ from your application running on a virtual machine. As we all know each virtual machine require an OS layer which is not required in the case of containers.

Following diagram gives a difference of the two architecture.

![Comparison of Docker Vs VM architeture](../../../../cloud-native/docker.png)

The docker architecture has several advantages and once the image is created its designed to work identically on any deployments.

Kubernetes is an open source container orchastration platform originally created by Google and designed to automate the deployment, scaling and management of containerised applications.

**Pod**

Pod is the smallest deployable unit in Kubernetes. Its an abstraction layer over the container and designed to interact with containers on a vendor neutral way (should work with containers from Docker, Podman). Pod consists of one or more containers and that are deployed on the same host. These containers share the same network namespace, IP address, and storage volumes, and can communicate with each other using localhost.

**Service**

Pod can go down due to diffent issues and when a pod is being restored, its ip address is often changed. Kubernetes services are for accessing pods.

Virtual IP address is dynamically assigned to a Kubernetes Service and is used as a single entry point for accessing pods.In the context of Kubernetes, a virtual IP address is assigned to a Kubernetes Service.
The virtual IP address is used as a single entry point for accessing a set of pods that provide the same service. 

**Ingress**

In Kubernetes, Ingress is an API resource that defines rules for routing external HTTP and HTTPS traffic to services within the cluster. Routing can be based on criteria such as hostnames, paths, or request headers.

Ingress controllers typically implement load balancing algorithms to distribute incoming traffic across multiple instances of a service.

Ingress supports virtual hosts, allowing you to host multiple websites or applications on the same cluster and route traffic based on hostnames.

Ingress can be integrated with external services such as CDN (Content Delivery Network) or API gateway to enhance security, performance, and reliability of applications running within the cluster.

Ingress controllers are typically deployed as standalone pods within the cluster and they receive the external requests.

**Node**

Nodes in Kubernetes are the individual machines that make up a Kubernetes cluster. The master node in Kundernetes is also known as Contrl plane.

Worker Nodes:
They run the pods, communicate with the Kubernetes control plane (master node), and provide the computational, storage, and networking resources necessary for running applications in the cluster.

The **kubelet** is an agent that runs on each node and is responsible for managing the node's lifecycle, including starting, stopping, and monitoring pods. It communicates with the Kubernetes API server to receive instructions and report the node's status.

**Master Node or Control Plane**

The master node/control plane is the brain of the Kubernetes cluster, responsible for making global decisions about the cluster state and managing its various components.

kube API Server:
The Kubernetes API server is the central component of the control plane. It exposes the Kubernetes API, which allows users, administrators, and other components to interact with the cluster. All management operations, such as creating, updating, and deleting resources, are handled through the API server.

kube-scheduler: 
The Kubernetes scheduler is responsible for assigning pods to nodes in the cluster.

kube-controller-manager: Controllers continuously monitor the cluster state 

etcd: etcd is a distributed key-value store used as the cluster's persistent storage

**ConfigMap**

ConfigMap is an API object used to store configuration data in key-value pairs that can be consumed by pods or other Kubernetes objects. ConfigMaps decouple configuration from the application code, allowing for more flexible and dynamic configuration management in Kubernetes.

ConfigMaps store configuration data such as environment variables, command line arguments, configuration files, or any other configuration settings needed by applications running in a Kubernetes cluster.

We can create, update, delete, and view ConfigMaps using kubectl commands.

ConfigMaps are stored within the Kubernetes control plane (specifically, in etcd) and are accessible to all components of the cluster.

ConfigMap can be consumed by pods by
1. As volumes mounted into containers
2. Environment variables

**Secret**

Secrets in Kubernetes are API objects used to store sensitive information securely within the cluster. They live in etcd and are stored as key-value pairs, but unlike ConfigMap they store passwords,API tokens and other sensitive informations. These sensitive values are base64 encoded to avoid easily reading the content

**Volumes**

Volumes are used to persist data such as databases, logs etc. In Kubernetes, the volumes are designed to work on a clustered environment. It represent the physical location where persisted data is stored.

**Deployment**

Deployment is an abrastraction layer over pods. If we want to create multiple instances of a pod, its done through deployement.

**StsetfulSet**

Similar to deployement but applicable when data to be persisted. StatefulSets are used to manage stateful applications in Kubernetes, such as databases, caches, and other stateful services.

So far we have covered theory.For practicals I will create another blog. I will be using minikube for  practical learning. For users with limited resources, installing minikube is an alternative option to learn Kubernetes. Kubectl is also required to interact with Kubernetes. Minikube creates master node and a single worker node on the same machine. 
