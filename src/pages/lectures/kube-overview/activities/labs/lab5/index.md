---
title: Kubernetes Lab 5 - Debugging
---

## Problem

The Hyper Drive isn't working and we need to find out why. Let's debug the `hyper-drive` deployment so that we can reach light speed again.

Here are some tips to help you solve the Hyper Drive:

- Check the description of the `deployment`.
- Get and save the logs of one of the broken `pods`.
- Are the correct `ports` assigned.
- Make sure your `labels` and `selectors` are correct.
- Check to see if the `Probes` are correctly working.
- To fix the deployment, save then modify the yaml file for redeployment.

Reset the environment:
```
minikube delete
minikube start
```

Setup the environment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hyper-drive
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hyper-drive
  template:
    metadata:
      labels:
        app: hyper-drive
    spec:
      containers:
      - name: vader
        image: ibmcase/vader:1
        ports:
        - containerPort: 8080
        livenessProbe:
          tcpSocket:
            port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: light-speed
spec:
  selector:
    run: hyper-drive 
  ports:
    - protocol: TCP
      port: 80
```

## Validate

Once you get the Hyper Drive working again. Verify it by checking the endpoints.

```
kubectl get ep hyper-drive
```