🛡️ Mini SOC Dashboard - Infrastructure Kubernetes

Ce projet est une plateforme de gestion de logs de sécurité (SOC) déployée sur un cluster Kubernetes local (Minikube). Il permet de collecter, stocker (MySQL) et visualiser des événements réseau via une architecture micro-services sécurisée et résiliente.

🏗️ Architecture

Frontend : React.js servi par Nginx.

Backend : Node.js Express (API REST).

Base de données : MySQL 8.0 pour la persistance.

Ingress : Nginx Ingress Controller (Gateway).

Sécurité : Contrôle d'accès basé sur les rôles (RBAC) et gestion des Secrets.

Voici le schéma de flux du Mini SOC :

┌──────────────────────────────────────────────────────────┐
│                   KUBERNETES (Minikube)                  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │      NGINX Ingress Controller (Gateway)             │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │ Routage :                                    │  │  │
│  │  │ - /    → frontend-service (port 80)          │  │  │
│  │  │ - /api → backend-service  (port 8000)        │  │  │
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


🚀 Guide de déploiement

Lancement Rapide (Automatisation)

1. Déploiement complet

Ce script démarre Minikube (avec Ingress), configure le démon Docker, build les images du SOC et déploie l'infrastructure complète (Deployments, Services, PVC, RBAC).

Sur Windows (PowerShell) :

./deploy.ps1


Sur Linux / macOS :

chmod +x deploy.sh
./deploy.sh


Note importante : Une fois le déploiement terminé, n'oubliez pas d'ouvrir un terminal dédié et de lancer la commande minikube tunnel pour rendre l'interface accessible sur votre navigateur à l'adresse http://localhost.

2. Nettoyage (garde la persistance)

Ce script supprime proprement les ressources Kubernetes (Pods, Services, Ingress, Secrets) mais conserve vos logs dans le volume MySQL.

Sur Windows (PowerShell) :

./cleanup.ps1


Sur Linux / macOS :

chmod +x cleanup.sh
./cleanup.sh


3. Nettoyage total

Ce script supprime toutes les ressources Kubernetes ainsi que les données en persistance sur la Base de Données (PVC).

Sur Windows (PowerShell) :

./cleanup-total.ps1


Sur Linux / macOS :

chmod +x cleanup-total.sh
./cleanup-total.sh


🛠️ Lancement Manuel (Pas à pas)

1. Prérequis

Assurez-vous d'avoir installé :

Docker Desktop

Minikube

kubectl

2. Démarrage de l'environnement

Ouvrez un terminal et lancez le cluster :

minikube start
minikube addons enable ingress


3. Préparation des images Docker (Locale)

Pour que Kubernetes utilise vos images sans passer par Docker Hub, buildez-les directement dans le démon Docker de Minikube :

Sur Windows (PowerShell) :

& minikube -p minikube docker-env --shell powershell | Invoke-Expression


Sur Linux / macOS / Git Bash :

eval $(minikube docker-env)


Ensuite, buildez les images :

# Build du Backend
docker build -t backend-service:latest ./backend

# Build du Frontend
docker build -t frontend-service:latest ./frontend


4. Déploiement des micro-services

Appliquez les configurations présentes dans le dossier k8s/ :

kubectl apply -f k8s/


📦 Gestion des Images (Docker Hub vs Local)

Par défaut, les fichiers YAML dans k8s/ pointent vers les images distantes sur Docker Hub :

chahrazedbouacida/backend-soc:v1

chahrazedbouacida/frontend-soc:v1

Pour utiliser vos propres images locales :

Modifiez la ligne image: dans les fichiers backend-deployment.yaml et frontend-deployment.yaml.

Remplacez le nom par backend-service:latest et frontend-service:latest (ou votre propre tag).

Assurez-vous d'avoir bien exécuté l'étape de préparation des images Docker locale ci-dessus.

5. Accès à l'application

Sur Windows / macOS : Lancez le tunnel dans un terminal séparé :

minikube tunnel


Sur Linux : Récupérez l'IP de l'Ingress :

kubectl get ingress


🔐 Sécurité & Résilience

💾 Persistance (PVC)

Le système sépare le calcul du stockage via un Persistent Volume Claim. Même si le Pod MySQL est supprimé, les données restent sur le disque et sont reconnectées au nouveau Pod.

🛡️ Sécurisation du Cluster (RBAC)

Mise en place du principe du moindre privilège via un ServiceAccount dédié (soc-viewer-sa).

Vérification des droits RBAC :

# Lecture autorisée (Attendu: yes)
kubectl auth can-i get pods --as=system:serviceaccount:default:soc-viewer-sa

# Suppression interdite (Attendu: no)
kubectl auth can-i delete pods --as=system:serviceaccount:default:soc-viewer-sa


Accès final : http://localhost