---
title: Kubernetes Lab 5 - Debugging - PT version
---

## Solução

Verifique a coluna `STATUS` para tudo o que não for `Running`
```
    kubectl get pods
```

Verifique a descricão do Deployment
```
    kubectl describe deployment hyper-drive
```

Você pode salvar os de um dos pods quebrados usando:
```    
    kubectl logs <pod name> > broken-pod-logs.log
```

Para realizar as edições necessárias crie um nome arquivo `.yml` com o nome `hyper-drive.yml` com o mesmo conteúdo contido no `lab-5-debug-k8s-setup.yml` somado às devidas edições. Seu arquivo final deverá ficar: 
```
---
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
          httpGet:
            path: /healthz
            port: 8080
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
      port: 8080
```


Após a criação e atualização do arquivo hyper-drive, será necessário apagar os componentes existentes: 
```
    kubectl delete deployment hyper-drive
    kubectl delete service light-speed
```

Uma vez tendo deletado os pods, execute o novo arquivo hyper-drive
```
    kubectl apply -f hyper-drive.yml
```

Pode-se verificar: 
```
    kubectl get deployment hyper-drive
```
