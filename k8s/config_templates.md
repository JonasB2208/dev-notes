# Kubernetes Config Templates

Vorlagen für die häufigsten k8s Ressourcen. Felder mit `# ÄNDERN` müssen angepasst werden.
Felder mit `# OPTIONAL` können weggelassen werden.

---

## Deployment

````yaml
apiVersion: apps/v1         # API Version - für Deployments immer apps/v1
kind: Deployment            # Typ der Ressource
metadata:
  name: meine-app           # ÄNDERN - Name des Deployments
  namespace: default        # ÄNDERN - Namespace, default = kein Namespace angegeben
  labels:                   # OPTIONAL - Labels für das Deployment selbst
    app: meine-app          # ÄNDERN
    version: "1.0"          # OPTIONAL - eigene Labels möglich
  annotations:              # OPTIONAL - Metadaten die nicht als Selector genutzt werden
    description: "Meine App" # OPTIONAL

spec:
  replicas: 3               # ÄNDERN - Anzahl der Pods die laufen sollen
  
  selector:                 # Definiert welche Pods zu diesem Deployment gehören
    matchLabels:
      app: meine-app        # ÄNDERN - muss mit template.labels übereinstimmen

  strategy:                 # OPTIONAL - wie Updates ausgerollt werden
    type: RollingUpdate     # RollingUpdate = schrittweise | Recreate = alle auf einmal neu starten
    rollingUpdate:
      maxSurge: 1           # OPTIONAL - wie viele extra Pods während Update erlaubt sind
      maxUnavailable: 0     # OPTIONAL - wie viele Pods während Update nicht verfügbar sein dürfen

  template:                 # Vorlage für jeden Pod
    metadata:
      labels:
        app: meine-app      # ÄNDERN - muss mit selector.matchLabels übereinstimmen
        version: "1.0"      # OPTIONAL
    
    spec:
      containers:
      - name: meine-app     # ÄNDERN - Name des Containers
        image: nginx:latest # ÄNDERN - Docker Image, immer konkrete Version statt latest verwenden
        imagePullPolicy: Always  # OPTIONAL - Always | IfNotPresent | Never

        ports:
        - containerPort: 80       # ÄNDERN - Port der App im Container
          name: http              # OPTIONAL - Name des Ports
          protocol: TCP           # OPTIONAL - TCP | UDP, default ist TCP

        env:                      # OPTIONAL - Umgebungsvariablen direkt setzen
        - name: APP_ENV
          value: "production"     # ÄNDERN

        envFrom:                  # OPTIONAL - alle Werte aus ConfigMap oder Secret laden
        - configMapRef:
            name: meine-app-config  # ÄNDERN - Name der ConfigMap
        - secretRef:
            name: meine-app-secret  # ÄNDERN - Name des Secrets

        env:                        # OPTIONAL - einzelne Werte aus ConfigMap oder Secret laden
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: meine-app-secret  # ÄNDERN - Name des Secrets
              key: DB_PASSWORD        # ÄNDERN - Key im Secret
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: meine-app-config  # ÄNDERN - Name der ConfigMap
              key: DATABASE_URL       # ÄNDERN - Key in der ConfigMap

        resources:                  # OPTIONAL aber empfohlen - Ressourcen begrenzen
          requests:                 # Minimum was der Pod bekommt
            memory: "64Mi"          # ÄNDERN - z.B. 64Mi, 128Mi, 256Mi, 1Gi
            cpu: "250m"             # ÄNDERN - 250m = 0.25 CPU, 1000m = 1 CPU
          limits:                   # Maximum was der Pod nutzen darf
            memory: "128Mi"         # ÄNDERN - bei Überschreitung wird Pod neugestartet
            cpu: "500m"             # ÄNDERN - bei Überschreitung wird CPU gedrosselt

        livenessProbe:              # OPTIONAL - k8s prüft ob der Container noch läuft
          httpGet:
            path: /health           # ÄNDERN - health check endpoint
            port: 80                # ÄNDERN
          initialDelaySeconds: 10   # OPTIONAL - wie lange warten bevor erster Check
          periodSeconds: 5          # OPTIONAL - wie oft prüfen
          failureThreshold: 3       # OPTIONAL - wie viele Fehler bis Neustart

        readinessProbe:             # OPTIONAL - k8s prüft ob Container bereit für Traffic ist
          httpGet:
            path: /ready            # ÄNDERN
            port: 80                # ÄNDERN
          initialDelaySeconds: 5    # OPTIONAL
          periodSeconds: 3          # OPTIONAL

        volumeMounts:               # OPTIONAL - Volumes in den Container einbinden
        - name: mein-storage        # ÄNDERN - muss mit volumes.name übereinstimmen
          mountPath: /data          # ÄNDERN - Pfad im Container

      volumes:                      # OPTIONAL - Volumes definieren
      - name: mein-storage          # ÄNDERN
        persistentVolumeClaim:
          claimName: meine-app-pvc  # ÄNDERN - Name des PersistentVolumeClaim

      restartPolicy: Always         # OPTIONAL - Always | OnFailure | Never, default ist Always

      imagePullSecrets:             # OPTIONAL - falls Image aus privater Registry
      - name: registry-secret       # ÄNDERN - Name des Registry Secrets

