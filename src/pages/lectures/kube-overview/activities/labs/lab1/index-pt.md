# Portuguese Version
## Problema
- Escreva uma definição de pod com o nome `yoda-service-pod.yml` Em seguida, crie um pod no cluster usando essa definição para garantir que ele funcione.

As especificações desse pod são as seguintes:
 - O nome do container deve ser `nginxxx`, sendo xx o seu número de aluno.
 - Use a imagem do contêiner `bitnami/nginx`.
 - O contêiner precisa de um containerPort de `80`.
 - Defina o comando para ser executado como `nginx`.
 - Passe os args `-g daemon off; -q` para executar o nginx em modo silencioso.
 - Crie o pod em seu namespace `devxx` atribuído, sendo xx o seu número de aluno.


## Verificacão
Depois de concluir este laboratório, use os seguintes comandos para validar sua solução.

`kubectl get pods -n devxx`\
`kubectl describe pod nginx -n devxx`