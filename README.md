# **PROJECT DEVOPS 2024**

`Author: Louan Bélicaud et Cyprien Crusson`

# 1 - Docker compose
Voici les étapes afin de publier les images dans l'Artifact Registry de Google Cloud Platform :
### a) Authentifier le PC à l'Artifact Registry
```
gcloud auth configure-docker <region>-docker.pkg.dev
```
### b) Définir le path des images dans le docker-compose.yaml
Il faut remplacer ces lignes par le bon path :
```
    image: europe-west9-docker.pkg.dev/geometric-flux-421107/voting-images/worker
```
### c) Construire les images
```
docker-compose build
```
### d) Poussez les images dans l'Artifact Registry
```
docker-compose push
```

# 2 - Kubernetes
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
### d) Créer les pods et les services 
```
kubectl apply -f k8s/
```
ou
```
kubectl create -f k8s/
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

# 3 - Ansible

### Construire les pods et services Kubernetes adaptés
Il faut avoir créé les pods et services Kubernetes adaptés pour Ansible. Vous pouvez dans un premier temps supprimer les précédents pods :
```
kubectl delete -f k8s/
```
Puis créer les nouveaux :
```
kubectl apply -f k8s/ansible/
```

## Lancer le projet
### a) Créer deux VMs
Créer une VM pour le serveur principal (primary) et une autre pour le serveur de backup (standby).
### b) Ajouter un tag à chaque VM
Il faut créer une règle de pare-feu sur GCP pour autoriser le traffic entre les deux VMs sur le port 5432 en TCP.
Il faut ensuite ajouter un tag à chaque VM pour pouvoir les associer à cette règle de pare-feu :
```
gcloud compute instances add-tags primary --tags=db-server
gcloud compute instances add-tags standby --tags=db-server
```
### c) Les renseigner dans l'inventaire
Il faut renseigner les adresses IP des VMs dans le fichier `inventories/gcp.yaml` (remplacer les TODO)
### d) Exécuter le playbook
Exécuter le playbook avec la commande suivante : 
```
ansible-playbook playbook.yml
```
Si vous voulez lancer que le rôle `pg_install` ou `pg_replication`, vous pouvez le faire avec les commandes suivantes :
```
ansible-playbook playbook.yml --tags="pg_install"
```
### e) Exécuter le playbook de backup
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
