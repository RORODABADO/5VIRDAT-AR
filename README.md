# 5VIRDAT-AR
Prérequis

Avant de commencer, assurez-vous d'avoir une instance Minikube opérationnelle. Pour ce projet, nous utilisons Minikube version 1.32.0.
Avoir aussi docker et docker compose de fonctionnelle. 
A savoir que tout les manifest en yaml son sur le repo !

Démarrage de Minikube

Lancez Minikube avec la commande suivante :

minikube start --listen-address=0.0.0.0 --memory=max --cpus=max --kubernetes-version=v1.32.0

Création des namespace : 

Dans le cadre du projet et en accord avec la consigne nous allons créer 3 environnement dev|preprod|prod (namespace) avec le manifest "create-namespace.yaml" : 

k create -f create-namespace.yaml

Donc appartire de là nous venons de créer 3 environnements (namespace) : 
- dev-namespace
- preprod-namespace
- prod-namespace

Pour confirmer et voir les namespace que nous venons de créer nous pouvon faire cette commande : 

k get namespaces

![image](https://github.com/user-attachments/assets/84cf19dd-1317-4068-b342-1ac2e782eaba)


Pour info, par default avec minikube nous somme dans l'envrionnement (namespace) "default", pour changer d'envrionnement nous utiliserons la commande suivante : 

k config set-context --current --namespace=nom-du-namespace


Déploiement / Configuration de la première application : 

Pour cette première nous allons utiliser "Rocket eCommerce", nous allons build l'image et push l'image sur mon docker hub. 
A savoir que l'image docker à été optimiser pour la rendre légère en utilisant python:3.11.5-alpine et nginx:alpine-slim. 
Et l'image (non-build) ce trouve sur mon repos git hub : "https://github.com/RORODABADO/5VIRDAT-AR-APPLI1"

Copie du repos de l'application : 
git clone https://github.com/RORODABADO/5VIRDAT-AR-APPLI1
cd 5VIRDAT-AR-APPLI1/

Build et push de l'image sur mon docker hub : 
docker build -t antoinerotinat/rocket-ecommerce:latest . && docker push antoinerotinat/rocket-ecommerce:latest

Nous utiliserons cette image pour déployer notre application avec minikube

Dans ce repos un fichier nommé "rocket-ecommerce.yaml", c'est le manifest pour déployer notre application dans le namespace "dev-namespace".
Dans ce manifest nous allons créer 3 composants tous relier au namespace "dev-namespace" : 

- un ConfiMap qui va nous permettre d'injecter les variables d'environnement (clé api) nécésaire pour le fonctionnement de l'application avec stripe 
- un Deployment qui va nous permet de créer l'application en spécifiant l'image que nous venons de builder précédament
- un Service, qui va permet de mettre en réseau notre application (type LoadBalancer)

Pour éxecuter le manifest, voici la commande : 
k create -f rocket-ecommerce.yaml

![image](https://github.com/user-attachments/assets/b0764824-3eda-454e-9468-9eff0e673311)

Ensuite nous pouvons vérifier les composant que nous venons de déployer avec la commande : 

k get all 

![image](https://github.com/user-attachments/assets/f2f87e0d-9beb-4e7c-b6c9-de5d87675a1f)



