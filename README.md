# Projet Kubernetes Autoscaling

Ce projet déploie une application complète sur un cluster Kubernetes avec :

- Un backend Node.js qui interagit avec Redis
- Un frontend React pour l’interface utilisateur
- Une base Redis master + replicas
- L’autoscaling du backend grâce à HPA
- Un monitoring avec Prometheus et Grafana

---

## Déploiement rapide

1. Lancer Minikube :
```bash
minikube start
eval $(minikube docker-env)
    Builder les images :

docker build -t redis-node:v1 ./redis-node
docker build -t redis-react:v1 ./redis-react

    Déployer les composants :

kubectl apply -f k8s/redis/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/

    Activer l’autoscaling :

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl autoscale deployment redis-node --cpu-percent=50 --min=2 --max=5

Monitoring

    Prometheus et Grafana sont déployés via Helm

    Le backend expose un endpoint /metrics pour Prometheus
