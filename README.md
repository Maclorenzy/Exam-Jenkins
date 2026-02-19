# Exam Jenkins - Laurent Hoarau
## Ingénieur DevOps - DataScientest

![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red)
![Kubernetes](https://img.shields.io/badge/Kubernetes-K3s-blue)
![Docker](https://img.shields.io/badge/Docker-Hub-blue)
![Python](https://img.shields.io/badge/Python-FastAPI-green)

## 📋 Description

Examen du déploiement d'une application de gestion de films et de castings basée sur une architecture **microservices** avec FastAPI, déployée automatiquement via un pipeline Jenkins sur un cluster Kubernetes K3s.

## 🏗️ Architecture

- **movie-service** : API FastAPI de gestion des films (PostgreSQL)
- **cast-service** : API FastAPI de gestion des castings (PostgreSQL)
- **nginx** : Reverse proxy

## 🚀 Pipeline CI/CD

| Stage | Description |
|-------|-------------|
| Docker Build | Construction des images movie-service et cast-service |
| Docker Run | Lancement des conteneurs de test avec PostgreSQL |
| Test Acceptance | Validation des endpoints via curl |
| Docker Push | Push des images sur DockerHub |
| Déploiement dev | Déploiement automatique en développement |
| Déploiement QA | Déploiement automatique en QA |
| Déploiement staging | Déploiement automatique en staging |
| Déploiement prod | Déploiement **manuel** depuis la branche master |

## 🌍 Environnements Kubernetes

| Namespace | Movie Service | Cast Service |
|-----------|-------------|--------------|
| dev | :30007 | :30008 |
| qa | :30009 | :30010 |
| staging | :30011 | :30012 |
| prod | :30013 | :30014 |

## 🔧 Stack technique

- **CI/CD** : Jenkins
- **Conteneurs** : Docker
- **Orchestration** : Kubernetes K3s
- **Déploiement** : Helm
- **Registry** : DockerHub ([laurenthoarau](https://hub.docker.com/u/laurenthoarau))
- **Backend** : Python 3.8 / FastAPI
- **Base de données** : PostgreSQL 12

## 📦 DockerHub

Images disponibles sur [hub.docker.com/u/laurenthoarau](https://hub.docker.com/u/laurenthoarau) :
- `laurenthoarau/movie-service`
- `laurenthoarau/cast-service`

## 👤 Auteur

**Laurent Hoarau** - Promotion jan26_bootcamp_devops - DataScientest
