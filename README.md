# EBoutiqueFree_CICD

## 📦 Description

Ce repository contient l'orchestration CI/CD complète de la plateforme **EBoutiqueFree**.

Il centralise le déploiement des 2 projets applicatifs :

- 🎯 **Backend Django (PostGIS)** : https://github.com/Challenge-Sorbonne-2025/EBoutiqueFree_Backend
- 🎯 **Frontend React (Vite + Nginx)** : https://github.com/Challenge-Sorbonne-2025/EBoutiqueFree_Frontend

Ce repository contient :

- ✅ Un Jenkinsfile global qui déclenche le build des deux projets.
- ✅ Un `docker-compose.yml` global pour lancer l’ensemble des services en local ou en production.
- ✅ Les configurations nécessaires à Jenkins pour l’intégration continue.
==================================================================================================================================================================
- *Exemple de lignes pour faire démarrer les containers apres exécution du pipeline Jenkins:*
```bash
docker run -d -p 2000:80 --name frontend cicdfreeshop-frontend:latest
docker run -d -p 9000:9000 --name backend cicdfreeshop-backend:latest
```

---

## 🗺️ Structure

```bash
EBoutiqueFree_CICD/
│
├── Jenkinsfile           # Pipeline Jenkins complet
├── docker-compose.yml    # Docker Compose global
└── README.md             # Ce fichier
