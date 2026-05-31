# Guide de déploiement

> **Version** : 5.16 · **Date** : 2026-05-31

Oomus CampaignID peut être déployé sur votre propre infrastructure (on-premises, cloud public ou hébergement souverain) ou utilisé en mode SaaS hébergé par Oomus.

---

## Options d'hébergement

| Mode | Description | Plans concernés |
|---|---|---|
| **SaaS Oomus** | Hébergé et géré par Oomus — zéro maintenance infrastructure | Tous |
| **Cloud public** | AWS, Azure, GCP, DigitalOcean | Regional Ops et au-dessus |
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
| Stockage objet | Local | S3-compatible |
| Cache | Redis 7.x | Redis 7.x (managé) |

---

## Architecture de référence

```
Internet (HTTPS)
       │
  Nginx (reverse proxy + TLS)
       ├── /api  →  API Backend
       └── /     →  Interface Frontend
                       │
                 Base de données
                 Cache Redis
                 Stockage objet
                       │
                 Workers asynchrones
                 (génération de cartes)
```

---

## Services

| Service | Rôle |
|---|---|
| Reverse proxy | TLS, rate limiting, routage |
| API Backend | Logique métier, endpoints REST |
| Frontend | Interface utilisateur Next.js |
| Base de données | Données persistantes |
| Cache | Broker de tâches + cache |
| Workers | Génération asynchrone de cartes |
| Stockage objet | Fichiers générés (S3-compatible) |

---

## Démarrage rapide (Docker Compose)

```bash
# 1. Cloner le dépôt
git clone https://github.com/Oomus-DPI/eHealth-CampaignID-SaaS.git
cd eHealth-CampaignID-SaaS

# 2. Configurer les variables d'environnement
cp .env.example .env
# Éditer .env avec vos valeurs de production

# 3. Lancer tous les services
docker compose up -d

# 4. Appliquer les migrations
docker compose exec backend alembic upgrade head

# 5. Initialiser la base de données
docker compose exec backend python -m app.db.init_db

# 6. Vérifier
curl http://localhost:8000/health
```

---

## Variables d'environnement essentielles

Les variables sensibles ne doivent **jamais** être committées dans Git. Utilisez un gestionnaire de secrets (HashiCorp Vault, AWS Secrets Manager, GitHub Secrets) en production.

```env
# ── Obligatoires ────────────────────────────────────────────
APP_ENV=production
DATABASE_URL=postgresql+asyncpg://user:pass@db/campaignid
ADMIN_EMAIL=admin@votre-organisation.sn
NEXT_PUBLIC_API_URL=https://api.votredomaine.sn
CORS_ORIGINS=https://app.votredomaine.sn

# ── Clés cryptographiques (générer avec openssl rand -hex 32+) ──
SECRET_KEY=<clé forte générée — voir documentation>
# Clés additionnelles pour les modules QR et messagerie
# Voir .env.example pour la liste complète

# ── Services tiers (selon activation) ───────────────────────
WHATSAPP_API_TOKEN=<token Meta>
ORANGE_SMS_AUTH_HEADER=<header Orange>
GOOGLE_WALLET_ISSUER_ID=<issuer ID>
```

> Le fichier `.env.example` à la racine du dépôt liste toutes les variables disponibles avec leur description.

---

## Migrations Alembic

Depuis la v5.0, Oomus CampaignID utilise **Alembic** pour les migrations de schéma. Chaque migration est idempotente et versionnée.

```bash
# Appliquer toutes les migrations en attente
alembic upgrade head

# Voir l'état actuel
alembic current

# Historique des révisions
alembic history --verbose

# Rollback d'une révision
alembic downgrade -1
```

---

## Checklist avant mise en production

### Sécurité

- [ ] Toutes les clés cryptographiques générées avec un CSPRNG (ex. `openssl rand`)
- [ ] Mot de passe administrateur fort défini — aucune valeur par défaut
- [ ] `APP_ENV=production` (masque la documentation API)
- [ ] `CORS_ORIGINS` limité aux domaines de production
- [ ] Services internes (base de données, cache) non exposés sur Internet
- [ ] TLS activé (Let's Encrypt ou certificat institutionnel)
- [ ] Sauvegardes PostgreSQL automatiques (rétention ≥ 30 jours)
- [ ] Fichiers `.env` exclus du dépôt Git

### Base de données

- [ ] `alembic upgrade head` appliqué avec succès
- [ ] `python -m app.db.init_db` exécuté (tables + compte admin)
- [ ] Compte admin testé avec 2FA activé

### Fonctionnel

- [ ] Health check `/health` → `{"status": "ok"}`
- [ ] Génération d'une carte de test réussie
- [ ] Portail de vérification accessible publiquement
- [ ] Distribution WhatsApp ou SMS testée (si activée)

---

## Mise à jour

```bash
# Workflow de mise à jour standard
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d --no-deps backend
docker compose exec backend alembic upgrade head

# Vérifier l'état post-migration
docker compose exec backend alembic current
curl https://mondomaine.sn/health
```

---

## Support déploiement

Pour un accompagnement sur le déploiement souverain, contactez : **ceo@oomus.org**
