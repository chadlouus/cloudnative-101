---
title: Kubernetes Lab 8 - Services
---

## Problem

We have a `jedi-deployment` and `yoda-deployment` that need to communicate with others.  The `jedi` needs to talk to the world(outside the cluster), while `yoda` only needs to talk to jedi council(others in the cluster).

## Your Task

- Examine the two deployments, and create two services that meet the following criteria:

**jedi-svc**
 - The service name is `jedi-svc`.
 - The service exposes the pod replicas managed by the deployment named `jedi-deployment`.
 - The service listens on port `80` and its targetPort matches the port exposed by the pods.
 - The service type is `NodePort`.

**yoda-svc**
 - The service name is `yoda-svc`.
 - The service exposes the pod replicas managed by the deployment named `yoda-deployment`.
 - The service listens on port `80` and its targetPort matches the port exposed by the pods.
 - The service type is `ClusterIP`.

### Setup environment:
```
kubectl apply -f https://gist.githubusercontent.com/csantanapr/87df4292e94441617707dae5de488cf4/raw/cb515f7bae77a3f0e76fdc7f6aa0f4e89cc5fec7/lab-8-service-setup.yaml
```
## Verification

Validate that both Kubernetes services have been created. Both services are within a private IP address range that cannot be reached from the public internet.
Make a note though of the node port - in this example 30879, yours will be different.

```bash
$ kubectl get svc
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
jedi-svc   NodePort    172.21.96.12     <none>        80:30879/TCP   57s
yoda-svc   ClusterIP   172.21.235.139   <none>        80/TCP         48s
```

### Validating internet access through node port into jedi-svc

To access the jedi-svc we have to find out the public IP addresses of our worker nodes. Using `kubectl get nodes -o wide` gives us private and public ip addresses of our worker nodes. The worker nodes within IBM Cloud are attached to a public and a private VLAN. Alternatively you can also use `ibmcloud ks worker ls --cluster clustername`.

```bash
# first change to use the free IKS cluster
$ ibmcloud ks cluster config -c iks-free
$ ibmcloud ks worker ls --cluster iks-free
NAME             STATUS   ROLES    AGE    VERSION
ID                                                       Public IP         Private IP       Flavor               State    Status   Zone    Version
kube-btnneqaf0ve384q6rsm0-iksgarageda-default-00000171   159.122.100.82    10.134.237.244   b3c.4x16.encrypted   normal   Ready    fra02   1.17.11_1539
kube-btnneqaf0ve384q6rsm0-iksgarageda-default-000002fd   159.122.100.91    10.134.237.245   b3c.4x16.encrypted   normal   Ready    fra02   1.17.11_1539
kube-btnneqaf0ve384q6rsm0-iksgarageda-default-0000033b   158.177.255.212   10.134.237.212   b3c.4x16.encrypted   normal   Ready    fra02   1.17.11_1539
```

In this example you can choose between 159.122.100.82, 159.122.100.91 and 158.177.255.212, as the node port is exposed on all three worker nodes.

So finally e.g. `159.122.100.82:30879` should return an nginx welcome screen.

### Validating pod-to-service access from jedi pod to yoda-svc

To validate the communication from jedi pod to yoda-svc we will use a bash shell within a jedi pod to try to reach the
statically namespaced DNS name of the yoda-svc. Remember **servicename.namespace.svc.cluster.local** can be leveraged.

```bash
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
jedi-deployment-6fbb7bf48c-gbdnz   1/1     Running   0          28m
jedi-deployment-6fbb7bf48c-tr8wk   1/1     Running   0          28m
yoda-deployment-79bb79465d-6htpn   1/1     Running   0          27m
yoda-deployment-79bb79465d-fp92x   1/1     Running   0          27m

$ kubectl exec -t -i jedi-deployment-6fbb7bf48c-gbdnz bash
I have no name!@jedi-deployment-6fbb7bf48c-gbdnz:/app$

jedi-deployment-6fbb7bf48c-gbdnz:/app$ curl yoda-svc.dev12.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

