[SOC] Mini SOC Dashboard - Infrastructure Kubernetes

Ce projet est une plateforme de gestion de logs de securite (SOC) deployee sur un cluster Kubernetes local (Minikube). Il permet de collecter, stocker (MySQL) et visualiser des evenements reseau via une architecture micro-services securisee et resiliente.

Architecture

Frontend : React.js servi par Nginx.

Backend : Node.js Express (API REST).

Base de donnees : MySQL 8.0 pour la persistance.

Ingress : Nginx Ingress Controller (Gateway).

Securite : Controle d'acces base sur les roles (RBAC) et gestion des Secrets.

Schema des flux :

┌──────────────────────────────────────────────────────────┐
│                   KUBERNETES (Minikube)                  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │      NGINX Ingress Controller (Gateway)             │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │ Routage :                                    │  │  │
│  │  │ - /    → frontend-service (port 80)          │  │  │
│  │  │ - /api → backend-service  (port 8000)        │  │  │
│  │  └──────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────┘  │
│                 │                        │               │
│                 ▼                        ▼               │
│      ┌──────────────────┐      ┌──────────────────┐      │
│      │   Frontend Pod   │      │   Backend Pod    │      │
│      │     (React)      │      │    (Node.js)     │      │
│      │     port 80      │      │    port 8000     │      │
│      └──────────────────┘      └────────┬─────────┘      │
│                                         │                │
│        [ RBAC Security ]                │ TCP/SQL (3306) │
│        (soc-viewer-sa)                  ▼                │
│                                ┌──────────────────┐      │
│                                │    MySQL Pod     │      │
│                                │   (mysql-soc)    │      │
│                                │    port 3306     │      │
│                                │ + PersistentVol  │      │
│                                └──────────────────┘      │
└──────────────────────────────────────────────────────────┘

Guide de deploiement

Lancement Rapide (Automatisation)

1. Deploiement complet

Ce script demarre Minikube (avec Ingress), configure le demon Docker, build les images du SOC et deploie l'infrastructure complete (Deployments, Services, PVC, RBAC).

Sur Windows (PowerShell) :

./deploy.ps1


Sur Linux / macOS :

chmod +x deploy.sh
./deploy.sh


Note importante : Une fois le deploiement termine, n'oubliez pas d'ouvrir un terminal dedie et de lancer la commande "minikube tunnel" pour rendre l'interface accessible sur votre navigateur a l'adresse http://localhost.

2. Nettoyage (garde la persistance)

Ce script supprime proprement les ressources Kubernetes (Pods, Services, Ingress, Secrets) mais conserve vos logs dans le volume MySQL.

Sur Windows (PowerShell) :

./cleanup.ps1


3. Nettoyage total

Ce script supprime toutes les ressources Kubernetes ainsi que les donnees en persistance sur la Base de Donnees (PVC).

Sur Windows (PowerShell) :

./cleanup-total.ps1


Lancement Manuel (Pas a pas)

1. Prerequis

Assurez-vous d'avoir installe :

Docker Desktop

Minikube

kubectl

2. Demarrage de l'environnement

minikube start
minikube addons enable ingress


3. Preparation des images Docker (Locale)

Pour utiliser vos images sans passer par Docker Hub, buildez-les directement dans le demon Docker de Minikube :

Sur Windows (PowerShell) :

& minikube -p minikube docker-env --shell powershell | Invoke-Expression


Ensuite, buildez les images :

# Build du Backend
docker build -t backend-service:latest ./backend

# Build du Frontend
docker build -t frontend-service:latest ./frontend


4. Deploiement des micro-services

kubectl apply -f k8s/


Gestion des Images (Docker Hub vs Local)

Par defaut, les fichiers YAML pointent vers les images sur Docker Hub :

chahrazedbouacida/backend-soc:v1

chahrazedbouacida/frontend-soc:v1

Pour utiliser vos images locales :

Modifiez la ligne "image:" dans les fichiers k8s/backend-deployment.yaml et k8s/frontend-deployment.yaml.

Remplacez le nom par "backend-service:latest" et "frontend-service:latest".

Securite et Resilience

Persistance (PVC)

Le systeme separe le calcul du stockage via un Persistent Volume Claim (PVC). Meme si le Pod MySQL est supprime, les donnees restent sur le disque et sont reconnectees au nouveau Pod.

Securisation du Cluster (RBAC)

Mise en place du principe du moindre privilege via un ServiceAccount dedie (soc-viewer-sa).

Verification des droits RBAC :

# Lecture autorisee (Attendu: yes)
kubectl auth can-i get pods --as=system:serviceaccount:default:soc-viewer-sa

# Suppression interdite (Attendu: no)
kubectl auth can-i delete pods --as=system:serviceaccount:default:soc-viewer-sa


Acces final : http://localhost