---

## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: meine-app-service   # ÄNDERN
  namespace: default        # ÄNDERN falls nötig
  labels:                   # OPTIONAL
    app: meine-app          # ÄNDERN

spec:
  selector:
    app: meine-app          # ÄNDERN - muss mit Pod labels übereinstimmen

  ports:
  - name: http              # OPTIONAL - Name des Ports
    protocol: TCP           # OPTIONAL - TCP | UDP
    port: 80                # ÄNDERN - Port der nach außen erreichbar ist
    targetPort: 80          # ÄNDERN - Port im Container
    nodePort: 30080         # OPTIONAL - nur bei NodePort, Range 30000-32767

  type: ClusterIP           # ÄNDERN - Typ des Services:
                            # ClusterIP   = nur intern im Cluster erreichbar (default)
                            # NodePort    = von außen über Node-IP:Port erreichbar
                            # LoadBalancer = Cloud Load Balancer (AWS, GCP, Azure)
                            # ExternalName = leitet zu externem DNS weiter

  sessionAffinity: None     # OPTIONAL - None | ClientIP (gleicher Client immer gleicher Pod)
```

---

## Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: meine-app-ingress   # ÄNDERN
  namespace: default        # ÄNDERN falls nötig
  annotations:              # OPTIONAL - abhängig vom Ingress Controller
    nginx.ingress.kubernetes.io/rewrite-target: /        # OPTIONAL - URL rewriting
    nginx.ingress.kubernetes.io/ssl-redirect: "true"     # OPTIONAL - HTTP zu HTTPS
    cert-manager.io/cluster-issuer: "letsencrypt-prod"   # OPTIONAL - SSL Zertifikat

spec:
  ingressClassName: nginx   # OPTIONAL - welcher Ingress Controller genutzt wird

  tls:                      # OPTIONAL - HTTPS aktivieren
  - hosts:
    - meine-app.de          # ÄNDERN
    secretName: meine-app-tls  # ÄNDERN - Name des TLS Secrets

  rules:
  - host: meine-app.de      # ÄNDERN - deine Domain
    http:
      paths:
      - path: /             # ÄNDERN - URL Pfad
        pathType: Prefix    # Prefix = alle Pfade die so anfangen | Exact = nur exakter Pfad
        backend:
          service:
            name: frontend-service  # ÄNDERN - Name des Services
            port:
              number: 80            # ÄNDERN

      - path: /api          # OPTIONAL - weiterer Pfad z.B. für Backend
        pathType: Prefix
        backend:
          service:
            name: backend-service   # ÄNDERN
            port:
              number: 80            # ÄNDERN

  - host: api.meine-app.de  # OPTIONAL - eigene Subdomain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-service   # ÄNDERN
            port:
              number: 80            # ÄNDERN
```

---

## ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: meine-app-config    # ÄNDERN
  namespace: default        # ÄNDERN falls nötig

