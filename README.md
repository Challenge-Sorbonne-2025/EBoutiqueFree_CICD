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


---

## 🚀 Fonctionnement du Pipeline

### 🔁 Étapes automatisées par Jenkins :

1. **Clonage des dépôts**
2. **Injection sécurisée du fichier `.env` backend**
3. **Construction des images Docker avec Docker Compose**
4. **Tagging :**
   - `backendboutique-latest`, `backendboutique-<build number>`
   - `frontendboutique-latest`, `frontendboutique-<build number>`
5. **Push des images sur DockerHub**
6. **déploiement sur Google Kubernetes Engine

---

---
  *Guide pour le pull et le run des images:*
  ```bash
docker pull senfidel/projetsvde:frontendboutique-latest
docker pull senfidel/projetsvde:backendboutique-latest

docker run -d -p 2000:80 --name frontend senfidel/projetsvde:frontendboutique-latest
docker run -d -p 9000:9000 --name backend senfidel/projetsvde:backendboutique-latest
```

---

## 🗺️ Structure du projet CICD

```bash
EBoutiqueFree_CICD/
│
├── Jenkinsfile           # Pipeline Jenkins complet
├── docker-compose.yml    # Docker Compose du front & du du back
└── README.md             
