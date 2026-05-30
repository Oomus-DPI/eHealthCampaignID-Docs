# Guide de déploiement

> **Version** : 5.12 · **Date** : 2026-05-23

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
SECRET_KEY=<openssl rand -hex 64>          # min 64 chars en production
DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/campaignid
ADMIN_EMAIL=admin@votre-organisation.sn
ADMIN_PASSWORD=<mot de passe fort ≥ 12 chars>   # sans valeur par défaut
APP_ENV=production

# ── QR Engine (HMAC + AES) ─────────────────────────────────────
QR_HMAC_SECRET=<openssl rand -hex 32>
QR_AES_KEY=<openssl rand -hex 32>      # 32 bytes = AES-256

# ── Celery / Redis ─────────────────────────────────────────────
REDIS_PASSWORD=<strong password>           # obligatoire (--requirepass)
REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379/0
CELERY_BROKER_URL=redis://:${REDIS_PASSWORD}@redis:6379/0
CELERY_RESULT_BACKEND=redis://:${REDIS_PASSWORD}@redis:6379/1

# ── MinIO (plus de défaut minioadmin) ──────────────────────────
MINIO_ROOT_USER=<your_minio_user>
MINIO_ROOT_PASSWORD=<strong password>

# ── Flower (lié à 127.0.0.1) ──────────────────────────────────
FLOWER_USER=<flower_admin>
FLOWER_PASSWORD=<strong password>

# ── Stockage ───────────────────────────────────────────────────
STORAGE_BACKEND=s3              # ou "local"
AWS_S3_BUCKET=campaignid-storage
AWS_S3_REGION=eu-west-1
AWS_ACCESS_KEY_ID=...           # sans valeur par défaut
AWS_SECRET_ACCESS_KEY=...       # sans valeur par défaut

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

- [ ] **SECRET_KEY** : générer avec `openssl rand -hex 64` (min 64 chars) — jamais hardcodé
- [ ] **ADMIN_PASSWORD** : minimum 12 caractères — aucune valeur par défaut, Docker refuse de démarrer sans elle
- [ ] **REDIS_PASSWORD** : obligatoire — Redis configuré avec `--requirepass`
- [ ] **MINIO_ROOT_USER / MINIO_ROOT_PASSWORD** : obligatoires — plus de défaut `minioadmin`
- [ ] **FLOWER_USER / FLOWER_PASSWORD** : obligatoires — Flower lié à `127.0.0.1` uniquement
- [ ] **QR_HMAC_SECRET** : clé dédiée pour la signature des QR — différente de SECRET_KEY
- [ ] **QR_AES_KEY** : clé AES-256 pour le chiffrement des payloads QR
- [ ] **APP_ENV=production** : masque `/docs`, `/redoc`, `/openapi.json`
- [ ] **HTTPS/TLS** : Nginx + Let's Encrypt (renouvellement automatique)
- [ ] **Pare-feu** : bloquer les ports PostgreSQL (5432), Redis (6379), MinIO (9000) sur Internet
- [ ] **Variables d'environnement** : fichiers `.env.prod` jamais commités dans Git
- [ ] **Sauvegardes** : backup PostgreSQL quotidien chiffré, rétention 30 jours

### Ce que vous obtenez par défaut

- **Rate limiting** sur `/auth/login` (10 req/min par IP)
- **JWT HS256** avec expiration **8h** (access) / 30 jours (refresh) — algorithme fixé, token versioning
- **Isolation réseau** — PostgreSQL et Redis sans port externe, MinIO et Flower liés à `127.0.0.1`
- **CORS strict** — méthodes et headers explicitement listés, pas de wildcard
- **Headers sécurité** — `Content-Security-Policy` + HSTS en production
- **Uploads sécurisés** — noms de fichiers sanitisés avec suffixe UUID
- **2FA TOTP** disponible pour chaque compte (Paramètres → Sécurité)
- **RBAC** — isolation complète des données par programme
- **Audit trail** immuable — toutes les actions significatives loggées
- **QR Engine** — signatures HMAC-SHA256 + chiffrement AES-256-GCM
- **Migrations Alembic** idempotentes — sûres au redémarrage
- **DHIS2 Guard** — accès lecture seule, 7 catégories de données sensibles protégées

---

## Checklist avant mise en production

### Infrastructure

