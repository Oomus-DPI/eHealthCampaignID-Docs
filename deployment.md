# Guide de déploiement

Oomus CampaignID peut être déployé sur votre propre infrastructure (on-premises, cloud public ou hébergement souverain) ou utilisé en mode SaaS hébergé par Oomus.

> Pour le guide technique complet (Docker Compose, Nginx, secrets, sauvegardes, runbooks), consultez le document interne [`DEPLOYMENT_GUIDE.md`](https://github.com/Oomus-DPI/eHealth-CampaignID-SaaS/blob/main/backend/docs/DEPLOYMENT_GUIDE.md) dans le dépôt.

---

## Options d'hébergement

| Mode | Description | Plans concernés |
|---|---|---|
| **SaaS Oomus** | Hébergé et géré par Oomus — zéro maintenance infrastructure | Tous |
| **Cloud public** | AWS, Azure, GCP, DigitalOcean — déploiement Docker Compose | Regional Ops et au-dessus |
| **Hébergement souverain** | Infrastructure nationale ou régionale dédiée, données en pays | Sovereign Enterprise |

---

## Prérequis minimaux (auto-hébergement)

| Ressource | Minimum | Recommandé |
|---|---|---|
| CPU | 4 vCPU | 8 vCPU |
| RAM | 8 Go | 16 Go |
| Disque | 100 Go SSD | 1 To |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS |
| Base de données | PostgreSQL 15+ | PostgreSQL 15+ (managé) |
| Stockage objet | Local / MinIO | S3-compatible |

---

## Architecture de référence

```
Internet (HTTPS)
       │
  Nginx (reverse proxy + TLS)
       ├── /api  →  FastAPI :8000
       └── /     →  Next.js :3000
                       │
                 PostgreSQL 15
                 Redis 7
                 S3 / MinIO
```

---

## Démarrage rapide (Docker Compose)

```bash
# 1. Cloner le dépôt
git clone https://github.com/Oomus-DPI/eHealth-CampaignID-SaaS.git
cd eHealth-CampaignID-SaaS

# 2. Configurer les variables d'environnement
cp .env.example .env
# Éditer .env — SECRET_KEY obligatoire en production
# openssl rand -hex 32  ← générer une clé sécurisée

# 3. Lancer tous les services
docker compose up -d

# 4. Initialiser la base de données
docker compose exec backend python -m app.db.init_db

# 5. Vérifier
curl http://localhost:8000/health
```

---

## Variables d'environnement essentielles

```env
# Obligatoires en production
SECRET_KEY=<openssl rand -hex 32>
DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/campaignid
ADMIN_EMAIL=admin@votre-organisation.sn
ADMIN_PASSWORD=<mot de passe fort>
APP_ENV=production

# Stockage
STORAGE_BACKEND=s3              # ou "local"
AWS_S3_BUCKET=campaignid-storage

# Services externes (selon activation)
WHATSAPP_API_TOKEN=...
ORANGE_SMS_AUTH_HEADER=...
GOOGLE_WALLET_ISSUER_ID=...
```

---

## Sécurité — Points essentiels

### Ce que vous devez impérativement faire

- [ ] **SECRET_KEY** : générer avec `openssl rand -hex 32` — jamais hardcodé
- [ ] **HTTPS/TLS** : Nginx + Let's Encrypt (renouvellement automatique)
- [ ] **Pare-feu** : bloquer les ports PostgreSQL (5432), Redis (6379), MinIO (9000) sur Internet
- [ ] **Variables d'environnement** : fichiers `.env.prod` jamais commités dans Git
- [ ] **Mot de passe admin** : minimum 16 caractères, majuscule + chiffre + symbole
- [ ] **Sauvegardes** : backup PostgreSQL quotidien, rétention 30 jours

### Ce que vous obtenez par défaut

- **Rate limiting** sur `/auth/login` (10 req/min par IP)
- **JWT HS256** avec expiration 30 min (access) / 30 jours (refresh)
- **2FA TOTP** disponible pour chaque compte (Paramètres → Sécurité)
- **RBAC** — isolation des données par programme
- **Audit trail** immuable — toutes les actions significatives loggées
- **Migrations idempotentes** — `init_db.py` peut être relu sans risque à chaque redémarrage

---

## Checklist avant mise en production

- [ ] `SECRET_KEY` fort (≥ 32 chars hex)
- [ ] `APP_ENV=production`
- [ ] `CORS_ORIGINS` limité aux domaines production
- [ ] Ports 5432 / 6379 fermés sur Internet
- [ ] TLS activé sur Nginx
- [ ] Sauvegardes PostgreSQL automatiques
- [ ] Health check `/health` → 200 OK
- [ ] Compte admin testé
- [ ] Upload logo programme fonctionnel
- [ ] Génération d'une carte de test réussie

---

## Migrations de base de données

Oomus CampaignID n'utilise pas Alembic CLI. Les migrations sont des fonctions `async def migrate_xxx()` dans `app/db/init_db.py`, toutes idempotentes (`ADD COLUMN IF NOT EXISTS`). Elles s'exécutent automatiquement au démarrage du backend.

```bash
# Appliquer manuellement (si nécessaire)
docker compose exec backend python -m app.db.init_db
```

---

## Support déploiement

Pour un accompagnement sur le déploiement souverain, contactez : **contact@oomus.health**
