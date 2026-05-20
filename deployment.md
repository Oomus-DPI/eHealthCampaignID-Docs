# Guide de déploiement

> **Version** : 5.6 · **Date** : 2026-05-19

Oomus CampaignID peut être déployé sur votre propre infrastructure (on-premises, cloud public ou hébergement souverain) ou utilisé en mode SaaS hébergé par Oomus.

> Pour le guide technique complet (Docker Compose, Nginx, secrets, sauvegardes, runbooks), consultez [`backend/docs/DEPLOYMENT_GUIDE.md`](https://github.com/Oomus-DPI/eHealth-CampaignID-SaaS/blob/main/backend/docs/DEPLOYMENT_GUIDE.md) dans le dépôt.

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
| Base de données | PostgreSQL 16+ | PostgreSQL 16+ (managé) |
| Stockage objet | Local / MinIO | S3-compatible |
| Redis | 7.x | 7.x (managé) |

---

## Architecture de référence

```
Internet (HTTPS)
       │
  Nginx (reverse proxy + TLS)
       ├── /api  →  FastAPI :8000
       └── /     →  Next.js :3000
                       │
                 PostgreSQL 16
                 Redis 7
                 S3 / MinIO
                       │
                 Celery Worker
                 (génération asynchrone)
```

---

## Services Docker

| Service | Image | Rôle |
|---|---|---|
| `nginx` | nginx:1.25-alpine | Reverse proxy, TLS, rate limiting |
| `backend` | python:3.12-slim | API FastAPI + uvicorn |
| `frontend` | node:20-alpine | Interface Next.js 16.2.4 |
| `db` | postgres:16-alpine | Données persistantes |
| `redis` | redis:7-alpine | Broker Celery + cache |
| `celery` | (même image que backend) | Workers de génération asynchrone |
| `flower` | (optionnel) | Monitoring workers Celery |
| `minio` | minio/minio | Stockage S3-compatible (dev/staging) |

---

## Démarrage rapide (Docker Compose)

```bash
# 1. Cloner le dépôt
git clone https://github.com/Oomus-DPI/eHealth-CampaignID-SaaS.git
cd eHealth-CampaignID-SaaS

# 2. Configurer les variables d'environnement
cp .env.example .env
# Éditer .env — SECRET_KEY et QR_HMAC_SECRET obligatoires en production
openssl rand -hex 32   # → SECRET_KEY
openssl rand -hex 32   # → QR_HMAC_SECRET

# 3. Lancer tous les services
docker compose up -d

# 4. Appliquer les migrations Alembic
docker compose exec backend alembic upgrade head

# 5. Initialiser la base de données (tables de base + admin)
docker compose exec backend python -m app.db.init_db

# 6. Vérifier
curl http://localhost:8000/health
```

---

## Variables d'environnement essentielles

```env
# ── Obligatoires en production ─────────────────────────────────
SECRET_KEY=<openssl rand -hex 32>
DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/campaignid
ADMIN_EMAIL=admin@votre-organisation.sn
ADMIN_PASSWORD=<mot de passe fort ≥ 16 chars>
APP_ENV=production

# ── QR Engine (HMAC + AES) ─────────────────────────────────────
QR_HMAC_SECRET=<openssl rand -hex 32>
QR_AES_KEY=<openssl rand -hex 32>      # 32 bytes = AES-256

# ── Celery / Redis ─────────────────────────────────────────────
REDIS_URL=redis://:PASSWORD@redis:6379/0
CELERY_BROKER_URL=redis://:PASSWORD@redis:6379/0
CELERY_RESULT_BACKEND=redis://:PASSWORD@redis:6379/1

# ── Stockage ───────────────────────────────────────────────────
STORAGE_BACKEND=s3              # ou "local"
AWS_S3_BUCKET=campaignid-storage
AWS_S3_REGION=eu-west-1
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...

# ── Frontend ───────────────────────────────────────────────────
NEXT_PUBLIC_API_URL=https://api.votredomaine.sn

# ── Services externes (selon activation) ───────────────────────
WHATSAPP_API_TOKEN=...
WHATSAPP_PHONE_NUMBER_ID=...
ORANGE_SMS_AUTH_HEADER=...
GOOGLE_WALLET_ISSUER_ID=...
GOOGLE_WALLET_SERVICE_ACCOUNT_JSON=...
```

---

## Migrations Alembic

Depuis la v5.0, Oomus CampaignID utilise **Alembic** pour les migrations de schéma. Chaque migration est idempotente et versionnée.

### Migrations incluses

| Révision | Description |
|---|---|
| `0001_safe_architecture_tables` | Tables de base (safe v5.0) |
| `0002_studio_print_quota_columns` | Colonnes quota impression Studio |
| `0003_premium_module_price_config` | Configuration prix modules premium |
| `0004_role_permission_configs` | Tables de permissions RBAC |
| `0005_programme_logo_url` | Colonne `logo_url` sur `programmes` |

### Commandes Alembic

```bash
# Appliquer toutes les migrations en attente
alembic upgrade head

# Voir l'état actuel
alembic current

# Historique des révisions
alembic history --verbose

# Rollback d'une révision
alembic downgrade -1

# Créer une nouvelle migration
alembic revision --autogenerate -m "description_courte"
```

---

## Sécurité — Points essentiels

### Ce que vous devez impérativement faire

- [ ] **SECRET_KEY** : générer avec `openssl rand -hex 32` — jamais hardcodé
- [ ] **QR_HMAC_SECRET** : clé dédiée pour la signature des QR — différente de SECRET_KEY
- [ ] **QR_AES_KEY** : clé AES-256 pour le chiffrement des payloads QR
- [ ] **HTTPS/TLS** : Nginx + Let's Encrypt (renouvellement automatique)
- [ ] **Pare-feu** : bloquer les ports PostgreSQL (5432), Redis (6379), MinIO (9000) sur Internet
- [ ] **Variables d'environnement** : fichiers `.env.prod` jamais commités dans Git
- [ ] **Mot de passe admin** : minimum 16 caractères, majuscule + chiffre + symbole
- [ ] **Sauvegardes** : backup PostgreSQL quotidien chiffré, rétention 30 jours

### Ce que vous obtenez par défaut

- **Rate limiting** sur `/auth/login` (10 req/min par IP)
- **JWT HS256** avec expiration 480 min (access) / 30 jours (refresh)
- **2FA TOTP** disponible pour chaque compte (Paramètres → Sécurité)
- **RBAC** — isolation complète des données par programme
- **Audit trail** immuable — toutes les actions significatives loggées
- **QR Engine** — signatures HMAC-SHA256 + chiffrement AES-256-GCM
- **Migrations Alembic** idempotentes — sûres au redémarrage
- **DHIS2 Guard** — accès lecture seule, 7 catégories de données sensibles protégées

---

## Checklist avant mise en production

### Infrastructure

- [ ] `SECRET_KEY` fort (≥ 32 chars hex, `openssl rand -hex 32`)
- [ ] `QR_HMAC_SECRET` et `QR_AES_KEY` générés séparément
- [ ] `APP_ENV=production`
- [ ] `CORS_ORIGINS` limité aux domaines production
- [ ] Ports 5432 / 6379 / 9000 fermés sur Internet
- [ ] TLS activé sur Nginx (Let's Encrypt ou certificat institutionnel)
- [ ] Sauvegardes PostgreSQL automatiques (cron quotidien)
- [ ] Worker Celery actif (`celery -A app.celery worker`)

### Base de données

- [ ] `alembic upgrade head` appliqué avec succès
- [ ] `python -m app.db.init_db` exécuté (tables de base + compte admin)
- [ ] Compte admin testé (connexion + 2FA activé)

### Fonctionnel

- [ ] Health check `/health` → `{"status": "ok"}`
- [ ] Upload logo programme fonctionnel
- [ ] Génération d'une carte de test réussie (job Celery complété)
- [ ] Portail de vérification `/verify?p=…&s=…` accessible publiquement
- [ ] Distribution WhatsApp ou SMS testée (si activée)
- [ ] Sovereign Wallet : création d'un pass test (si activé)

---

## Migrations de base de données

```bash
# Workflow de mise à jour standard
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d --no-deps backend
docker compose exec backend alembic upgrade head
# Les migrations sont idempotentes — sûres à relancer

# Vérifier l'état post-migration
docker compose exec backend alembic current
curl https://mondomaine.sn/health
```

---

## Nouveau dans la v5.6

- **Redesign UI mature** — Page Connexion et page Campagnes refaites : esthétique minimaliste sobre (fond `#080C14`), suppression des animations décoratives, table épurée pour les campagnes
- **Campagnes sans affichage des coûts** — Les coûts ne s'affichent plus par défaut dans la liste des campagnes ; uniquement signalés en cas de dépassement de quota
- **Persistance logo programme** — `POST /api/v1/auth/logo` encode l'image en base64 et la stocke en colonne `TEXT` (`logo_url`). Aucun stockage fichier requis. La sidebar et le profil affichent automatiquement le logo réel
- **2FA TOTP corrigée** — Champ `two_factor_enabled` (et non `totp_enabled`) aligné entre le modèle SQLAlchemy, le schéma Pydantic (`ProgrammeOut`) et les 3 endpoints 2FA. `totp_secret` déclaré en `Mapped[Optional[str]]`
- **Avatar profil** — `SettingsPage` et `Sidebar` utilisent `programme.logo_url` pour afficher le vrai logo, avec fallback sur les initiales

## Nouveau dans la v5.4

- **Panneau Admin complet** — 8 pages de gouvernance entièrement fonctionnelles (DHIS2, Portails, Analytics, Fraude, Campagnes, Jobs, PVC, MPI)
- **Template Sovereign** — Nouveau modèle de carte navy/or premium (boarding pass)
- **Portail de vérification public** — Route Next.js standalone `/verify` sans authentification
- **WalletPassVerifyPage** — Vérification de pass in-app (wallet-verify)
- **Migrations Alembic 0002–0005** — Colonnes Studio, prix modules premium, RBAC, logo_url programme
- **Interface de connexion modernisée** — Animations dynamiques, statistiques en temps réel, sélecteur mode

---

## Support déploiement

Pour un accompagnement sur le déploiement souverain, contactez : **contact@oomus.health**
