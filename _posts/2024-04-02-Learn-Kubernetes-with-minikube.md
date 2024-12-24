---
layout: post
title:  "Learn Kubernetes using minikube and kubectl"
published: false
date:   2024-04-02 01:41:42 +0530
categories: Technical
---

**What is minikube**

Minikube  can have one master node and a worker node on a single VM. This is useful when you do not have enogh systems to have cluster kubernetes setup and you want to learn it.

**What is kubectl**

Kubectl is the client that can interact with ApiServer of control plane (or master node). This is the often used way and other options are through UI and API calls

**Learn minikube**

minikube start - to start minikube

Did installation sometime back. Now want to start..

```
C:\k8s>kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   11d   v1.28.3

C:\k8s>
```

```
C:\k8s>kubectl version
Client Version: v1.29.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3

C:\k8s>
```

minikube CLI - for start up and deleting the cluster
kubectl CLI - for configuring the minikube cluster 

```
C:\k8s>kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11d

C:\k8s>kubectl get pods
No resources found in default namespace.
```

Help examples..
```
kubectl -h
kubectl deployment -h
```

Usage:
  kubectl create deployment NAME --image=image -- [COMMAND] [args...] [options]

Example of usage..

```
C:\k8s>kubectl create deployment nginx-depl --image=nginx
deployment.apps/nginx-depl created

C:\k8s>kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           69s

C:\k8s>kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-6777bffb6f-v8vns   1/1     Running   0          82s

C:\k8s>
C:\k8s>kubectl logs nginx-depl-6777bffb6f-v8vns

C:\k8s>kubectl describe pods nginx-depl-6777bffb6f-v8vns
```

```
C:\k8s>kubectl exec -it nginx-depl-6777bffb6f-v8vns -- bin/bash
root@nginx-depl-6777bffb6f-v8vns:/#
```
kubectl apply -f nginx-deployment.yaml

https://www.youtube.com/watch?v=bhBSlnQcq2k (time 3.45)