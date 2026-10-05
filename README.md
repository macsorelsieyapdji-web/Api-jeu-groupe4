# API Jeux

API REST d'un catalogue de jeux vidéo (jeux, éditeurs, comptes utilisateurs), écrite avec FastAPI, SQLAlchemy et Pydantic. Elle s'adresse aux développeurs du front et aux équipes qui reprennent le projet.

## Prérequis

- Python 3.12 ou plus (testé avec 3.12 et 3.13)
- Git
- Aucune base de données à installer : le démarrage rapide utilise SQLite

## Démarrage rapide

Les commandes ci-dessous se lancent dans un terminal, à la racine du dépôt.

**Windows (PowerShell)**

```powershell
git clone https://github.com/macsorelsieyapdji-web/Api-jeu-groupe4.git
cd Api-jeu-groupe4
python -m venv .venv
Set-ExecutionPolicy -Scope Process Bypass
.venv\Scripts\activate
pip install -r requirements-dev.txt
copy .env.example .env
```

**macOS / Linux**

```bash
git clone https://github.com/macsorelsieyapdji-web/Api-jeu-groupe4.git
cd Api-jeu-groupe4
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
cp .env.example .env
```

Ouvrez ensuite le fichier `.env` et remplacez ces deux lignes :

```
DATABASE_URL=sqlite:///./jeux.db
CLE_SECRETE=remplacez-moi
```

Puis remplissez la base avec le catalogue de démonstration et lancez l'API :

```bash
python scripts/peupler.py
fastapi dev app/main.py
```

Résultat attendu :

- `peupler.py` affiche la création d'un administrateur et de 8 jeux.
- `http://127.0.0.1:8000` répond `{"message": "API opérationnelle", ...}`.
- `http://127.0.0.1:8000/docs` affiche la liste des routes.

Le compte de démonstration est `admin@example.com` avec le mot de passe `motdepasse123`. Il n'existe que dans la base locale créée par `peupler.py`.

## Configuration

L'API lit ses réglages dans le fichier `.env` (modèle : `.env.example`). Sans les deux variables obligatoires, elle refuse de démarrer avec une erreur de validation.

| Variable | Rôle | Obligatoire | Défaut |
|---|---|---|---|
| `DATABASE_URL` | Adresse de la base. SQLite : `sqlite:///./jeux.db`. PostgreSQL : `postgresql+psycopg://utilisateur:motdepasse@hote:5432/base` | Oui | aucun |
| `CLE_SECRETE` | Clé qui signe les jetons de connexion. Qui la détient peut forger un jeton d'administrateur | Oui | aucun |
| `ALGORITHME_JETON` | Algorithme de signature des jetons | Non | `HS256` |
| `DUREE_JETON_MINUTES` | Durée de validité d'un jeton | Non | `30` |
| `ORIGINES_AUTORISEES` | Origines autorisées par CORS, séparées par des virgules | Non | `http://localhost:5173` |
| `ENVIRONNEMENT` | `developpement` ou `production` | Non | `developpement` |
| `NIVEAU_JOURNAL` | Niveau des journaux (`INFO`, `DEBUG`...) | Non | `INFO` |
| `ECHO_SQL` | Affiche chaque requête SQL dans le terminal | Non | `false` |
| `MAX_TENTATIVES_CONNEXION` | Échecs de connexion autorisés par email dans la fenêtre | Non | `5` |
| `FENETRE_TENTATIVES_MINUTES` | Durée de la fenêtre de comptage des échecs | Non | `15` |

Pour générer une vraie clé secrète :

```bash
python -c "import secrets; print(secrets.token_urlsafe(32))"
```

Le fichier `.env` ne se commite jamais : il est dans le `.gitignore`.

## Utilisation

La documentation complète des routes est générée par FastAPI : `http://127.0.0.1:8000/docs`.

Trois exemples (l'API doit tourner) :

```bash
# Lister les jeux (public)
curl "http://127.0.0.1:8000/api/v1/jeux"

# Statistiques du catalogue (public)
curl "http://127.0.0.1:8000/api/v1/jeux/statistiques"

# Obtenir un jeton de connexion
curl -X POST "http://127.0.0.1:8000/api/v1/connexion" -d "username=admin@example.com&password=motdepasse123"
```

Sous PowerShell, tapez `curl.exe` à la place de `curl`.

Pour les routes qui modifient des données, envoyez le jeton reçu dans l'en-tête `Authorization: Bearer <jeton>`. Dans `/docs`, le bouton **Authorize** fait cette étape.

## Tests

```bash
python -m pytest
ruff check .
```

`pytest` doit afficher uniquement des tests réussis, et `ruff` doit répondre `All checks passed!`. Les tests utilisent leur propre base en mémoire : ils ne touchent pas à `jeux.db`.

## Architecture

Une requête traverse les couches dans un seul sens. Les services et les dépôts n'importent jamais FastAPI : un test (`tests/test_architecture.py`) le vérifie.

```mermaid
flowchart LR
    Client[Client HTTP] -->|requête| Routeurs
    Routeurs -->|appelle| Services
    Services -->|appelle| Depots
    Depots -->|lit et écrit| Tables
    Tables --> Base[(SQLite ou PostgreSQL)]
    Routeurs -.->|valide les entrées et sorties| Modeles
```

| Dossier ou fichier | Rôle |
|---|---|
| `app/routeurs/` | Reçoit la requête HTTP, délègue, renvoie la réponse |
| `app/services/` | Règles métier (droits, titres uniques, validations) |
| `app/depots/` | Requêtes SQL, une fonction par besoin |
| `app/tables/` | Tables de la base (SQLAlchemy) |
| `app/modeles/` | Formats d'entrée et de sortie de l'API (Pydantic) |
| `app/main.py` | Création de l'application, gestion des erreurs |
| `app/config.py` | Lecture et validation du `.env` |
| `scripts/` | Outils en ligne de commande (peupler, importer, exporter) |
| `tests/` | Tests automatiques |
| `donnees/` | Catalogue de démonstration (`jeux.csv`) |

## Contribuer

1. Ouvrez ou choisissez une issue, et assignez-vous.
2. Créez une branche depuis un `main` à jour : `fix/<numéro>-description` ou `feat/<numéro>-description`.
3. Faites de petits commits au format `type(portée): résumé`, avec `Refs #<numéro>`.
4. Ouvrez une pull request avec le contexte, les changements, l'impact et la façon de la tester, et `Closes #<numéro>`.
5. Un autre membre relit. Après approbation et CI verte, fusion par *squash*, puis suppression de la branche.

On ne pousse jamais directement sur `main`.
