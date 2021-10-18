
# POC Minio > HIVE Metastore > Trino (anciennement Presto) > Superset


### A quoi servent toutes ces briques ?

- Minio : Stockage objet de type S3, ce sera notre service de stockage de nos données
- Hive Metastore : Métadonnées permettant la constitution de base de données relationnelles sur des données persistentes réparties (par exemple des fichiers dans un bucket S3) 
- Trino : Anciennement, Presto. Moteur distribué de requête SQL, aisément scalable.
- Superset : Outil de dataviz / BI

NB : Toutes ces briques sont open source
### Installation

- modifier les fichiers `conf/metastore-site.xml` et `etc/catalog/minio.properties` pour renseigner les propriétés `MINIO_URL`, `MINIO_USER` et `MINIO_PASSWORD`
- `docker-compose up -d`

### Connection d'un bucket Minio avec Trino

```
# Pour rentrer dans l'interface de commande TRINO
docker exec -it trino trino 
# Pour créer le schéma de connexion vers un bucket S3
CREATE SCHEMA IF NOT EXISTS minio.<CHOISISSEZ_UN_NOM_DE_SCHEMA> WITH (location = 's3a://<BUCKET_NAME>/'); 
# Pour créer une table vers un répertoire d'un bucket
CREATE TABLE IF NOT EXISTS minio.<CHOISISSEZ_UN_NOM_DE_SCHEMA>.<CHOISISSEZ_UN_NOM_DE_TABLE> (a VARCHAR, b VARCHAR, c VARCHAR) WITH (external_location = 's3a://<BUCKET_NAME>/<PATH_OF_FOLDER_DATA>', format='csv');
# On peut alors requêter la table facilement :
# SELECT * FROM minio.<CHOISISSEZ_UN_NOM_DE_SCHEMA>.<CHOISISSEZ_UN_NOM_DE_TABLE> LIMIT 10;
```

### Installation Superset

NB : Ces étapes pourraient être intégrées dans un entrypoint.
```
# Création du user admin (login / mdp peuvent être modifiés)
docker exec -it superset superset fab create-admin \
               --username admin \
               --firstname Superset \
               --lastname Admin \
               --email admin@superset.com \
               --password admin

# Upgrade de la DB superset
docker exec -it superset superset db upgrade

# Setup roles
docker exec -it superset superset init
```

### Lien Superset > Trino

- se connecter sur superset (localhost:8088)
- Data > Databases > + Database
- Type Presto, SQL Alchemy URI : `trino://trino@trino:8080/minio`

> Vous pouvez maintenant visualiser les données minio depuis superset


### Ressources intéressantes :

- https://medium.com/codex/modern-data-platform-using-open-source-technologies-212ba8273eab
- https://janakiev.com/blog/presto-trino-s3/
- https://itnext.io/trino-on-nomad-79cb398a826


