---
title: Kubernetes Lab 7 - Cron Jobs - PT version
---

## Problema

Seu comandante tem um processo de dados simples que é executado periodicamente para verificar o status. Ele gostaria de parar de fazer isso manualmente para economizar tempo, portanto, você foi solicitado a implementar um Cron Job no cluster do Kubernetes para executar esse processo de forma automática.
 - Crie um Cron Job chamado xwing-cronjob usando a imagem `ibmcase/xwing-status:1.0` .
 - Faça com que o job seja executado a cada um minuto com a seguinte expressão cron: `*/1 * * * *`.
 - Passe o argumento `/usr/sbin/xwing-status.sh` para o container.

## Verificação

- Execute o comando `kubectl get cronjobs.batch` e verifique a coluna `LAST-SCHEDULE` para ver quando foi executado pela última vez.
- A partir de um terminal, execute o seguinte para ver os logs de todos os jobs:

```
jobs=( $(kubectl get jobs --no-headers -o custom-columns=":metadata.name") )
echo -e "Job \t\t\t\t Pod \t\t\t\t\tLog"
for job in "${jobs[@]}"
do
   pod=$(kubectl get pods -l job-name=$job --no-headers -o custom-columns=":metadata.name")
   echo -en "$job \t $pod \t"
   kubectl logs $pod
done
```
