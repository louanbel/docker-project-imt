# **PROJECT DEVOPS 2024**

`Author: Louan Bélicaud et Cyprien Crusson`
# Sommaire
- [1 - Docker compose](#1---docker-compose)
- [2 - Ansible](#2---ansible)
- [3 - Kubernetes](#3---kubernetes)

# 1 - Docker compose
Voici les étapes afin de publier les images dans l'Artifact Registry de Google Cloud Platform :
### a) Authentifier le PC à l'Artifact Registry
```
gcloud auth configure-docker <region>-docker.pkg.dev
```
### b) Définir le path des images dans le docker-compose.yaml
Il faut remplacer ces lignes par le bon path :
```
    image: us-central1-docker.pkg.dev/tuto-ansible-422206/voting-images/worker
```
### c) Construire les images
```
docker-compose build
```
### d) Poussez les images dans l'Artifact Registry
```
docker-compose push
```

# 2 - Ansible

### Construire les pods et services Kubernetes adaptés
### a) Créer deux VMs
Depuis Google Cloud, créer une VM pour le serveur principal (primary) et une autre pour le serveur de backup (standby) sur la même région.

### b) Ajouter un tag à chaque VM
Il faut créer une règle de pare-feu sur GCP pour autoriser le traffic entre les deux VMs sur le port 5432 en TCP.
Il faut ensuite ajouter un tag à chaque VM pour pouvoir les associer à cette règle de pare-feu :
```
gcloud compute instances add-tags primary --tags=db-server
gcloud compute instances add-tags standby --tags=db-server
```
### c) Les renseigner dans l'inventaire
Il faut renseigner les adresses IP des VMs dans le fichier `inventories/gcp.yaml` et `playbook.yaml` (remplacer les TODO).
### d) Exécuter le playbook
Exécuter le playbook avec la commande suivante :
```
ansible-playbook playbook.yml
```
Si vous voulez lancer que le rôle `pg_install` ou `pg_replication`, vous pouvez le faire avec les commandes suivantes :
```
ansible-playbook playbook.yml --tags="pg_install"
```
### e) Renseigner l'IP du serveur primary dans le fichier `/k8s-ansible/endpoint-slice.yaml`
Il faut renseigner l'adresse IP de la VM primary dans le fichier `/k8s-ansible/endpoint-slice.yaml` pour que les services Kubernetes puissent communiquer avec la base de données.

### f) Exécuter le playbook de backup
Exécuter le playbook de backup avec la commande suivante :
```
ansible-playbook backup.yml
```
Le fichier de backup sera nommé `db.sql`.

## Organisation du projet
Le projet est organisé de la manière suivante :
- `/inventories` : Contient le fichier `gcp.yaml` qui définit notre inventaire avec nos VMs.
- `/roles` : Contient deux rôles, `pg_install` pour l'installation et la configuration de postgresql et `pg_replication` pour la configuration de la réplication.
- `/playbook.yml` : Le playbook principal qui appelle les deux rôles.
- `/backup.yaml`: Le playbook a utiliser pour faire un backup de la base de données d'une VM.
- `/db.sql` : La sauvegarde de la base de données.
- `pgpass_template.j2` : Le template pour le fichier `.pgpass` qui permet de se connecter à la base de données.

# 3 - Kubernetes
Voici les étapes afin de déployer les services sur Kubernetes. Il faut au préalable avoir créer un projet GCP
### a) Se connecter au project GCP
```
gcloud config set project <PROJECT_ID>
```
### b) Créer un cluster Kubernetes
```
gcloud container clusters create NAME-CLUSTER --machine-type n1-standard-2 --num-nodes 3 --zone us-central1-c
```
### c) Se connecter au cluster
```
gcloud container clusters get-credentials CLUSTER-NAME --region REGION --project PROJECT-ID
```
### d) Créer les éléments kubernetes adaptés pour communiquer avec la DB persistante (deployment, services, pvc, job) 
```
kubectl apply -f k8s-ansible/
```
ou
```
kubectl create -f k8s-ansible/
```

### e) Vérifier que les services sont bien déployés
```
kubectl get pods
kubectl get services
```
### f) Se connecter aux interfaces web à partir des IP externes des services

Vous pouvez retrouver les IPs externes des services avec la commande suivante :
```
kubectl get services
```
