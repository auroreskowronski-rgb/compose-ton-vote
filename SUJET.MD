# [header] FDI-DOKR | Project contained vote app

# [module] Modalités

| Type | Description |
| - | - |
| Dépôt | https://rendu-git.etna-alternance.net/module-10600/activity-57763/group-1087548 |
| Correction | Moulinette |
| Durée | 2 jours |
| Effectif | Seul |

# [goals] Objectifs

- Conteneuriser une application de vote avec plusieurs services, le tout orchestré par un docker-compose

# [project] Consignes Dockerfiles

Vous devez créer 3 images.
Les spécifications de chaque image sont décrites ci-dessous.

## [exercices] Poll

L'image doit être basée sur une image officielle de [`Python alpine`](https://hub.docker.com/_/python).

Les dépendances de l'application doivent être installées.

L'application peut être démarrée avec :

`flask run --host=0.0.0.0 --port=80`

Elle s'exposera au port `80`.

## [exercices] Result
L'image doit être basée sur une image officielle [`Node.js 23 slim`](https://hub.docker.com/_/node).

Les dépendances de l'application doivent être installées.

Le répertoire `node_modules` doit être exclu du contexte de construction.

L'application peut être démarrée avec :

`npm start`

Elle s'exposera au port `80`.

## [exercices] Worker

L'image sera construite en utilisant une construction en plusieurs étapes.

### [exercices] Première étape - build

La première étape doit être basée sur `maven:3.9.9-eclipse-temurin-24-alpine` et être nommée `builder`.

Elle doit être utilisée pour construire (bien sûr) et empaqueter l'application Worker en utilisant les commandes suivantes :
- `mvn dependency:resolve`, depuis le répertoire contenant `pom.xml`.
- puis, `mvn package`, à partir du répertoire contenant le répertoire `src`.

Cela génère un fichier dans le répertoire cible nommé `worker-jar-with-dependencies.jar` dans le dossier `target` (relatif à votre répertoire `WORKDIR`).

### [exercices] Deuxième étape - exécution
 
La deuxième étape doit être basée sur `eclipse-temurin:24-jre-alpine`.

C'est celle qui exécute réellement le worker en utilisant la commande :

`java -jar worker-jar-with-dependencies.jar`

# [project] Consignes Docker compose

Vous avez maintenant 3 Dockerfiles qui créent 3 images isolées.

Il est maintenant temps de les faire fonctionner ensemble en utilisant Docker Compose !

Créez un fichier `docker-compose.yml` qui sera responsable de l'exécution et de la liaison de vos conteneurs.

Votre fichier Docker Compose doit contenir :

## [project] **5 services**
### [exercices] poll
- Construit votre image de `poll`.
- Redirige le port `5000` de l'hôte vers le port `80` du conteneur.
- Définit correctement la variable d'environnement nécessaire.

### [exercices] redis
- Utilise une image officielle existante de [`Redis 7`](https://hub.docker.com/_/redis).
- Ouvre le port `6379`.

### [exercices] worker
- Construit votre image de `worker`
- Définit correctement les mêmes variables d'environnement que les services `poll` et `result`.

### [exercices] db
- Utilise une image officielle existante de `PostgreSQL 16`(https://hub.docker.com/_/postgres).
- A son schéma de base de données créé lors du premier démarrage du conteneur.
- Définit correctement les variables d'environnement appropriées.
- Ouvre le port `5432`

### [exercices] result
- Construit votre image de `result`.
- Redirige le port `5001` de l'hôte vers le port `80` du conteneur.
- Définit correctement les variables d'environnement nécessaires.

## [project] **3 réseaux**
- `poll-network` pour permettre le service `poll` de communiquer avec `redis`.
- `result-network` pour permettre le service `result` de communiquer avec `db`.
- `worker-network` pour permettre le service `worker` de communiquer avec `redis` et `db`.

## [project] **1 volume**
- `db-data` pour permettre aux données de la base de données d'être persistantes, si le conteneur s'éteint.

## [warning] Attention
Il est interdit d'utiliser l'instruction `ENTRYPOINT` dans les ***Dockerfile***.
Il est également interdit d'utiliser les `links` dans le ***docker-compose***

# [project] Architecture du projet

Vous pouvez pousser les fichiers donnés en ressources, ce n'est pas obligatoire, mais dans un portfolio c'est plus fonctionnel avec toute l'application :)

```
.
├── docker-compose.yml
├── ...
├── poll/
|    |   ...
|    └── Dockerfile
├── result/
|    |   ...
|    └── Dockerfile
└── worker/
     |   ...
     └── Dockerfile
```


