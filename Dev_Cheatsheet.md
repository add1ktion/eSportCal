# 🎮 eSportCal - Developer Cheat Sheet

## ⚠️ Rappel important : Pour toutes les commandes Backend, Base de données et Tests, assure-toi d'être dans le dossier backend/ de ton terminal (cd backend).

### 🚀 1. Lancement du Serveur (Local Development)
- Lancer le serveur en mode "Auto-Reload" (Nodemon) :

```Bash
npm run dev
```

- Lancer le serveur en mode normal (Production) :

```Bash
npm start
```

- Arrêter le serveur (dans le terminal) :
`Ctrl + C`

### 🧪 2. Tests & Qualité (QA / DevOps)
- Lancer la suite de tests automatisés (Jest) :

```Bash
npm test
```

- Adresse locale pour tester ton API (Postman / Navigateur) :

http://localhost:5001/api/matches

### 🐘 3. Base de données (PostgreSQL)
- Réinitialiser la base de données locale avec les faux utilisateurs de test :

```Bash
psql -d esportcal_db -f seed.sql
```

(Si ton Mac te demande ton utilisateur Postgres : psql -U ton_nom_mac -d esportcal_db -f seed.sql)

🔀 4. Git Flow (Le rituel quotidien)
Le matin en arrivant (Pour récupérer les derniers changements) :

```Bash
git checkout dev
git pull origin dev
```

- Créer une nouvelle branche pour coder une nouvelle fonctionnalité :

```Bash
git checkout -b feature/nom-de-la-feature
```

- Sauvegarder et envoyer ton travail :

```Bash
git add .
git commit -m "feat: description in english"
git push origin feature/nom-de-la-feature 
```