# Technical Architecture — OOMUS eHealth CampaignID v4.2

## Vue d'ensemble

```text
┌──────────────────────────────────────────────────────────────────┐
│  Next.js 15 Frontend (port 3000)                                 │
│  AuthContext → api.ts (typed HTTP client) → hooks.ts            │
│  AppLayout → Sidebar → 24 pages                                 │
│  Recharts · Lucide Icons · Onest + IBM Plex Mono                │
└───────────────────────┬──────────────────────────────────────────┘
                        │ HTTP REST + WebSocket
┌───────────────────────▼──────────────────────────────────────────┐
│  FastAPI 0.115 (port 8000) — /api/v1/                            │
│  auth · campaigns · jobs · billing · admin · security            │
│  dhis2 · distribution · verify · sms · studio                   │
│  simulation · analytics · fraud · pvc · quota-plans             │
└──────┬───────────────────────┬───────────────────────────────────┘
       │                       │
┌──────▼──────┐   ┌────────────▼──────────┐   ┌───────────────────┐
│ PostgreSQL  │   │  Redis (Celery broker) │   │  MinIO / S3       │
│ 25+ tables  │   │  + résultats tasks     │   │  PDFs, ZIPs,      │
└─────────────┘   └──────┬────────┬────────┘   │  verify bundles,  │
                         │        │             │  proformas        │
             ┌───────────▼──┐  ┌──▼──────────┐ └───────────────────┘
             │ Celery Worker │  │ Celery Beat │
             │ generate_cards│  │ dhis2_sync_ │
             │ dhis2_cards() │  │ master()    │
             └───────────────┘  └────────────┘
                                     │
              ┌──────────────────────┴──────────────────┐
              │  Services externes                       │
              │  Meta Graph API (WhatsApp v25.0)         │
              │  Orange SMS API (OAuth2, West Africa)    │
              │  Google Wallet API (Generic Pass RS256)  │
              │  DHIS2 Tracker REST API                  │
              └─────────────────────────────────────────┘
```

---

## Stack technique

| Couche | Technologies |
| ------ | ------------ |
| Frontend | Next.js 15, TypeScript 5, inline styles, Recharts, Lucide Icons |
| Backend | FastAPI 0.115, SQLAlchemy 2 (async), Pydantic v2 |
| Base de données | PostgreSQL 16 (asyncpg) — 25+ tables, BigInteger pour montants |
| Queue | Celery 5 + Redis 7 (broker + backend) |
| Scheduler | Celery Beat — synchronisation DHIS2 automatique |
| Stockage | MinIO (dev) / AWS S3 / DigitalOcean Spaces (prod) |
| Génération cartes | Pillow, qrcode, ReportLab, scikit-learn, cryptography |
| Documents | ReportLab A4 (PDF proforma), openpyxl 4 feuilles (Excel) |
| Google Wallet | Generic Pass JWT RS256 |
| WhatsApp | Meta Graph API v25.0 — Cloud API |
| SMS | Orange SMS API OAuth2 (West Africa) |
| Auth | JWT HS256 + Refresh Token 30 j, bcrypt |
| Monitoring | Flower, WebSocket temps réel |

---

## Principes techniques

- **SQLAlchemy 2 async** pour toutes les opérations API — aucun blocage I/O.
- **Tâches longues hors cycle HTTP** — génération, DHIS2 sync, distribution via Celery.
- **Artefacts référencés par clé de stockage** — URLs générées à la demande, signées.
- **Multi-tenant par programme** — chaque programme est isolé (données, quotas, balance).
- **Vérification offline** depuis registres cryptographiques (blockchain SHA-256 chainée).
- **Auditabilité totale** — chaque action écrite dans `audit_logs` (immuable).
- **Quota-first billing** depuis v4.2 — abonnement mensuel + overages auto-calculés, aucun prix unitaire exposé.

---

## Modules principaux

### Moteur de génération (`generator_v2.py`)

