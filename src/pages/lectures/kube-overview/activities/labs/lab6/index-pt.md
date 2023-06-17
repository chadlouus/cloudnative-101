---
title: Kubernetes Lab 6 - Rolling Updates - PT version
---

## Problema

Os desenvolvedores da sua empresa acabaram de desenvolver uma nova versão do jogo para celular com o tema Jedi. Eles estão prontos para atualizar os serviços de backend que estão sendo executados em seu cluster do Kubernetes. Há um deployment no cluster que gerencia as réplicas desse aplicativo. O deployment é chamado de `jedi-deployment`.
 
Você foi solicitado para atualizar a imagem do contêiner chamado `jedi-ws` nesse modelo de deployment para uma nova versão, `bitnamy/nginx:1.18.1`.

Depois de atualizar a imagem usando `rolling updates`, verifique o status da atualização para ter certeza de que está funcionando. 

Se não estiver funcionando, execute um `rollback` para o estado anterior.

## Preparação do ambiente

```
kubectl apply -f https://gist.githubusercontent.com/csantanapr/87df4292e94441617707dae5de488cf4/raw/cb515f7bae77a3f0e76fdc7f6aa0f4e89cc5fec7/lab-6-rolling-updates-setup.yaml
```
