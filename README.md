# 3DOKR
Application de vote conteneurisée sous Docker

## Table des matières

- [Introduction](#introduction)
- [Installation](#installation)
- [Informations Docker-compose](#informations-docker-compose)
- [Déploiement Swarm](#déploiement-docker-swarm)
- [Contributeurs](#contributeurs)

## Introduction

Ce projet vise à conteneuriser le processus de déploiement de l'application de vote, qui permet à un utilisateur de voter entre deux propositions. Actuellement, l'application est lancée à l'aide de scripts bash et utilise Python, .NET Core, Node.js, PostgreSQL et Redis. L'objectif est de transformer ce processus en utilisant des conteneurs Docker pour simplifier la gestion, le déploiement et facilité de gestion de l'application.

## Installation

### 1.Récupération du Code Source
```bash
git clone https://github.com/jurjerie1/3DOKR.git
```

### 2.Build avec Docker Compose
Assurez-vous d'être dans le répertoire du projet où se trouve le fichier `compose.yaml`.
```bash
docker compose build
```

### 3.Démarrage de l'Application
Une fois que l'application a bien été build, vous pouvez la démarrer:
```bash
docker compose up
```
L'application devrait être accessible via votre naviguateur web sur :
- [Pour voter](http://localhost:8080/) (Port 8080)
- [Résultats des votes](http://localhost:8081/) (Port 8081)

### Remarque
Assurez-vous que Docker est en cours d'exécution sur votre système. Vous pouvez vérifier l'état de Docker en exécutant `docker --version` et `docker-compose --version`.

## Informations Docker compose

### Depends on
Des `depends_on` ont été mis en place afin de définir des dépendances entre les composants:
```yaml
depends_on:
      - redis
```
(depends_on de l'application web vote, il ne peut pas se lancer tant que la database redis dont il dépend n'a pas été allumé)

### Healthcheck
Un service de Healthcheck a été mis en place pour chaque partie de l'application afin de vérifier leur statut à interval régulier :
```yaml
healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
```
(Mise en place du Healthcheck pour la database redis)

### Volumes
Des volumes ont été mis en place afin de conserver les données même après redémarrage de l'application :
```yaml
volumes:
      - ./redis-data:/data
```
(Mise en place du volume pour la database redis)

### Réseaux 
Nous avons mis en place 3 réseaux, 1 pour : vote + Redis, 1 pour : result + Postgre et 1 pour : worker + Redis + Postgre :

```yaml
networks:
  vote:
  result:
  link-vote-result:
```

## Déploiement Docker Swarm
### Mise en place du Docker Swarm
Une fois que le compose classique avec les Dockerfile a été mis en place, nous avons mis en place un cluster Docker Swarm.
Nous avons :
1. Push les images utilisées sur Docker Hub (modification du build en image et mettre la référence sur Docker Hub)
2. Rajouter pour chaque partie de l'application un replica (construction de plusieurs instances d'un conteneur Docker) :
```yaml
deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == worker
```
3. Créer des clusters de machine avec 1 manager et 2 worker, utilisation de vragrant (code vu en cours)
4. Envoi du fichier `compose-swarm.yaml` sur le manager
### Installation du Docker Swarm par l'utilisateur
Pour lancer le Swarm il faut :
1. Clone le repo `docker-swarm`
2. Exécuter la commande : 
```yaml
docker stack deploy -c compose-swarm.yaml voting-app
```
## Contributeurs
- [https://github.com/jurjerie] (Pierre Van Maele)
- [https://github.com/maddogos] (Léonard Trève)
