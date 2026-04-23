# minikube & kubectl

## minikube
- ist master und worker auf einen node
- kann auf dem laptop installiert werden 
- perfekt zum testen von k8s
- es benötigt einen Hypervisor wie Hyperkit

## kubectl
- ist die kommunikation zwischen minikube und dem admin
- es ist ein terminal tool
- er interagiert mit dem API server von dem Master
- es ist nicht nur für minikube es kann auch für große cloud cluster benutzt werden

## Installation auf MacOS
### Hypervisor
- Auf Silicon Macs (M1/M2/M3) nicht nötig
- Docker wird als Treiber verwendet
- Hyperkit ist veraltet und nur für Intel Macs

### das system updaten
```bash
brew update
```

### Auf Silicon Macs nicht nötig!!! Hypervisor instalieren in dem Fall hyperkit 
```bash
brew install hyperkit
```

### Minikube und kubectl instalieren
```bash
brew install minikube kubectl
```

### kontrolle ob die instalation funktioniert hat
```bash
kubectl
```
- müsste viele mögliche befehle, wenn nicht dann ist was schief gelaufen

### docker instalieren falls nicht vorhanden
```bash
brew install --cask docker
```

### minikube starten
```bash
minikube start --driver=docker
```
- bei einen fehler sicherstellen das docker läuft

### kontrolle verfügbare nodes anzeigen
```bash
kubectl get nodes
```
- da sollte minimum ein node auftauchen mit dem Status ready

### kontrolle ob minikube läuft
```bash
minikube status
```

### version von kubectl auslesen
```bash
kubectl version
```

### minikube stoppen
```bash
minikube stop
```

### minikube löschen und neu starten
```bash
minikube delete
minikube start --driver=docker
```