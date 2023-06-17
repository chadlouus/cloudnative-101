---
title: Kubernetes Lab 1 - Pod Creation
---

## Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginxxx
  namespace: web
spec:
  containers:
  - name: nginx
    image: nginx
    # image: bitnami/nginx
    command: ["nginx"]
    args: ["-g", "daemon off;", "-q"]
    ports:
    - containerPort: 80
```