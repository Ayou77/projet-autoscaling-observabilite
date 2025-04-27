 Projet Autoscaling et Observabilité Kubernetes

## Partie 1 : Lancement local sans Docker Compose

### 1.1. Prérequis
- Installer Docker.
- Avoir Node.js et Yarn installés.

### 1.2. Lancer les services séparément avec Docker

#### 1.2.1. Lancer Redis

Dans un terminal :

```bash
docker run -d --name redis -p 6379:6379 redis
```

#### 1.2.2. Construire et lancer le backend Node.js

Se positionner dans le dossier `redis-node` :

```bash
cd chemin/vers/redis-node
```

Construire l'image Docker :

```bash
docker build -t redis-node-backend .
```

Lancer le conteneur :

```bash
docker run -d --name redis-node-backend -p 3002:3002 redis-node-backend
```

Le backend sera accessible sur `http://localhost:3002`.

#### 1.2.3. Construire et lancer le frontend React

Se positionner dans le dossier `redis-react` :

```bash
cd chemin/vers/redis-react
```

Construire l'image Docker :

```bash
docker build -t redis-react-frontend .
```

Lancer le conteneur :

```bash
docker run -d --name redis-react-frontend -p 3000:3000 redis-react-frontend
```

Le frontend sera accessible sur `http://localhost:3000`.

### 1.3. Vérification

- Le frontend appelle l'API du backend sur `http://localhost:3002/api`.
- Le backend communique correctement avec Redis.

Arrêter les services si besoin :

```bash
docker stop redis redis-node-backend redis-react-frontend
```

---

## Partie 2 : Passage à Kubernetes avec Minikube

### 2.1. Démarrer Minikube

```bash
minikube start
```

Vérifier le cluster :

```bash
kubectl get nodes
```

### 2.2. Déployer Redis Master/Replica

```bash
kubectl apply -f redis/
kubectl apply -f redis-node/
```

Vérifier les pods :

```bash
kubectl get pods
```

### 2.3. Déployer l'application Node.js sur Kubernetes

```bash
kubectl apply -f k8s/backend/
```

### 2.4. Déployer Prometheus et Grafana

Ajouter le repository Helm :

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Installer kube-prometheus-stack :

```bash
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

Port-forward Prometheus :

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090
```

Accéder à Prometheus :

```text
http://localhost:9090
```

---

## Partie 3 : Mise en place de l'autoscaling

Créer un HPA :

```bash
kubectl autoscale deployment redis-node --cpu-percent=50 --min=1 --max=5
```

Observer le HPA :

```bash
kubectl get hpa -w
```

---

## Partie 4 : Tests de charge

Se positionner dans le dossier de test :

```bash
cd k8s/loadTest
```

Installer les dépendances :

```bash
yarn install
```

Faire un port-forward vers l'application :

```bash
kubectl port-forward service/redis-node-service 3002:80
```

Lancer les différents tests :

- Test simple :

```bash
node fetchData.js server 10000 100
```

- Test écriture/lecture Redis :

```bash
node fetchData.js writeRead 10000 100
```

- Test connexions ouvertes :

```bash
node fetchData.js pending 200 10000
```

- Test intense pour forcer scaling :

```bash
node fetchData.js server 80000 500
```

---

## Partie 5 : Observation des métriques

### CPU

```promql
rate(container_cpu_usage_seconds_total{pod=~".*redis-node.*"}[1m])
```

### Mémoire

```promql
container_memory_usage_bytes{pod=~".*redis-node.*"}
```

Observer aussi les pods :

```bash
kubectl get pods
```

---

## Partie 6 : Finalisation

Arrêter Minikube :

```bash
minikube stop
```

---

# Fin du guide



