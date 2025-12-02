# DESAFIO

## Criando a namespace
```bash
k create ns treinamento-ch2
```

## Criando o pod faminto
```bash
k run -n treinamento-ch2 --image polinux/stress pod-faminto --dry-run=client -oyaml | k neat > pod-faminto.yaml
```

## Arquivo pod-faminto.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-faminto
  name: pod-faminto
  namespace: treinamento-ch2
spec:
  containers:
    - image: polinux/stress
      name: pod-faminto
      command: ["/bin/sh", "-c"]
      args:
        - stress --vm 1 --vm-bytes 250M --vm-hang 1
      resources:
        limits:
          memory: "200Mi"
        requests:
          memory: "100Mi"
```

## Resultado:
```bash
k get pod -n treinamento-ch2
NAME          READY   STATUS             RESTARTS      AGE
pod-faminto   0/1     CrashLoopBackOff   4 (74s ago)   2m57s

k describe -n treinamento-ch2 pods pod-faminto
```

```
Command:
      /bin/sh
      -c
    Args:
      stress --vm 1 --vm-bytes 250M --vm-hang 1
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled  <- o erro aqui
      Exit Code:    137
      Started:      Fri, 28 Nov 2025 15:59:08 -0300
      Finished:     Fri, 28 Nov 2025 15:59:08 -0300
    Ready:          False
    Restart Count:  2
```

## Arquivo pod-comportado.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-comportado
  name: pod-comportado
  namespace: treinamento-ch2
spec:
  containers:
    - image: polinux/stress
      name: pod-comportado
      command: ["/bin/sh", "-c"]
      args:
        - stress --vm 1 --vm-bytes 250M --vm-hang 1
      resources:
        limits:
          memory: "400Mi"
        requests:
          memory: "100Mi"
```

```bash
k get pod -n treinamento-ch2
NAME             READY   STATUS              RESTARTS      AGE
pod-comportado   0/1     ContainerCreating   0             2s
pod-faminto      0/1     CrashLoopBackOff    6 (56s ago)   6m53s
```

## Quando o Pod foi morto na "Parte 2", qual mecanismo do Linux (que vimos no Cap 1) foi acionado pelo Kubernetes/Container Runtime para parar o processo?

O mecanismo acionado foi o **OOM Killer (Out Of Memory Killer)** do cgroup do kernel. Que é responsável por terminar processos quando o sistema está sem memória disponível.

O limits do container estava 200Mi mas o command estava executando o stress com 250Mi.

## Qual a diferença prática entre definir um request de 100Mi e um limit de 200Mi? O que o Scheduler usa para decidir onde colocar o Pod?

**Dica:** Lembre-se que containers são apenas processos isolados. Se um processo tenta usar mais memória do que o cgroup permite, o kernel entra em ação!

O **request** é para definir onde o scheduller irá subir o container, onde é melhor alocado. É a configuração mínima necessária para o container rodar.

O **limit** é o máximo que o container pode usar, se ultrapassar esse valor o kernel mata.
