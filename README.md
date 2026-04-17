<div align="center">

# 🛡️ Mini SOC Dashboard
### Plateforme de gestion de logs de sécurité sur Kubernetes

![Kubernetes](https://img.shields.io/badge/Kubernetes-Minikube-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![React](https://img.shields.io/badge/Frontend-React.js-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Node.js](https://img.shields.io/badge/Backend-Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white)
![MySQL](https://img.shields.io/badge/Database-MySQL%208.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Nginx](https://img.shields.io/badge/Gateway-Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)

</div>

---

## 📋 Table des matières

- [Vue d'ensemble](#-vue-densemble)
- [Architecture](#-architecture)
- [Prérequis](#-prérequis)
- [Déploiement rapide](#-déploiement-rapide)
- [Déploiement manuel](#-déploiement-manuel)
- [Gestion des images Docker](#-gestion-des-images-docker)
- [Sécurité et résilience](#-sécurité-et-résilience)
- [Accès à l'application](#-accès-à-lapplication)

---

## 🔍 Vue d'ensemble

Ce projet est une plateforme **SOC (Security Operations Center)** déployée sur un cluster Kubernetes local (**Minikube**). Il permet de :

- 📥 **Collecter** des événements réseau et des logs de sécurité
- 🗄️ **Stocker** les données de manière persistante via MySQL
- 📊 **Visualiser** les événements via une interface React moderne
- 🔐 **Sécuriser** les accès grâce au RBAC Kubernetes

---

## 🏗️ Architecture

### Stack technique

| Composant | Technologie | Port |
|-----------|-------------|------|
| **Frontend** | React.js servi par Nginx | `80` |
| **Backend** | Node.js / Express (API REST) | `8000` |
| **Base de données** | MySQL 8.0 | `3306` |
| **Gateway** | Nginx Ingress Controller | — |
| **Sécurité** | RBAC + Kubernetes Secrets | — |

### Schéma des flux

```
┌──────────────────────────────────────────────────────────┐
│                   KUBERNETES (Minikube)                  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │      NGINX Ingress Controller (Gateway)            │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │ Routage :                                    │  │  │
│  │  │  /     →  frontend-service  (port 80)        │  │  │
│  │  │  /api  →  backend-service   (port 8000)      │  │  │
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
```

---

## ✅ Prérequis

Avant de commencer, assurez-vous d'avoir installé les outils suivants :

| Outil | Description | Lien |
|-------|-------------|------|
| 🐳 **Docker Desktop** | Moteur de conteneurs | [Installer](https://www.docker.com/products/docker-desktop) |
| ☸️ **Minikube** | Cluster Kubernetes local | [Installer](https://minikube.sigs.k8s.io/docs/start/) |
| 🔧 **kubectl** | CLI Kubernetes | [Installer](https://kubernetes.io/docs/tasks/tools/) |

---

## 🚀 Déploiement rapide

> Scripts d'automatisation : démarrage Minikube, build des images, et déploiement complet en une seule commande.

### 1️⃣ Déploiement complet

<details>
<summary><b>🪟 Windows (PowerShell)</b></summary>

```powershell
./deploy.ps1
```

</details>

<details>
<summary><b>🐧 Linux / macOS</b></summary>

```bash
chmod +x deploy.sh
./deploy.sh
```

</details>

> [!IMPORTANT]
> Une fois le déploiement terminé, ouvrez un **terminal dédié** et lancez :
> ```bash
> minikube tunnel
> ```
> L'interface sera alors accessible sur → **http://localhost**

---

### 2️⃣ Nettoyage (avec conservation des données)

Supprime les ressources Kubernetes mais **conserve les logs** dans le volume MySQL.

```powershell
# Windows
./cleanup.ps1
```

---

### 3️⃣ Nettoyage total

Supprime **toutes les ressources**, y compris le PVC et les données persistantes.

```powershell
# Windows
./cleanup-total.ps1
```

> [!WARNING]
> Le nettoyage total supprime définitivement toutes les données de la base MySQL. Cette action est irréversible.

---

## 🔧 Déploiement manuel

### Étape 1 — Démarrage de l'environnement

```bash
minikube start
minikube addons enable ingress
```

### Étape 2 — Build des images Docker en local

Pour utiliser vos images **sans passer par Docker Hub**, buildez-les directement dans le démon Docker de Minikube :

```powershell
# Windows (PowerShell) — pointer Docker vers Minikube
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
```

Puis buildez les images :

```bash
# Build du Backend
docker build -t backend-service:latest ./backend

# Build du Frontend
docker build -t frontend-service:latest ./frontend
```

### Étape 3 — Déploiement des micro-services

```bash
kubectl apply -f k8s/
```

---

## 🐋 Gestion des images Docker

### Images Docker Hub (par défaut)

Les fichiers YAML pointent par défaut vers les images publiques :

```yaml
image: chahrazedbouacida/backend-soc:v1
image: chahrazedbouacida/frontend-soc:v1
```

### Passer en images locales

Modifiez les fichiers suivants :

- `k8s/backend-deployment.yaml`
- `k8s/frontend-deployment.yaml`

Remplacez la ligne `image:` par :

```yaml
# Backend
image: backend-service:latest

# Frontend
image: frontend-service:latest
```

---

## 🔐 Sécurité et résilience

### 💾 Persistance des données (PVC)

Le système sépare **calcul** et **stockage** via un `Persistent Volume Claim`. Même si le Pod MySQL est supprimé ou redémarré, les données restent sur le disque et sont automatiquement reconnectées au nouveau Pod.

```
[ Pod MySQL supprimé ] ──▶ [ Données toujours sur le PVC ] ──▶ [ Nouveau Pod reconnecté ✅ ]
```

---

### 🛡️ Contrôle d'accès RBAC

Application du **principe du moindre privilège** via un `ServiceAccount` dédié : `soc-viewer-sa`.

#### Vérification des droits

```bash
# ✅ Lecture autorisée (résultat attendu : yes)
kubectl auth can-i get pods \
  --as=system:serviceaccount:default:soc-viewer-sa

# ❌ Suppression interdite (résultat attendu : no)
kubectl auth can-i delete pods \
  --as=system:serviceaccount:default:soc-viewer-sa
```

---

## 🌐 Accès à l'application

Une fois `minikube tunnel` lancé, ouvrez votre navigateur :

```
http://localhost
```

---

<div align="center">

Développé avec ❤️ par **Chahrazed Bouacida**

</div>