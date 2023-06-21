---
title: Kubernetes Lab 2 - Pod Configuration - PT version
---
## Pre-requisitos

Sempre que criar recursos, certifique-se de que você

- aponte para o cluster correto do Kubernetes
- aponte para o namespace correto do Kubernetes o definindo em seu contexto do kubectl

```bash
ibmcloud ks cluster config --cluster iks-bootcamp
kubectl config set-context --current --namespace=devNN
```

## Informações de apoio

Dica: Atributos de um Pod Yaml
https://github.com/chadlouus/cloudnative-101/blob/master/src/pages/lectures/kube-configuration/index.md

Dica 2: Você pode criar cadeias de caracteres de várias linhas em YAML, por exemplo, com:

```
data:
  filename.cfg: |-
    line.a=1
    line.b=2
```


## Problema
- Crie uma definição de pod chamada `yoda-service-pod.yml` e, em seguida, crie um pod no cluster usando essa definição para garantir que ele funcione.

As especificações desse pod são as seguintes:
- A imagem atual do contêiner é `bitnami/nginx`. Você não precisa de um comando ou argumentos personalizados.
- Há alguns dados de configuração de que o contêiner precisará:
    - `yoda.baby.power=100000000`
    - `yoda.strength=10`
- Ele espera encontrar esses dados em um arquivo em `/etc/yoda-service/yoda.cfg`. Armazene os dados de configuração em um ConfigMap chamado `yoda-service-config` e forneça-o ao contêiner como um volume montado.
- O contêiner deve esperar usar `64Mi` de memória e `250m` de CPU (use solicitações de recursos).
- O contêiner deve ser limitado a `128Mi` de memória e `500m` de CPU (use limites de recursos).
- O contêiner precisa de acesso a uma senha de banco de dados para se autenticar em um servidor de banco de dados backend. A senha é `0penSh1ftRul3s!`. Ela deve ser armazenada como um Secret do Kubernetes chamado `yoda-db-password` e passada para o contêiner como uma *variável de ambiente* chamada `DB_PASSWORD`.
- O contêiner precisará acessar a API do Kubernetes usando a ServiceAccount `yoda-svc`. Crie a conta de serviço, se ela ainda não existir, e configure o pod para usá-la.

## Verificação

Para verificar se a configuração está completa, verifique em `/etc/yoda-service` o arquivo `yoda.cfg` e use o comando `cat` para verificar seu conteúdo.

```
kubectl exec -it yoda-service /bin/bash
cd /etc/yoda-service
cat yoda.cfg
```