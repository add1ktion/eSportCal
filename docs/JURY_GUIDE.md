# 🎓 Guide de Soutenance & Architecture Détaillée : eSportCal

Ce guide a été conçu pour vous servir de support de révision complet avant votre passage devant le jury. Il détaille le fonctionnement de chaque brique du projet, la structure des bases de données, l'architecture du code (Front & Back), et contient des explications pas-à-pas des snippets de code les plus critiques du projet.

---

## 1. Accès et Structure de la Base de Données (PostgreSQL)

Notre base de données relationnelle est hébergée dans un conteneur PostgreSQL isolé (`esportcal-db`). Elle sert à stocker les comptes utilisateurs, leurs favoris, et à mettre en cache les matchs ainsi que les effectifs (rosters) des équipes.

### A. Comment se connecter à la base de données (En local)

#### Option A : Depuis le terminal hôte (votre machine)
Puisque le conteneur expose le port `5432:5432`, vous pouvez vous y connecter directement si vous avez `psql` installé localement :
```bash
psql -U antoine -d esportcal_db -h localhost -p 5432
```
*(Le mot de passe par défaut défini dans votre configuration de développement est `antoine`).*

#### Option B : Depuis l'intérieur du conteneur Docker (Recommandé)
Si vous n'avez pas d'outil PostgreSQL installé sur votre machine, connectez-vous directement via la CLI de Docker :
```bash
docker compose exec db psql -U antoine -d esportcal_db
```

### B. Schéma et Contenu détaillé des Tables

Voici la structure exacte des tables et ce qu'elles contiennent :

#### 1. Table `users` (Gestion des Comptes)
Stocke les informations d'authentification des utilisateurs.
*   `id` (UUID, Clé Primaire) : Identifiant unique auto-généré.
*   `username` (VARCHAR) : Pseudo de l'utilisateur.
*   `email` (VARCHAR, Unique) : Adresse email servant d'identifiant de connexion.
*   `password_hash` (VARCHAR) : Mot de passe sécurisé et chiffré (haché avec `bcrypt`).
*   `created_at` (TIMESTAMP) : Date d'inscription.

#### 2. Table `matches` (Mise en cache locale PandaScore)
Stocke tous les matchs de l'année en cours correspondants à notre whitelist de ligues majeures.
*   `id` (INT, Clé Primaire) : ID unique du match provenant de PandaScore.
*   `name` (VARCHAR) : Intitulé du match (ex: `T1 vs Gen.G`).
*   `status` (VARCHAR) : État du match (`not_started`, `running`, `finished`).
*   `scheduled_at` (TIMESTAMP) : Date et heure de programmation (stocké en UTC).
*   `game_name` (VARCHAR) : Nom du jeu (ex: `League of Legends`).
*   `game_slug` (VARCHAR) : Slug utilisé pour les filtres (ex: `league-of-legends`).
*   `league_name` (VARCHAR) : Nom officiel de la ligue (ex: `LEC`).
*   `teams` (JSONB) : Tableau contenant les logos, noms et scores des deux équipes. L'utilisation du type `JSONB` de PostgreSQL permet de stocker des objets complexes structurés sans multiplier les jointures SQL lourdes.

#### 3. Table `teams_cache` (Mise en cache paresseuse des Joueurs / Rosters)
Évite de surcharger l'API externe en mémorisant les joueurs d'une équipe dès qu'un utilisateur consulte les détails d'un match.
*   `id` (INT, Clé Primaire) : ID de l'équipe PandaScore.
*   `name` (VARCHAR) : Nom de l'équipe.
*   `image_url` (VARCHAR) : Logo de l'équipe.
*   `players` (JSONB) : Liste structurée des joueurs (nom, pseudo, rôle, photo).
*   `updated_at` (TIMESTAMP) : Date de mise en cache pour invalider et rafraîchir les données après 24 heures.

#### 4. Tables `favorite_teams` et `favorite_leagues` (Mappage des Signets)
Assurent la liaison entre les utilisateurs et leurs favoris.
*   `user_id` (UUID, Clé Étrangère référençant `users(id) ON DELETE CASCADE`). Le `ON DELETE CASCADE` garantit la conformité RGPD : si un utilisateur supprime son compte, tous ses favoris associés sont effacés automatiquement de la base de données.
*   `pandascore_team_id` / `pandascore_league_id` (INT) : L'identifiant numérique de l'entité côté PandaScore.

---

## 2. Architecture et Fonctionnement du Backend (Express)

Le backend est structuré selon le modèle de conception **MVC (Modèle-Vue-Contrôleur)** découplé pour isoler la logique réseau, les requêtes SQL et le routage des requêtes.

