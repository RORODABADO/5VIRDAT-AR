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
Les Namespaces sont utilisée avec des composants de haut niveau seulement (pods, ingress, jobs, deployments, replicaset,configmap, secret etc...), il permet de relier ces composant dans différents environnements.
Trois namespaces sont créés pour organiser les déploiements :
- `dev`
- `preprod`
- `prod`

### 2️⃣ ConfigMaps
Les ConfigsMaps servent à stocker des données de configuration sous forme de paires clé-valeur, a séparer la configuration de l'application du code.
Chaque environnement possède un ConfigMap stockant les variables d'environnement de l'application (c'est dans ces ConfigMaps qu'il faudra rentrer les clés API) :
- `rocket-ecommerce-config` (défini pour chaque namespace)

### 3️⃣ Deployments
Un Deployment est un objet Kubernetes qui définit et gère le déploiement d’une application sous forme de pods.
L'application est déployée dans chaque environnement avec un **Deployment Kubernetes**. 
- Nom du déploiement : `rocket-ecommerce-deployment`
- Nombre de réplicas : `1` (modifiable pour la scalabilité)
- Conteneur utilisé : `antoinerotinat/rocket-ecommerce:latest`
- Port exposé : `5005`
- Variables d'environnement injectées via le ConfigMap

### 4️⃣ Services (NodePort)
Le type NodePort permet d’acéder à ses pods (applications) depuis l'exterieur de kubernetes.
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

Nous pouvons aussi ouvrir un shell sur le pod pour véfifer que les variables ont bien été injecté : 
```sh
k exec -it <nom-du-pod> -- printenv
```

Pour tester le bon fonctionnement avec l'api STRIPE, nous pouvons effecter un achat sur l'application pour voir si cela remonte bien dans le dashboard STRIPE (pour l'utilisation de l'application, il faut ce référé sur le repo de celle-ci) : 

![image](https://github.com/user-attachments/assets/abd8ecee-2d4f-4705-99fa-5c42aa7ad30f)
![image](https://github.com/user-attachments/assets/0a7a1c11-89d9-4253-a0c8-4540558f4bd3)

Nous ponvons voir que les transactions remonte bien (des 3 applications des diffrents environnement) et que les clé api ce sont bien injecter via les ConfigMap : 

![image](https://github.com/user-attachments/assets/f4417728-dec9-4795-8b03-bf0aa32744fa)


# Déploiement de la deuxième application

Pour cette deuxième applications, nous allons utilisée "django-admnlte", nous allons build l'image et push l'image sur mon docker hub. A savoir que l'image docker à été optimiser pour la rendre légère en utilisant python:3.11.5-alpine et nginx:alpine-slim. Et l'image (non-build) ce trouve sur mon repos github : "https://github.com/RORODABADO/5VIRDAT-AR-APPLI2/tree/master"

Copie du repos de l'application : 

```sh
git clone https://github.com/RORODABADO/5VIRDAT-AR-APPLI2/tree/master
```

```sh
cd 5VIRDAT-AR-APPLI2/
```
Build et push de l'image sur mon docker hub :

```sh
docker build -t antoinerotinat/django-adminlte:latest . && docker push antoinerotinat/django-adminlte:latest
```

Cette application fonctionne avec une base de données mysql.

Par manque de temps, je n'ai pas pu déployer cette application sur les 3 namespace, donc il n'y aura que pour le namespace "dev".

En premier il faut télécharger le manifest en yaml puis créer les composants (https://github.com/RORODABADO/5VIRDAT-AR/blob/main/django-adminlte.yaml). 

```sh
git clone https://github.com/RORODABADO/5VIRDAT-AR/blob/main/django-adminlte.yaml
kubectl apply -f django-adminlte.yaml
```
### 5️⃣ Secret
Dans ce manifest nous déployons un nouveau composant à kubernetes "Secret", ce composant dans Kubernetes permet de stocker des informations sensibles telles que des mots de passe, des jetons d'authentification ou des clés d'API. Ces données sont encodées en base64 ou par des algorithme plus sécurisé pour être ainsi injecter dans d'autre composants, et elles ne sont pas visibles en texte clair dans les fichiers de configuration.

Ensuite j'ai forcer la creation de la bdd en ouvrant un shell sur le pod de mon application : 

```sh
kubectl exec -it django-adminlte-deployment-67df864757-sqtvh -n dev -- /bin/bash
python manage.py migrate
```

### Tests et vérification
Pour vérifier que notre application fonctionne correctement, et est bien relier à notre pod bdd on peut ce connecter sur le pod bdd et rechercher la database et les tables : 
```sh
kubectl exec -it django-adminlte-deployment-67df864757-sqtvh -- bash
mysql -u root -p
SHOW DATABASES;
USE appseed_db_dev;
SELECT * FROM auth_user;
```
![image](https://github.com/user-attachments/assets/5a76d3d5-84be-44e3-9629-4e0a58187a62)
![image](https://github.com/user-attachments/assets/7c914036-a538-408d-9125-dab7e21434e1)


Nous pouvons voir que la base à bien été créer et que les information sont bien saisie.



## Auteurs
- **Antoine Rotinat**


