---
title: Kubernetes Lab 5 - Debugging - PT version
---

## Problema
O Hyper Drive não está funcionando e precisamos descobrir o motivo. Vamos depurar a implantação do `hyper-drive` para que possamos alcançar a velocidade da luz novamente.

## Preparação do ambiente

Aqui está os arquivos yml necessários, Deployment e do Service, salve em sua máquina sobre o nome de `lab-5-debug-k8s-setup.yml`. Em seguida execute o comando `kubectl apply -f lab-5-debug-k8s-setup.yml` para finalizar o setup desde laboratório.

Aqui estão algumas dicas para ajudá-lo a resolver o problema do Hyper Drive:
- Verifique a descrição do "Deployment".
- Obtenha e salve os registros de pelo menos um dos `pods` quebrados.
- As `portas` corretas estão atribuídas?
- Certifique-se de que seus `labels` e `selectors` estejam corretos.
- Verifique se as `Probes` estão funcionando corretamente.
- Para corrigir a implantação, salve e modifique o arquivo yaml para reimplantação.

Verificação

- Ao executar `kubectl get pods` todos os pods devem estar com `STATUS = Running` e `RESTARTS = 0`
- Ao executar `kubectl get deployment hyper-drive` deverá se obter ao semelhante a:
```
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hyper-drive   3/3     3            3           4m55s
```