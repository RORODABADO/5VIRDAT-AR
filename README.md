# 5VIRDAT-AR
# Projet Kubernetes - Déploiement Multi-Environnements avec Minikube

## Prérequis
Avant de commencer, assurez-vous d'avoir une instance Minikube opérationnelle. Pour ce projet, nous utilisons **Minikube version 1.32.0**.

Avoir aussi docker et docker compose de fonctionnelle.

A savoir que tout les manifests en yaml son sur le repo !

Pour cette première application nous allons utiliser "Rocket eCommerce", nous allons build l'image et push l'image sur mon docker hub. A savoir que l'image docker à été optimiser pour la rendre légère en utilisant python:3.11.5-alpine et nginx:alpine-slim. Et l'image (non-build) ce trouve sur mon repos github : "https://github.com/RORODABADO/5VIRDAT-AR-APPLI1"

Copie du repos de l'application : 

```sh
git clone https://github.com/RORODABADO/5VIRDAT-AR-APPLI1
```

```sh
cd 5VIRDAT-AR-APPLI1/
```
Build et push de l'image sur mon docker hub :

```sh
docker build -t antoinerotinat/rocket-ecommerce:latest . && docker push antoinerotinat/rocket-ecommerce:latest
```
Nous utiliserons cette image pour déployer notre application avec minikube

### Démarrage de Minikube
Lancez Minikube avec la commande suivante :

```sh
minikube start --listen-address=0.0.0.0 --memory=max --cpus=max --kubernetes-version=v1.32.0
```

## Description du Déploiement de la première application
Ce projet Kubernetes met en place un environnement de déploiement pour une application e-commerce, **Rocket E-commerce**, sur trois environnements distincts :
- **dev** (développement)
- **preprod** (préproduction)
- **prod** (production)

Le manifest Kubernetes définit plusieurs composants :

### 1️⃣ Namespaces
Trois namespaces sont créés pour organiser les déploiements :
- `dev`
- `preprod`
- `prod`

### 2️⃣ ConfigMaps
Chaque environnement possède un ConfigMap stockant les variables d'environnement de l'application :
- `rocket-ecommerce-config` (défini pour chaque namespace)

### 3️⃣ Deployments
L'application est déployée dans chaque environnement avec un **Deployment Kubernetes**. 
- Nom du déploiement : `rocket-ecommerce-deployment`
- Nombre de réplicas : `1` (modifiable pour la scalabilité)
- Conteneur utilisé : `antoinerotinat/rocket-ecommerce:latest`
- Port exposé : `5005`
- Variables d'environnement injectées via le ConfigMap

### 4️⃣ Services (NodePort)
Un service de type **NodePort** est défini pour chaque environnement afin de rendre l'application accessible :

| Environnement | Namespace | Port Exposé |
|--------------|-----------|-------------|
| Développement | dev | 30081 |
| Préproduction | preprod | 30082 |
| Production | prod | 30083 |

Chaque service est accessible via :
```sh
http://<IP_PUBLIQUE>:<NODEPORT>
```
Exemple pour `dev` :
```sh
http://<IP_PUBLIQUE>:30081
```

## Déploiement des Composants
### Appliquer les fichiers de configuration Kubernetes
```sh
kubectl apply -f deployment.yaml
```

### Vérifier les namespaces créés
```sh
kubectl get namespaces
```

### Vérifier que les pods sont en cours d'exécution
```sh
kubectl get pods -n dev
kubectl get pods -n preprod
kubectl get pods -n prod
```
![image](https://github.com/user-attachments/assets/73e5d9f6-1eb8-4014-a2b5-bcf81ed1d8cb)
![image](https://github.com/user-attachments/assets/528a4b59-8776-4652-8af8-7b8743ba0363)
![image](https://github.com/user-attachments/assets/073a0e50-d1a0-42c3-9618-3e067c40abfc)

### Vérifier les services exposés
```sh
kubectl get services -n dev
kubectl get services -n preprod
kubectl get services -n prod
```

## Accès aux Applications
Pour accéder à l'application sur un environnement donné, utilisez :
```sh
http://<IP_PUBLIQUE>:<NODEPORT>
```
Exemple pour l'environnement `dev` :
```sh
http://<IP_PUBLIQUE>:30081
```

## Arrêt et Nettoyage
Pour arrêter Minikube et nettoyer l'environnement :
```sh
minikube stop
kubectl delete namespace dev preprod prod
```

## Auteurs
- **Antoine Rotinat**


