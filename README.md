# 🗳️ Compose ton vote

![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Compose](https://img.shields.io/badge/Docker_Compose-2496ED?logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?logo=nodedotjs&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?logo=openjdk&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?logo=redis&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)

Conteneurisation d'une application de vote multi-services, orchestrée avec Docker Compose.

> 🎓 **Exercice du module FDI-DOKR (ETNA)**. L'énoncé complet est disponible dans ce dépôt : [`SUJET.html`](SUJET.html) / [`SUJET.MD`](SUJET.MD).
> Le code applicatif (`poll`, `result`, `worker`) est fourni par l'ETNA. Mon travail porte sur les **Dockerfile** et le **`docker-compose.yml`**.

## 🧩 Fonctionnement

1. L'utilisateur vote sur la page **poll** (Flask).
2. Le vote est placé dans une file **Redis**.
3. Le **worker** (Java) lit Redis et enregistre le vote dans **PostgreSQL**.
4. La page **result** (Node.js / Socket.io) affiche les résultats en direct.

```
Navigateur ──:5000──▶ poll ──▶ redis ──▶ worker ──▶ db ◀── result ◀──:5001── Navigateur
```

| Service | Technologie | Rôle | Port hôte → conteneur |
|---|---|---|---|
| `poll` | Python / Flask (`python:3-alpine`) | Page de vote | 5000 → 80 |
| `redis` | `redis:7` | File d'attente des votes | 6379 |
| `worker` | Java / Maven, build multi-stage | Transfère les votes de Redis vers la base | – |
| `db` | `postgres:16` | Stockage des votes | 5432 |
| `result` | Node.js (`node:23-slim`) | Affichage des résultats | 5001 → 80 |

## 🌐 Réseaux et volume

| Réseau | Services | Rôle |
|---|---|---|
| `poll-network` | poll, redis | poll n'accède qu'à Redis |
| `result-network` | result, db | result n'accède qu'à la base |
| `worker-network` | worker, redis, db | le worker relie Redis et la base |

Le volume **`db-data`** conserve les données PostgreSQL après l'arrêt des conteneurs. Le schéma (`schema.sql`) est appliqué automatiquement au premier démarrage via `/docker-entrypoint-initdb.d`.

## 🚀 Lancer le projet

Prérequis : Docker et Docker Compose.

```bash
git clone https://github.com/auroreskowronski-rgb/compose-ton-vote.git
cd compose-ton-vote
docker compose up --build -d
```

- Voter : http://localhost:5000
- Résultats : http://localhost:5001

```bash
docker compose ps              # état des services
docker compose logs -f worker  # suivre le traitement des votes
docker compose down            # arrêter (les votes sont conservés)
docker compose down -v         # arrêter et supprimer aussi les données
```

## 🔍 Points techniques

- **Worker en multi-stage** : l'étape `builder` (`maven:3.9.9-eclipse-temurin-24-alpine`) compile le jar, l'image finale (`eclipse-temurin:24-jre-alpine`) ne contient que le JRE et `worker-jar-with-dependencies.jar`.
- **Cache de build** : dépendances installées avant la copie du code (`requirements.txt`, `package.json`, `pom.xml`).
- **`result/.dockerignore`** : `node_modules` est exclu du contexte de build.
- **Communication par nom de service** : `REDIS_HOST=redis`, `POSTGRES_HOST=db` (DNS interne de Compose).
- **Contraintes de l'énoncé respectées** : pas d'`ENTRYPOINT` dans les Dockerfile, pas de `links` dans le compose.

## 📁 Structure

```
.
├── docker-compose.yml
├── schema.sql
├── poll/     (Dockerfile + app Flask)
├── result/   (Dockerfile + .dockerignore + app Node.js)
├── worker/   (Dockerfile multi-stage + projet Maven)
├── SUJET.html / SUJET.MD
└── README.md
```

## 👤 Auteure

Aurore Skowronski – étudiante à l'ETNA · [GitHub](https://github.com/auroreskowronski-rgb)
