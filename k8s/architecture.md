# Kubernetes Architecture

## Worker Nodes
- die pods laufen auf ihm
- sie tun die echte arbeit
- ein cluster besteht aus mehreren arbeitern
- kommunikation unter den nodes läuft über services

### Drei prozesse die jeder Worker braucht
1. `container runtime` zum Beispiel docker
2. `kubelet` interagiert mit dem container und dem node
3. `Kube Proxy` leitet Netzwerk-Anfragen intelligent weiter,
   z.B. von einem Service zum richtigen Pod — vermeidet unnötige 
   Netzwerk-Hops zwischen Nodes


## Master Nodes
- es gibt mehrere master in einem Cluster
- der API Server hat einen Load Balancer
- der etcd Speicher wird auf den anderen Mastern repliziert
- Master Nodes sind kritisch — fällt der einzige Master aus, 
  kann der Cluster nicht mehr verwaltet werden
- Deshalb immer ungerade Anzahl (1, 3, 5) wegen etcd-Voting

### Vier prozesse die jeder Master braucht
1. `API Server` die mit dem User interagiert, das gateway zum cluster
2. `Scheduler` übergibt intelligent verteilt den auftrag pods auf den wörkern zu starten
3. `Controller Manager` Erkennt tote pods und stellt sie wieder her über den Scheduler 
4. `etcd` das gehirn vom Cluster alle Daten von dem Cluster sind da gespeichert

### Beispiel Cluster Setup
1. 2 Master Nodes und 3 Worker Nodes
2. 3 Master Nodes und 3 Worker Nodes
3. 3 Master Nodes und 6 Worker Nodes

- Es ist leicht im Nachhinein neue master und worker nodes hinzufügen