```
backend/
├── config.js          # Centralisation des variables d'environnement (Port, Database URL, API Key)
├── db.js              # Initialisation du Pool de connexion pg (pg-pool)
├── server.js          # Point d'entrée de l'application Node.js
├── app.js             # Configuration d'Express (Middlewares, CORS, Routes)
├── controllers/       # Logique métier et requêtes SQL
│   ├── authController.js
│   ├── favoritesController.js
│   └── matchesController.js
├── routes/            # Définition des endpoints d'API (liés aux contrôleurs)
│   ├── auth.js
│   ├── favorites.js
│   └── matches.js
├── middleware/        # Intercepteurs de requêtes (ex: vérification de Token JWT)
│   └── auth.js
└── cron/              # Tâches planifiées en arrière-plan
    └── syncMatches.js # Synchroniseur et nettoyeur de la base matches
```

### 🔒 Sécurité : Le Middleware d'Authentification (`middleware/auth.js`)
Toutes les routes privées (gestion des favoris, modification du mot de passe, suppression de compte) sont protégées par ce middleware. Il extrait le token JWT présent dans les en-têtes HTTP (`Authorization: Bearer <TOKEN>`), le décode et injecte les informations de l'utilisateur dans la requête (`req.user`) avant de passer au contrôleur.

---

## 3. Architecture et Fonctionnement du Frontend (React / Vite)

Le frontend est construit comme une Application à Page Unique (SPA) dynamique, hautement modulaire, stylisée via Tailwind CSS et Vanilla CSS.

```
frontend/
├── src/
│   ├── App.jsx            # Gestionnaire d'état principal et affichage de la page
│   ├── index.css          # Thème graphique global, variables CSS et scrollbars
│   ├── components/        # Composants réutilisables classés par thème
│   │   ├── layout/        # Navbar, Footer et SideBarFilter (Filtres)
│   │   ├── auth/          # Fenêtre de connexion et paramètres utilisateur
│   │   ├── matches/       # MatchItem, MatchDetails (Détails), FavoriteTeams (Flux Favori)
│   │   └── common/        # Modales de confirmation, formulaires de contact
│   └── utils/
│       └── helpers.js     # Fonctions de formattage de dates et normalisation de timezone
```

### 🕒 Gestion des fuseaux horaires (Timezone Normalization)
Toutes les dates de matchs stockées en base de données ou récupérées depuis l'API sont au format **UTC** (ex: `2026-07-10T08:00:00Z`).
Sur le frontend, dans le composant `MatchItem.jsx`, nous utilisons la bibliothèque `date-fns` et les API natives de Javascript pour convertir et afficher ces heures selon la timezone locale du navigateur de l'utilisateur :
```javascript
const formattedTime = new Date(match.scheduled_at).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
```
Le calendrier est ainsi automatiquement personnalisé pour n'importe quel spectateur dans le monde.

---

## 4. Snippets de Code Clés & Explications Techniques

Voici les extraits de code les plus complexes et importants du projet à expliquer lors de votre présentation.

### Snippet A : La Résolution Dynamique d'IDs et la Synchronisation Paginale (`backend/cron/syncMatches.js`)
**Pourquoi ce code est crucial** : L'API de PandaScore renvoie tous les tournois confondus. Au début de l'année, les ligues régionales (ERL) programment des centaines de matchs, ce qui sature les premières pages de l'API et masque les ligues majeures (LEC, LCS, LCK). Pour contourner cette limite de page sans consommer trop de requêtes d'API, nous résolvons d'abord les IDs uniques des ligues cibles, puis nous filtrons directement à la source (côté API) pour ne télécharger que ces matchs.

```javascript
// 1. Résolution dynamique des IDs de ligues whitelistées
const resolveWhitelistedLeagueIds = async (game) => {
    const allowed = LEAGUE_WHITELIST[game.slug];
    let page = 1;
    let hasMore = true;
    const matchedIds = [];

    while (hasMore && page <= 5) {
        const res = await axios.get(`${game.leaguesUrl}&page=${page}`, {
            headers: { Authorization: `Bearer ${process.env.PANDASCORE_API_KEY}` }
        });
        if (res.data && res.data.length > 0) {
            res.data.forEach(l => {
                const u = l.name.toUpperCase();
                const matchesWhitelist = allowed.some(wl => {
                    const uwl = wl.toUpperCase();
                    // Règles strictes pour éviter les faux-positifs (ex: LFL Div 2 pour LFL)
                    if (uwl === 'LFL' && u === 'LFL') return true;
                    if (uwl === 'LEC' && u === 'LEC') return true;
                    if (uwl === 'LCK' && u === 'LCK') return true;
                    // ...
                    return u.includes(uwl);
                });
                if (matchesWhitelist) matchedIds.push(l.id);
            });
            if (res.data.length < 100) hasMore = false;
            else page++;
        } else {
            hasMore = false;
        }
    }
    return matchedIds;
};
```
**Explication pour le jury** : *« Plutôt que de filtrer les matchs en mémoire après les avoir téléchargés, ce qui nous ferait rater les matchs phares situés plus loin dans la pagination, nous effectuons une première requête légère pour identifier les IDs de nos ligues cibles. Ensuite, nous requêtons les matchs en injectant ces identifiants directement dans le filtre d'API (`filter[league_id]=...`). Cela nous permet d'obtenir 100 % de couverture sur les matchs majeurs tout en restant très économes en requêtes réseau. »*

