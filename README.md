# 5VIRDAT-AR
# Projet Kubernetes - Déploiement Multi-Environnements avec Minikube

## Prérequis
Avant de commencer, assurez-vous d'avoir une instance Minikube opérationnelle. Pour ce projet, nous utilisons **Minikube version 1.32.0**.

Avoir aussi docker et docker compose de fonctionnelle.

Avoir des clés API fonctionnelle de STRIPE, nécéssaire pour la première application (https://dashboard.stripe.com)

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

# Déploiement de la première application
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
Chaque environnement possède un ConfigMap stockant les variables d'environnement de l'application (c'est dans ces ConfigMaps qu'il faudra rentrer les clés API) :
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

## Déploiement des Composants

### Appliquer les fichiers de configuration Kubernetes
Le fichier yaml ce trouve sur ce repo (https://github.com/RORODABADO/5VIRDAT-AR/blob/main/rocket-ecommerce.yaml) !
```sh
git clone https://github.com/RORODABADO/5VIRDAT-AR/blob/main/rocket-ecommerce.yaml
kubectl apply -f rocket-ecommerce.yaml
```

### Vérifier les namespaces créés
```sh
kubectl get namespaces
```
![image](https://github.com/user-attachments/assets/4b7022ae-127c-4a95-82c9-0c63c160a451)

### Vérifier que les pods sont en cours d'exécution
Ces commandes vont nous permettre de voir les pod créer sur les 3 environnements 
```sh
kubectl get pods -n dev
kubectl get pods -n preprod
kubectl get pods -n prod
```
![image](https://github.com/user-attachments/assets/73e5d9f6-1eb8-4014-a2b5-bcf81ed1d8cb)
![image](https://github.com/user-attachments/assets/528a4b59-8776-4652-8af8-7b8743ba0363)
![image](https://github.com/user-attachments/assets/073a0e50-d1a0-42c3-9618-3e067c40abfc)

### Vérifier les services exposés
Ces commandes vont nous permettre de voir les services créer sur les 3 environnements 
```sh
kubectl get services -n dev
kubectl get services -n preprod
kubectl get services -n prod
```
![image](https://github.com/user-attachments/assets/c455c4cd-791e-4ac0-b36f-4db6eb77ed29)
![image](https://github.com/user-attachments/assets/bc700115-e6d0-4ee6-b504-a33ffd4bcf80)
![image](https://github.com/user-attachments/assets/b576d080-cb13-4b91-9c23-3b5f4c62bc93)

### Rendre accesible l'application depuis l'addresse ip publique de notre minikube

| Environnement | Namespace | Port Exposé Publiquement | URL |
|--------------|-----------|-------------|-------------|
| Développement | dev | 5005 | http://188.213.128.250:5005
| Préproduction | preprod | 5006 | http://188.213.128.250:5006
| Production | prod | 5007 | http://188.213.128.250:5007

Ces commandes permet de forward les port des pods des 3 environnement pour les rendre accésible depuis l'ip publique du minikube
```sh
kubectl port-forward --address 0.0.0.0 pod/$(kubectl get pod -l app=rocket-ecommerce -n dev -o jsonpath="{.items[0].metadata.name}") -n dev 5005:5005
kubectl port-forward --address 0.0.0.0 pod/$(kubectl get pod -l app=rocket-ecommerce -n preprod -o jsonpath="{.items[0].metadata.name}") -n preprod 5006:5005
kubectl port-forward --address 0.0.0.0 pod/$(kubectl get pod -l app=rocket-ecommerce -n prod -o jsonpath="{.items[0].metadata.name}") -n prod 5007:5005
```

### Tests du bon fonctionnement sur les 3 applications avec l'api STIPE 
Une fois nos applications exposé nous pouvons nous connecter via les url plus haut : 
- Dev : ![image](https://github.com/user-attachments/assets/c6359b09-da53-436d-9e88-5026f8b28590)
- PreProd : ![image](https://github.com/user-attachments/assets/7c2cf121-7a4e-4574-bd9b-d35657ca7f08)
- Prod :![image](https://github.com/user-attachments/assets/63b0888f-9d87-4293-8244-ec1b5ab38c9f)


## Auteurs
- **Antoine Rotinat**


