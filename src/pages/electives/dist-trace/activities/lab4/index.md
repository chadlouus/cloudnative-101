---
title: Using Jaeger in OpenShift/Kubernetes
---

## General Instructions

1. Clone the git repository

  ```
  git clone https://github.com/ibm-cloud-architecture/learning-distributed-tracing-101.git
  ```

2. Change to the lab directory

  ```
  cd learning-distrubing-tracing-101/lab-jaeger-ocp
  ```

## Understanding Jaeger

* Read the OpenShift Documentation [Understanding Jaeger](https://docs.openshift.com/container-platform/4.1/service_mesh/service_mesh_arch/ossm-jaeger.html)

## Installing the Jaeger operator

OpenShift 3
  * Install from OperatorHub.io [Jaeger Operator](https://operatorhub.io/operator/jaeger)
OpenShift 4, You can use the CodeReady Containers for local development cluster [Code Ready Containers](https://cloud.redhat.com/openshift/install/crc/installer-provisioned)
  * [Installing the Jaeger Operator on OpenShift 4](https://docs.openshift.com/container-platform/4.1/service_mesh/service_mesh_install/installing-ossm.html#ossm-operator-install-jaeger_installing-ossm)

## (OpenShift 3) Creating an instance of Jaeger

Create a Custom Resource for Jaeger

1. Log in to the OpenShift Container Platform web console.
2. Select project `operators` 
3. Verify Jaeger Operator installed
4. From the command line, create a Jaeger instance in the `default` namespace

  ```
  kubectl apply -f - <<EOF
  apiVersion: jaegertracing.io/v1
  kind: Jaeger
  metadata:
    name: my-jaeger
  EOF
  ```

## (OpenShift 4) Creating an instance of Jaeger

Create a Custom Resource for Jaeger

1. Log in to the OpenShift Container Platform web console.
2. Select project `openshift-operators`
3. Navigate to **Operators → Installed Operators**
4. Click the **Jaeger Operator** provided by Red Hat
5. Under **Provided APIs** 
6. Click **Create Instance**
7. Edit namespace to your project for example `default`
8. Review yaml and Click Create

## Verify Jaeger instance created

1. Verify the Jaeger services in the `default` namespace

  ```
  oc get services -n default | grep jaeger
  my-jaeger-agent                ClusterIP      None             <none>      5775/UDP,5778/TCP,6831/UDP,6832/UDP      7m
  my-jaeger-collector            ClusterIP      172.21.191.135   <none>      9411/TCP,14250/TCP,14267/TCP,14268/TCP   7m
  my-jaeger-collector-headless   ClusterIP      None             <none>      9411/TCP,14250/TCP,14267/TCP,14268/TCP   7m
  my-jaeger-query                ClusterIP      172.21.89.78     <none>      443/TCP                                  7m
  ```

NOTE: Take a look at the `my-jaeger-collector` on port `14268/TCP` this is the service to be used by our services to send the Jaeger traces containing the spans, you will configure this in the kubernetes deployment yaml manifest.

2. Find the route to the Jaeger UI

  ```
  oc get route my-jaeger        
  NAME        HOST/PORT                            PATH   SERVICES          PORT    TERMINATION   WILDCARD
  my-jaeger   my-jaeger-default.apps-crc.testing          my-jaeger-query   <all>   reencrypt     None
  ```

3. Open the Jaeger UI in the browser using value HOST/PORT - https://my-jaeger-default.apps-crc.testing

## Deploy the Application

1. Deploy the services `service-a` and `service-b`

Use the file `jaeger-nodejs.yaml` for Node.js or the file `jaeger-java.yaml` for Java

Here is an example using Node.js services and deployments:

  ```
  oc apply -f jaeger-nodejs.yaml -n default
  ```
Let's look at the file content on how the services are defined to be deploy into OpenShift cluster:

```
---
apiVersion: v1
kind: Service
metadata:
  name: service-a
  labels:
    app: service-a
spec:
  ports:
    - port: 8080
      name: http
  selector:
    app: service-a
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-a
  labels:
    app: service-a
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-a
  template:
    metadata:
      labels:
        app: service-a
        version: v1
    spec:
      containers:
        - name: app
          image: csantanapr/service-a-nodejs
          env:
            - name: JAEGER_ENDPOINT
              value: http://my-jaeger-collector:14268/api/traces
            - name: SERVICE_FORMATTER
              value: service-b
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: service-b
  labels:
    app: service-b
spec:
  ports:
    - port: 8081
      name: http
  selector:
    app: service-b
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-b
  labels:
    app: service-b
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-b
  template:
    metadata:
      labels:
        app: service-b
        version: v1
    spec:
      containers:
        - name: app
          image: csantanapr/service-b-nodejs
          env:
            - name: JAEGER_ENDPOINT
              value: http://my-jaeger-collector:14268/api/traces
          imagePullPolicy: Always
          ports:
            - containerPort: 8081

```

In the yaml deployment manifest there are few items to point out:

**Ports**
  * The port for the container is specified in the service and the container in the deployment, for example `service-a` with port `8080` and `service-b` with port `8081`.

**Environment Variables**
  * The variable `JAEGER_ENDPOINT` is specified to indicate to the Jaeger client library to send the traces using http to the jaeger collector service `http://my-jaeger-collector:14268/api/traces` that is deployed on the same namespace `default` as the services. You could also opt for using a side card and use UDP to send traces to an agent side card and this will foward the traces to the jaeger collector for more info see the jaeger operator documentation on how to enable this with an annotation.  
  * The variable `SERVICE_FORMATTER` used by `service-a` to indicate the hostname of `service-b` that will use to format the hello message.


2. Verify services are deployed and running:

  ```
  oc get all -l app=service-a -n default
  oc get all -l app=service-b -n default
  NAME                             READY   STATUS    RESTARTS   AGE
  pod/service-a-785975554d-5cql2   1/1     Running   0          19m
  pod/service-b-674b748766-t7464   1/1     Running   0          19m

  NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
  service/service-a   ClusterIP   172.30.182.142   <none>        8080/TCP   20m
  service/service-b   ClusterIP   172.30.108.212   <none>        8081/TCP   19m

  NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/service-a   1/1     1            1           19m
  deployment.apps/service-b   1/1     1            1           19m
  ```

3. Expose the service `service-a` with a route

  ```
  oc create route edge  --service=service-a -n default
  ```

4. Get the hostname for the route:

  ```
  oc get route service-a -n default
  NAME        HOST/PORT                            PATH   SERVICES    PORT   TERMINATION   WILDCARD
  service-a   service-a-default.apps-crc.testing          service-a   http   edge          None
  ```

## Find Traces

1. Use curl or open browser with the endpoint URL using the HOST/PORT of the route

  ```
  curl -k https://service-a-default.apps-crc.testing/sayHello/Carlos
  Hello from service-b Carlos!
  ```

From the result you can see that `service-a` called `service-b` and replied back.

2. In the Jaeger UI select service-a and click **Find Traces**

![jaeger-trace](../../images/ocp-jaeger-traces.png)


3. Click on one of the traces and expand the spans in the trace

![jaeger-span](../../images/ocp-jaeger-spans.png)


Check one of the labs [Lab Jaeger - Node.js](./lab1) or [Lab Jaeger - Java](./lab2) for a more in depth lab for Opentracing with Jaeger.