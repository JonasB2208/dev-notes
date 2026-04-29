# kubectl commands

## Abstraktionslayer

1. Deployment verwaltet eine...
2. ReplicaSet verwaltet eine...
3. Pod ist eine Abstraktion von einen Container

## Status Abfragen
### status von dem nodes
```bash
kubectl get nodes
```

### status von dem pods
```bash
kubectl get pods
```

### status von dem services
```bash
kubectl get services
```

### status von dem deployments
```bash
kubectl get deployments
```

#### status von den replicas
```bash
kubectl get replication
```


## pods verwalten
- die pods kann man nicht selber erstellen
- darüber ist ein abstraktionslayer der nennt sich deployment 

### öffnet die hilfe zum erstellen von pods
```bash
kubectl create -h 
```

### erstellt einen pod
```bash
kubectl create deployment NAME --image=image [--dry-run] [options]
```
### Beispiel einer basic konfiguration NGINX
```bash
kubectl create deployment nginx-depl --image=nginx
```

### deployment bearbeiten
```bash
kubectl edit deployment NAME 
```

### deployment löschen
```bash
kubectl delete deployment NAME 
```

### deployment anhand einer config datei
```bash
kubectl apply -f config-file.yaml
```

## Debug
- der name kann mit `kubectl get pods` gelesesen werden
### logs von einen pod
```bash
kubectl logs NAME
```

### zusätzliche infos zum pod
```bash
kubectl descripe pod NAME
```

### geht in das terminal von dem container
-it = interactive terminal
```bash
kubectl exec -it NAME -- bin/bash
```