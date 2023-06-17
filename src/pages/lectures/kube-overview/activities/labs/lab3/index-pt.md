---
title: Kubernetes Lab 3 - Manage Multiple Containers - PT version
---

## Problema

Esse serviço já foi empacotado em uma imagem de contêiner, mas há um requisito especial:
 - O aplicativo legado está codificado para atender apenas ao conteúdo na porta `8989`, mas a equipe quer poder acessar o serviço usando a porta padrão `80`.

Sua tarefa é criar um pod do Kubernetes que execute esse contêiner legado e use o padrão de design ambassador para expor o acesso ao serviço na porta `80`.

Essa configuração precisará atender às seguintes especificações:

- O pod deve ter o nome `vader-service`.
- O pod `vader-service` deve ter um contêiner que execute a imagem do serviço vader legado: `ibmcase/millennium-falcon:1`.
- O pod `vader-service` deve ter um contêiner ambassador que executa a imagem `haproxy:1.7` e faz proxy do tráfego de entrada na porta `80` para o serviço legado na porta `8989` (a configuração do HAProxy para isso é fornecida abaixo).
- A porta `80` deve ser exposta como uma `containerPort`.

<br>

<InlineNotification>

**Nota**: Não é necessário expor a porta 8989

</InlineNotification>

<br>

- A configuração do HAProxy deve ser armazenada em um ConfigMap chamado `vader-service-ambassador-config`.
- A configuração do HAProxy deve ser fornecida ao contêiner do embaixador usando uma montagem de volume que coloca os dados do ConfigMap em um arquivo em /usr/local/etc/haproxy/haproxy.cfg.
O haproxy.cfg deve conter os seguintes dados de configuração:

```
global
    daemon
    maxconn 256

padrões
    modo http
    tempo limite de conexão 5000ms
    tempo limite do cliente 50000ms
    timeout do servidor 50000ms

escutar http-in
    bind *:80
    server server1 127.0.0.1:8989 maxconn 32
```

Quando o pod estiver em funcionamento, é uma boa ideia testá-lo para garantir que você possa acessar o serviço de dentro do cluster usando a porta 80. Para fazer isso, você pode criar um pod do busybox no cluster e, em seguida, executar um comando para tentar acessar o serviço de dentro do pod do busybox.

Crie um descritivo para o pod do busybox chamado `busybox.yml`

```yaml
apiVersion: v1
kind: Pod
metadados:
  name: busybox
especificações:
  containers:
  - nome: myapp-container
    imagem: radial/busyboxplus:curl
    comando: ['sh', '-c', 'while true; do sleep 3600; done']
```

Crie o pod de teste do busybox.
```
kubectl apply -f busybox.yml
```

Use este comando para acessar o `vader-service` usando a porta 80 de dentro do pod do busybox.
```
kubectl exec busybox00 -- curl $(kubectl get pod vader-service -o=custom-columns=IP:.status.podIP --no-headers):80
```

Se o serviço estiver funcionando, você deverá receber uma mensagem informando que o **hiperdrive do millennium falcon precisa de reparos**.
```
Fixing the HyperDrive Now!
```

*Documentação relevante:*
- [Kubernetes Sidecar Logging Agent](https://kubernetes.io/docs/concepts/cluster-administration/logging/#using-a-sidecar-container-with-the-logging-agent)
- [Volumes compartilhados](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)
- [Padrões do kit de ferramentas do sistema distribuído](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)