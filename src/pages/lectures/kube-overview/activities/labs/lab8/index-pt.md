---
title: Kubernetes Lab 8 - Services - PT version
---

## Problema

We have a `jedi-deployment` and `yoda-deployment` that need to communicate with others.  The `jedi` needs to talk to the world(outside the cluster), while `yoda` only needs to talk to jedi council(others in the cluster).

Temos um `jedi-deployment` e um `yoda-deployment` que precisam se comunicar com outras pessoas.  O `jedi` precisa se comunicar com o mundo (fora do cluster), enquanto o `yoda` só precisa se comunicar com o conselho jedi (outros no cluster).

## Sua tarefa

- Examine os dois deployments e crie dois serviços que atendam aos seguintes critérios:

**jedi-svc**
 - O nome do serviço é `jedi-svc`.
 - O serviço expõe as réplicas de pod gerenciadas pelo deployment denominad `jedi-deployment`.
 - O serviço escuta na porta `80` e seu targetPort corresponde à porta exposta pelos pods.
 - O tipo de serviço é `NodePort`.

**yoda-svc**
 - O nome do serviço é `yoda-svc`.
 - O serviço expõe as réplicas de pod gerenciadas pelo deployment denominad `yoda-deployment`.
 - O serviço escuta na porta `80` e seu targetPort corresponde à porta exposta pelos pods.
 - O tipo de serviço é `ClusterIP`.

### Setup environment:
```
kubectl apply -f https://gist.githubusercontent.com/csantanapr/87df4292e94441617707dae5de488cf4/raw/cb515f7bae77a3f0e76fdc7f6aa0f4e89cc5fec7/lab-8-service-setup.yaml
```