- [ ] `SECRET_KEY` fort (≥ 64 chars hex, `openssl rand -hex 64`)
- [ ] `ADMIN_PASSWORD` défini (min 12 chars, sans défaut)
- [ ] `REDIS_PASSWORD`, `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, `FLOWER_USER`, `FLOWER_PASSWORD` définis
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

## Nouveau dans la v5.14.1

- **Signatures HMAC factures** : `_sign_invoice` utilise désormais `hmac.new(SECRET_KEY, ...)` — les signatures sont vérifiables et résistantes à la falsification. Les factures émises avant v5.14.1 portent une signature SHA-256 legacy (non-HMAC).
- **Validation admin prix PVC** : `PUT /pvc/admin/prices` valide désormais `lead_days >= 0` et `min_qty > 0` — un délai négatif ne peut plus générer une date de livraison dans le passé.
- **Audit log `verify_platform.create`** restauré dans `billing.py` — événement tracé avec `resource_type='verify_platform'`.
- **Correction `sync_plan_modules_for_all`** dans `init_db.py` : commit inconditionnel sur les réactivations de modules (corrige les modules qui restaient `is_active=False` après restart).
- **Nettoyage dead code** : helpers PVC `_get_unit_price` et `_get_lead_days` supprimés, `_PLAN_LEGACY_MAP` promu en constante module-level, regex compilé dans `get_billing_ledger`.

## Nouveau dans la v5.14

- **Billing Infrastructure v6 — Ledger centralisé** : tout débit (plan, module, PVC, génération) génère atomiquement `Transaction + Invoice + AuditLog`. Numéro facture `INV-{TYPE}-{YYYYMM}-{6chars}`. Onglet "Registre comptable" dans BillingPage.
- **PVC Cartes Physiques v2** : deux types d'impression (`standard_pvc` / `offset_industriel`), prix configurables admin, timeline horizontale 5 étapes avec `status_history` JSON, `estimated_delivery_at` automatique.
- **BillingPage v6** : 7 onglets, QuoteModal deux parcours (débit auto / devis institutionnel), usage réel `GET /billing/my-usage`, AdminBillingCenter 6 onglets.
- **Simulation Engine v2** : 5 modes (`rapid / regional / national / multicountry / sovereign`), approche capability-first, `recommend_plan()` avec confidence score, `PlatformSetting` pour config moteur.
- **Panneau Admin 9 pages** : overview, dhis2, verification-portals, analytics, fraud, campaigns, jobs, pvc, mpi — entièrement documenté dans `docs-public/admin/`.

## Nouveau dans la v5.12

- **CI/CD — Release débloquée** : le job `release` utilise désormais `if: always() && ... && needs.docker-build.result != 'failure'` — il ne peut plus être skippé automatiquement si `docker-build` est lent ou en attente. Un push de tag `v*.*.*` crée toujours la GitHub Release dès que ce critère est satisfait
- **CI/CD — Déploiement préflight** : le job `deploy` vérifie en amont si `DEPLOY_HOST` et `DEPLOY_SSH_KEY` sont configurés. Si absent, le job s'affiche comme ignoré avec un récapitulatif des secrets à configurer (plus de crash SSH)
- **Données réelles garanties** : l'endpoint `/analytics/studio-stats` expose désormais `dhis2_cards` depuis `EngineUsageRecord` (source de vérité DB persistante). Le fallback `programme.cards_generated` ne peut plus afficher de données seedées. Le compte démo est initialisé avec `cards_generated=0`
- **DHIS2 FORMAT DE SORTIE** : le sélecteur Boarding Pass / Wallet Pass est remplacé par Boarding Pass uniquement — format standard impression PVC / PDF

## Nouveau dans la v5.9

- **Durcissement sécurité complet** : isolation réseau Docker (PostgreSQL/Redis sans port externe, MinIO/Flower sur 127.0.0.1), nouvelles variables obligatoires (`REDIS_PASSWORD`, `MINIO_ROOT_USER/PASSWORD`, `FLOWER_USER/PASSWORD`), CORS strict, CSP header, HSTS, docs API masquées en production
- **Access token réduit à 8h** : `ACCESS_TOKEN_EXPIRE_MINUTES=480` (était 1440)
- **Token versioning JWT** : changement de mot de passe invalide tous les tokens existants
- **Politique de mots de passe renforcée** : min 10 chars + complexité pour les utilisateurs ; min 12 chars pour l'admin
- **Sovereign card `SovereignCardConfig`** : dataclass configurable (couleurs, font scale, max attributs), icône empreinte digitale, format 1011×375 px @ 300 DPI

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

Pour un accompagnement sur le déploiement souverain, contactez : **ceo@oomus.org**