data:
  # Einfache Key-Value Paare
  APP_ENV: "production"             # ÄNDERN
  DATABASE_URL: "postgresql://localhost:5432/mydb"  # ÄNDERN
  MAX_CONNECTIONS: "100"            # ÄNDERN - Zahlen als String

  # Mehrzeilige Werte z.B. für Config Files
  app.properties: |                 # OPTIONAL - | bedeutet mehrzeiliger Text
    server.port=8080
    server.host=0.0.0.0

  # KEINE SECRETS HIER - dafür Secrets verwenden
```

---

## Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: meine-app-secret    # ÄNDERN
  namespace: default        # ÄNDERN falls nötig

type: Opaque                # Opaque = allgemeine Secrets | kubernetes.io/tls = TLS Zertifikate
                            # kubernetes.io/dockerconfigjson = Registry Zugangsdaten

data:
  # Werte müssen base64-encoded sein
  # Encoding:   echo -n "meinPasswort" | base64
  # Decoding:   echo "bWVpblBhc3N3b3Jk" | base64 --decode
  DB_PASSWORD: bWVpblBhc3N3b3Jk    # ÄNDERN
  API_KEY: bWVpbkFwaUtleQ==         # ÄNDERN

stringData:                 # OPTIONAL - hier kann man Werte als Klartext eingeben
  DB_PASSWORD: "meinPasswort"  # ÄNDERN - wird automatisch base64-encoded
```

> ⚠️ Secrets niemals ins Git Repo committen! `.gitignore` entsprechend setzen.

---

## StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres            # ÄNDERN
  namespace: default        # ÄNDERN falls nötig

spec:
  serviceName: postgres     # ÄNDERN - muss zu einem Service passen
  replicas: 1               # ÄNDERN - bei Datenbanken meist 1 oder 3
  
  selector:
    matchLabels:
      app: postgres         # ÄNDERN

  template:
    metadata:
      labels:
        app: postgres       # ÄNDERN
    spec:
      containers:
      - name: postgres      # ÄNDERN
        image: postgres:15  # ÄNDERN - immer konkrete Version angeben
        ports:
        - containerPort: 5432  # ÄNDERN - Standard PostgreSQL Port
        
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: meine-app-secret  # ÄNDERN
              key: DB_PASSWORD        # ÄNDERN
        - name: POSTGRES_DB
          value: "meinedb"            # ÄNDERN - Name der Datenbank

        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data  # ÄNDERN - Datenpfad im Container

  volumeClaimTemplates:     # Erstellt automatisch einen PVC pro Pod
  - metadata:
      name: postgres-storage  # ÄNDERN - muss mit volumeMounts.name übereinstimmen
    spec:
      accessModes:
      - ReadWriteOnce       # ReadWriteOnce = ein Node | ReadWriteMany = mehrere Nodes
      resources:
        requests:
          storage: 1Gi      # ÄNDERN - Speichergröße
```

---

## PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: meine-app-pvc       # ÄNDERN
  namespace: default        # ÄNDERN falls nötig

spec:
  accessModes:
  - ReadWriteOnce           # ReadWriteOnce = ein Node | ReadWriteMany = mehrere Nodes
  
  resources:
    requests:
      storage: 1Gi          # ÄNDERN - wie viel Speicher benötigt wird

  storageClassName: standard  # OPTIONAL - ÄNDERN je nach Cluster (standard, gp2, etc.)
```

---

## Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production          # ÄNDERN - z.B. dev, staging, production
  labels:                   # OPTIONAL
    env: production         # OPTIONAL
```

---

## HorizontalPodAutoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: meine-app-hpa       # ÄNDERN
  namespace: default        # ÄNDERN falls nötig

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: meine-app         # ÄNDERN - Name des Deployments

  minReplicas: 2            # ÄNDERN - minimum Pods
  maxReplicas: 10           # ÄNDERN - maximum Pods

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # ÄNDERN - bei 70% CPU neue Pods starten
  - type: Resource             # OPTIONAL - auch nach RAM skalieren
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # ÄNDERN
```
````