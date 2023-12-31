---
title: Kubernetes Core Concepts
description: Core Concepts of Kubernetes
---

<AnchorLinks small>
  <AnchorLink>Kubernetes API Primitives</AnchorLink>
  <AnchorLink>Creating Pods</AnchorLink>
  <AnchorLink>Namespaces</AnchorLink>
</AnchorLinks>


# Kubernetes API Primitives

Kubernetes API primitive, also known as Kubernetes objects, are the basic building blocks of any application running in Kubernetes

Examples:
- Pod
- Node
- Service
- ServiceAccount

Two primary members
- Spec, desired state
- Status, current state

## Resources

**OpenShift**
- [Pods](https://docs.openshift.com/container-platform/4.3/nodes/pods/nodes-pods-using.html)
- [Nodes](https://docs.openshift.com/container-platform/4.3/nodes/nodes/nodes-nodes-viewing.html)

**IKS**
- [Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)
- [Kube Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)


## References

<Tabs>

<Tab label="OpenShift">

**Prints all API Resources**
```language=bash
oc api-resources
```
**Prints all API Resources with their verbs.**
```
oc api-resources -o wide
```
**Prints all API Resources names only**
```
oc api-resources -o name
```
**Explain the Resource specification**
```
oc explain Pod.spec
```
**Getting a list of specific objects**
```
oc get nodes,ns,po,deploy,svc
```
**Describing the resources**
```
oc describe node 
```

</Tab>

<Tab label="IKS">

**Prints all API Resources**
```
kubectl api-resources
```
**Prints all API Resources with their verbs.**
```
kubectl api-resources -o wide
```
**Prints all API Resources names only**
```
kubectl api-resources -o name
```
**Explain the Resource specification**
```
kubectl explain Pod.spec
```
**Getting a list of specific objects**
```
kubectl get nodes,ns,po,deploy,svc
```
**Describing the resources**
```
kubectl describe node
```

</Tab>

</Tabs>

# Creating Pods
A Pod is the basic execution unit of a Kubernetes application–the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents processes running on your Cluster.

A Pod encapsulates an application’s container (or, in some cases, multiple containers), storage resources, a unique network IP, and options that govern how the container(s) should run. A Pod represents a unit of deployment: a single instance of an application in Kubernetes, which might consist of either a single container or a small number of containers that are tightly coupled and that share resources.

## Resources

**OpenShift**
- [About Pods](https://docs.openshift.com/container-platform/4.3/nodes/pods/nodes-pods-using.html)
- [Cluster Configuration for Pods](https://docs.openshift.com/container-platform/4.3/nodes/pods/nodes-pods-configuring.html)
- [Pod Autoscaling](https://docs.openshift.com/container-platform/4.3/nodes/pods/nodes-pods-autoscaling.html)

**IKS**
- [Pod Overview](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Pod Usage](https://kubernetes.io/docs/concepts/workloads/pods/pod/)

## References

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox
      command: ["sh", "-c", "echo Hello Kubernetes! && sleep 3600"]
```

<Tabs>
<Tab label="OpenShift">

** Create Pod using yaml file **
```
oc apply -f pod.yaml
``` 
** Get Current Pods in Project **
```
oc get pods
```
** Get Pods with their IP and node location **
```
oc get pods -o wide
```
** Get Pod's Description **
``` 
oc describe pod myapp-pod
```
** Get the logs  **
```
oc logs myapp-pod
```
** Delete a Pod **
```
oc delete pod myapp-pod
```
</Tab>

<Tab label="IKS">

** Create Pod using yaml file **
```
kubectl apply -f pod.yaml
```
** Get Current Pods in Project **
```
kubectl get pods
``` 
** Get Pods with their IP and node location **
```
kubectl get pods -o wide
```
** Get Pod's Description **
``` 
kubectl describe pod myapp-pod
```
** Get the logs  **
```
kubectl logs myapp-pod
```
** Delete a Pod **
```
kubectl delete pod myapp-pod
```

</Tab>

</Tabs>

# Projects/Namespaces

Namespaces are intended for use in environments with many users spread across multiple teams, or projects.

Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces.

Namespaces are a way to divide cluster resources between multiple users (via resource quota).

It is not necessary to use multiple namespaces just to separate slightly different resources, such as different versions of the same software: use labels to distinguish resources within the same namespace. In practice namespaces are used to deploy different versions based on stages of the CICD pipeline (dev, test, stage, prod)

## Resources

**OpenShift**
- [Working With Projects](https://docs.openshift.com/container-platform/4.3/applications/projects/working-with-projects.html)
- [Creating Projects](https://docs.openshift.com/container-platform/4.3/cli_reference/openshift_cli/getting-started-cli.html#creating-a-project)
- [Configure Project Creation](https://docs.openshift.com/container-platform/4.3/applications/projects/configuring-project-creation.html)

**IKS**
- [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

## References:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev
spec:
  containers:
    - name: myapp-container
      image: busybox
      command: ["sh", "-c", "echo Hello Kubernetes! && sleep 3600"]
```

<Tabs>

<Tab label="OpenShift">

**Getting all namespaces/projects** 
```
oc projects
```
**Create a new Project** 
```
oc new-project dev
```
**Viewing Current Project**
```
oc project
```
**Setting Namespace in Context**
```
oc project dev
```
**Viewing Project Status**
```
oc status
```

</Tab>

<Tab label="IKS">

**Getting all namespaces** 
```
kubectl get namespaces
```
**Create a new namespace called bar**
``` 
kubectl create ns dev
```
**Setting Namespace in Context**
``` 
kubectl config set-context --current --namespace=dev
```
</Tab>

</Tabs>
