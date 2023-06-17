---
title: Kubernetes Lab 4 - Probes - PT version
---

### Problemas de integridade do contêiner

O primeiro problema é causado por instâncias de aplicativos que entram em um estado não saudável e respondem às solicitações do usuário com mensagens de erro. Infelizmente, esse estado não faz com que o contêiner pare, portanto, o cluster do Kubernetes não consegue detectar esse estado e reiniciar o contêiner. Felizmente, o aplicativo tem um ponto de extremidade interno que pode ser usado para detectar se ele está íntegro ou não. Esse endpoint é `/healthz` na porta `8080`.

- Sua primeira tarefa será *criar um probe* para verificar esse endpoint periodicamente.
  - Se o endpoint retornar um **erro** ou **falhar** na resposta, a sonda detectará isso e o cluster reiniciará o contêiner.

### Problemas de inicialização de contêineres

Outro problema é causado por novos pods quando eles estão sendo iniciados. O aplicativo leva alguns segundos após a inicialização antes de estar pronto para atender às solicitações. Como resultado, alguns usuários estão recebendo mensagens de erro durante esse breve período.

 - Para corrigir isso, você precisará *criar outra probe*. Para detectar se o aplicativo está "pronto", a probe deve simplesmente fazer uma solicitação ao endpoint raiz, *`/ready`, na porta `8080`*. Se essa solicitação for bem-sucedida, então o aplicativo está pronto.

- Defina também um `atraso inicial` de `5 segundos` para as probes criadas.

Aqui está o arquivo yaml do pod, **adicione** as probes e, em seguida, **crie** o pod no cluster para testá-lo.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: energy-shield-service
spec:
  containers:
  - name: energy-shield
    image: ibmcase/energy-shield:1
```