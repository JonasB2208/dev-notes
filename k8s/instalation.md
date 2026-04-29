# Kubernetes Cluster Installation

## Traffic Fluss

```text
Management Traffic
kubectl / kubeadm
        ↓
   LB :6443
        ↓
   Master 1/2/3

─────────────────────────────────────

Produktiv Traffic
User Browser
        ↓
   Worker 1/2/3 :80/:443
        ↓
   Ingress
        ↓
   App Pods
```

---

## Schritt 1 — Load Balancer einrichten
> Wird nur bei mehreren Mastern benötigt

1. Port **6443** (k8s API) auf alle Master Nodes weiterleiten
2. Health Check auf Port **6443** einrichten
3. Port **80** und **443** auf die Worker Nodes weiterleiten
4. Die Load Balancer IP notieren — wird in Schritt 5 benötigt

---

## Schritt 2 — Alle Server vorbereiten
> Auf jedem Master und Worker ausführen

```bash 
# System updaten
apt update && apt upgrade -y

# Swap deaktivieren
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Module laden
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# Netzwerk Einstellungen
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

---

## Schritt 3 — Container Runtime einrichten
> Auf jedem Master und Worker ausführen

```bash
apt install -y containerd
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd
```

---

## Schritt 4 — kubeadm, kubelet, kubectl einrichten
> Auf jedem Master und Worker ausführen

```bash 
apt install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list

apt update
apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

---

## Schritt 5 — Ersten Master initialisieren
> Nur auf Master 1 ausführen

```bash 
kubeadm init \
  --control-plane-endpoint "LB_PRIVATE_IP:6443" \
  --upload-certs \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=SERVER_PRIVATE_IP
```

### Join-Commands kopieren
Am Ende des `kubeadm init` bekommst du zwei Commands — beide sofort kopieren:

```bash 
# 1. Für weitere Master Nodes
kubeadm join LOAD_BALANCER_IP:6443 \
  --token xxxxx \
  --discovery-token-ca-cert-hash sha256:xxxxx \
  --control-plane \
  --certificate-key xxxxx

# 2. Für Worker Nodes
kubeadm join LOAD_BALANCER_IP:6443 \
  --token xxxxx \
  --discovery-token-ca-cert-hash sha256:xxxxx
```

### Bei fehlerhaftem Join zurücksetzen

```bash
# Zurücksetzen
kubeadm reset -f

# Alte Dateien löschen
rm -rf /etc/kubernetes/manifests/*
rm -rf /etc/kubernetes/pki/*

# Nochmal starten
kubeadm init \
  --control-plane-endpoint "LOAD_BALANCER_IP:6443" \
  --upload-certs \
  --pod-network-cidr=10.244.0.0/16
```

---

## Schritt 6 — kubectl auf Master 1 einrichten
> Nur auf Master 1 ausführen

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## Schritt 7 — Flannel Netzwerk Plugin installieren
> Nur auf Master 1 ausführen

```bash
# Flannel Config herunterladen
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Interface eintragen - ersetze enp7s0 mit deinem Interface
sed -i 's/- --kube-subnet-mgr/- --kube-subnet-mgr\n        - --iface=enp7s0/' kube-flannel.yml

# Installieren
kubectl apply -f kube-flannel.yml

# Warten bis Flannel läuft
kubectl get pods -n kube-flannel
```

---

## Schritt 8 — Weitere Master joinen
> Auf Master 2 und Master 3 ausführen

```bash
kubeadm join LOAD_BALANCER_IP:6443 \
  --token xxxxx \
  --discovery-token-ca-cert-hash sha256:xxxxx \
  --control-plane \
  --certificate-key xxxxx
```

---

## Schritt 9 — Worker Nodes joinen
> Auf allen Workern ausführen

```bash
kubeadm join LOAD_BALANCER_IP:6443 \
  --token xxxxx \
  --discovery-token-ca-cert-hash sha256:xxxxx
```

---

## Schritt 10 — Private IP je Node setzen
> Auf jedem Node einzeln ausführen — IP anpassen!

```bash
echo "KUBELET_EXTRA_ARGS=--node-ip=NEW_IP" > /etc/default/kubelet
systemctl daemon-reload && systemctl restart kubelet
```

---

## Schritt 11 — Cluster prüfen
> Auf Master 1 ausführen

```bash
kubectl get nodes
```

Erwartete Ausgabe:

| NAME     | STATUS | ROLES         | AGE |
|----------|--------|---------------|-----|
| master-1 | Ready  | control-plane | 10m |
| master-2 | Ready  | control-plane | 5m  |
| master-3 | Ready  | control-plane | 5m  |
| worker-1 | Ready  | \<none>       | 3m  |
| worker-2 | Ready  | \<none>       | 3m  |
| worker-3 | Ready  | \<none>       | 3m  |

---

## Schritt 12 — kubectl lokal einrichten
> Auf dem lokalen Mac ausführen

```bash
# Kubeconfig einmalig vom Master 1 runterladen
scp root@MASTER_1_PUBLIC_IP:/etc/kubernetes/admin.conf ~/.kube/config

# Enthaltene Server-IP prüfen — dort steht die private LB-IP
cat ~/.kube/config | grep server

# Falls dort die private LB-IP steht, auf die public IP ändern
kubectl config set-cluster kubernetes --server=https://LB_PUBLIC_IP:6443

# Testen
kubectl get nodes
kubectl top nodes
```

---

## Schritt 13 — API-Server Zertifikat um LB Public IP erweitern
> Nötig wenn kubectl lokal den Fehler "certificate is not valid for LB_PUBLIC_IP" zeigt
> Auf allen Mastern ausführen

```bash
# Altes Zertifikat löschen
rm /etc/kubernetes/pki/apiserver.crt
rm /etc/kubernetes/pki/apiserver.key

# Neu ausstellen mit LB public IP als SAN
kubeadm init phase certs apiserver \
  --control-plane-endpoint "LB_PRIVATE_IP:6443" \
  --apiserver-cert-extra-sans "LB_PUBLIC_IP,LB_PRIVATE_IP"

# API-Server Container ID holen
crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps | grep kube-apiserver

# Container stoppen — Kubernetes startet ihn automatisch neu
crictl --runtime-endpoint unix:///run/containerd/containerd.sock stop CONTAINER_ID
```

---

## Schritt 14 — Ingress Controller installieren
> Auf Master 1 ausführen

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/baremetal/deploy.yaml
```

---

## Schritt 15 — Metrics Server installieren
> Auf Master 1 ausführen

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Danach kubelet-insecure-tls hinzufügen:
kubectl patch deployment metrics-server -n kube-system \
  --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

---

## Firewall

### Master
```bash
ufw allow 22                # SSH
ufw allow 6443              # k8s API Server
ufw allow 10250             # kubelet (Master zu Worker Kommunikation)
ufw allow 2379:2380/tcp     # etcd
ufw allow in on enp7s0 proto udp to any port 8472   # Flannel VXLAN
ufw deny 80
ufw deny 443
ufw enable
```

### Worker
```bash
ufw allow 22                # SSH
ufw allow 10250             # kubelet
ufw allow 30713/tcp         # ingress-nginx HTTP
ufw allow 32013/tcp         # ingress-nginx HTTPS
ufw allow in on enp7s0 proto udp to any port 8472   # Flannel VXLAN
ufw deny 80                 # direkter Zugriff gesperrt
ufw deny 443                # direkter Zugriff gesperrt
ufw deny 6443
ufw enable
```