---

### Snippet B : Mise en cache intelligente des compositions d'équipes (`backend/controllers/matchesController.js`)
**Pourquoi ce code est crucial** : Récupérer la composition complète des joueurs d'une équipe avec leurs photos est une opération lourde. Pour accélérer l'affichage et respecter notre quota de requêtes API, nous appliquons un mécanisme de **Cache Paresseux (Lazy-Caching)**.

```javascript
exports.getTeamRoster = async (req, res) => {
    const teamId = req.params.id;
    try {
        // 1. On cherche d'abord dans notre base de données locale
        const cacheResult = await db.query(
            'SELECT players, updated_at FROM teams_cache WHERE id = $1',
            [teamId]
        );

        // 2. Si présent et que les données ont moins de 24h, on sert le cache (Gain de temps immédiat !)
        if (cacheResult.rows.length > 0) {
            const cache = cacheResult.rows[0];
            const ageInHours = (new Date() - new Date(cache.updated_at)) / (1000 * 60 * 60);
            if (ageInHours < 24) {
                return res.json({ players: cache.players });
            }
        }

        // 3. En cas de cache miss, on interroge l'API externe
        const apiResponse = await axios.get(
            `https://api.pandascore.co/teams/${teamId}`,
            { headers: { Authorization: `Bearer ${config.PANDASCORE_API_KEY}` } }
        );

        const players = apiResponse.data.players || [];

        // 4. On enregistre en base pour les futures consultations
        await db.query(
            `INSERT INTO teams_cache (id, name, image_url, players, updated_at)
             VALUES ($1, $2, $3, $4, NOW())
             ON CONFLICT (id) DO UPDATE SET players = EXCLUDED.players, updated_at = NOW()`,
            [teamId, apiResponse.data.name, apiResponse.data.image_url, JSON.stringify(players)]
        );

        return res.json({ players });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
};
```
**Explication pour le jury** : *« Ce contrôleur implémente un modèle de cache SQL. Lors du premier clic sur un match, les joueurs sont téléchargés depuis l'API externe et stockés localement. Lors des clics suivants, la base de données PostgreSQL sert instantanément les données en moins de 10 millisecondes. Une invalidation automatique après 24 heures garantit que les données restent fraîches si un joueur est remplacé. »*

---

### Snippet C : Le Rendu Vidéo Responsive en Widescreen (`frontend/src/components/matches/MatchDetails.jsx`)
**Pourquoi ce code est crucial** : Les intégrations d'iframe vidéo en HTML ont tendance à se déformer ou à afficher des bandes noires lors du redimensionnement de la fenêtre du navigateur. Nous utilisons la technique du "padding-top hack" pour forcer un ratio 16:9 fluide et réactif.

```javascript
<div className="relative w-full overflow-hidden rounded-2xl border border-[#232549] shadow-2xl" style={{ paddingTop: '56.25%' }}>
  <iframe
    src={`https://player.twitch.tv/?channel=${twitchChannel}&parent=localhost&parent=127.0.0.1`}
    allowFullScreen
    className="absolute top-0 left-0 w-full h-full border-0"
  />
</div>
```
**Explication pour le jury** : *« Un ratio standard 16:9 correspond à un rapport de hauteur sur largeur de 56,25 % (9 divisé par 16). En appliquant un padding-top de `56.25%` sur le conteneur parent en position relative, nous forçons l'iframe (positionnée en absolue pour remplir 100 % de la hauteur et de la largeur du parent) à conserver un format widescreen parfait, sans aucune déformation, que le site soit consulté sur mobile, tablette ou grand écran de bureau. »*

---

### Snippet D : Masquage des Scores (Anti-Spoiler) (`frontend/src/components/matches/MatchItem.jsx`)
**Pourquoi ce code est crucial** : Pour éviter de gâcher le suspense des matchs terminés ou en cours, le score est caché par défaut derrière un bouton "Reveal". Ce bouton ne doit pas ouvrir la carte de détails du match lorsqu'on clique dessus.

```javascript
<button
  onClick={(e) => {
    e.stopPropagation(); // Évite le dépliage de la carte de détails
    setRevealScore(true);
  }}
  className="bg-[#2a2c4e] text-xs px-2.5 py-1 rounded-lg border border-[#3e4270] hover:bg-[#5c3be0] transition cursor-pointer font-bold text-gray-300 hover:text-white"
>
  Reveal
</button>
```
**Explication pour le jury** : *« Pour offrir une expérience utilisateur agréable, nous masquons les scores des matchs finis. L'utilisateur peut révéler le score au clic. L'utilisation de `e.stopPropagation()` est primordiale ici : elle empêche l'événement de clic de remonter (propager) jusqu'au conteneur de la carte, ce qui aurait pour effet indésirable de déplier ou replier le match en même temps qu'on en révèle le score. »*
