# TP ansible 2023-2024
`Author:  Louan Belicaud et Cyprien Crusson`
## Lancer le projet
### Créer deux VMs
Créer une VM pour le serveur principal (primary) et une autre pour le serveur de backup (standby).
### Ajouter un tag à chaque VM
Il faut créer une règle de pare-feu sur GCP pour autoriser le traffic entre les deux VMs sur le port 5432 en TCP.
Il faut ensuite ajouter un tag à chaque VM pour pouvoir les associer à cette règle de pare-feu :
```
gcloud compute instances add-tags primary --tags=db-server
gcloud compute instances add-tags standby --tags=db-server
```
### Exécuter le playbook
Exécuter le playbook avec la commande suivante : 
```
ansible-playbook playbook.yml
```
Si vous voulez lancer que le rôle `pg_install` ou `pg_replication`, vous pouvez le faire avec les commandes suivantes :
```
ansible-playbook playbook.yml --tags="pg_install"
```
### Exécuter le playbook de backup
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
