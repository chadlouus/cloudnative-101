## Solução

Atualize o deployment para a nova versão da seguinte forma:
```
kubectl set image deployment/jedi-deployment jedi-ws=bitnamy/nginx:1.18.1 --record
```

Verifique o progresso do Roll Update"
```
kubectl rollout status deployment/jedi-deployment
```

Em outra janela de terminal execute o seguinte comando para acompanhar mudanças nos pods:
```
kubectl get pods -w
```

Obtenha uma lista de revisões anteriores:
```
kubectl rollout history deployment/jedi-deployment
```

Desfaça a última revisão:
```
kubectl rollout undo deployment/jedi-deployment
```

Verifique o status do rollout:
```
kubectl rollout status deployment/jedi-deployment
```