| Composant | Détail |
| --------- | ------ |
| Codes | Base36 8 chars — 2,8 milliards de combinaisons |
| Sécurité QR | AES-256-GCM — intégrité + confidentialité |
| Registre | Blockchain SHA-256 — chaîne immuable |
| Anomalies | IsolationForest (scikit-learn) |
| Qualité | PNG lossless → PDF FlateDecode, DPI × 1 / 1.4 / 2.0 |
| i18n | FR / EN / WO (Wolof) |
| Reprise | Checkpoint automatique (`--checkpoint`) |

### Moteur de facturation (`billing_engine.py`)

Depuis v4.2, la facturation est entièrement quota-based :

- `QUOTA_PLANS` — 4 plans avec quotas bénéficiaires, SMS, WhatsApp, vérifications, stockage.
- `INFRA_AUTO_RATES` — taux infra fixes ; seul `infra_factor` est configurable par admin par plan.
- `QuotaPlan` table DB — 4 lignes seed, gouvernées depuis l'interface admin.
- `quota_analysis` JSON ajouté aux simulations — usage %, overage costs, overage_risks avec sévérité.

### Moteur de simulation (`simulation_engine.py`)

- `run_simulation()` — calcul complet : abonnement + overages + infra + RBAC + modules premium.
- `build_proforma()` — génère PDF ReportLab A4 OOMUS-branded + Excel openpyxl 4 feuilles.
- `build_provisioning_config()` — snapshot JSON/YAML complet pour provisioning infra.
- Workflow : `draft → computed → submitted → approved / rejected / modification_requested → provisioned`.

### Dashboard temps réel

- Polling 30 s via `setInterval` + `Promise.allSettled` (dégradation gracieuse si un appel échoue).
- `ActivityChart` — Recharts `ComposedChart` : Bar (jobs/mois), Area (cartes/mois), Line (DHIS2/mois).
- Fallback : si le module `advanced_analytics` est inactif, le chart se construit depuis `JobOut[]` directement.

---

## Modèles de données principaux

| Modèle | Table | Description |
| ------ | ----- | ----------- |
| `Programme` | `programmes` | Tenant principal — balance, plan, is_admin |
| `Campaign` | `campaigns` | Campagne avec template YAML et statut |
| `GenerationJob` | `generation_jobs` | Job async avec progression, coût, render_scale |
| `QuotaPlan` | `quota_plans` | 4 plans avec quotas et overage rates |
| `CampaignSimulation` | `campaign_simulations` | Simulation complète avec quota_analysis JSON |
| `ProformaInvoice` | `proforma_invoices` | PDF/Excel/JSON/YAML stockés S3 |
| `PremiumModuleActivation` | `premium_module_activations` | Modules actifs par programme |
| `AuditLog` | `audit_logs` | Journal immuable de toutes les actions |
| `ApprovalRequest` | `approval_requests` | Workflow d'approbation multi-niveaux |
| `EngineUsageRecord` | `engine_usage_records` | Consommation par moteur et période |

---

## Sécurité

- JWT HS256 access (30 min) + refresh (30 jours).
- Tokens WhatsApp chiffrés Fernet AES-256 en base de données.
- QR codes chiffrés AES-256-GCM — aucune donnée médicale en clair.
- `require_module(key)` dependency FastAPI — gating endpoints par module premium.
- RBAC : permissions granulaires par programme, workflows d'approbation jusqu'à 10 niveaux.
- Audit log immuable : acteur, action, ressource, IP, user agent, metadata JSON.

---

## Infrastructure de déploiement recommandée

```text
Internet → Cloudflare (WAF + SSL) → Nginx reverse proxy
    → FastAPI :8000 + Next.js :3000
    → PostgreSQL 16 + Redis 7
    → Celery Workers × 2 + Beat × 1
    → AWS S3 / DigitalOcean Spaces
```

Voir le [Deployment Runbook](../deployment/deployment-runbook.md) pour les détails complets.
