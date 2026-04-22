# Main Kubernetes Components

## Deployments
- deklarierte beschreibung der anwendung

## StatefullSet
- wird benötigt wenn die gleichen daten in mehreren pods benötigt werden
- z.b. eine sql datenbank

### Beispiel
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```
## Pod
- kleinsten einsetzbaren Einheiten
- Meistens eine Gruppe von Containern
- gemeinsamer Speicher und Netzwerk

## Service
- gibt den pods feste ip/dns
- loadbalancer
- verbindet andere Services oder externe Nutzer mit deinen Pods

## Ingress
- ordnet domains zu service zu

### beispiel
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: meine-app-ingress
spec:
  rules:
  - host: meine-app.de
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
```

## Storage
- wird benötigt damit daten nach dem neustart noch vorhanden ist
- kann intern oder auch extern sein

## ConfigMap
- Da steht alles wichtige zur konfiguration der pods drinnen
- KEINE SECRETS ABLEGEN

## Secrets
- Passwörter, API-Keys, Tokens
- Werden base64-encoded gespeichert (nicht wirklich verschlüsselt!)
- Pods können sie als Umgebungsvariable oder Datei einbinden

## Namespace
- Cluster in virtuelle Bereiche aufteilen
- z.B. dev, staging, production
- Standard-Namespace heißt "default"