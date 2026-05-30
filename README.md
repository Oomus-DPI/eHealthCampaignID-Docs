# Oomus CampaignID

**Infrastructure souveraine de santé publique numérique pour les programmes nationaux en Afrique et dans les pays à ressources limitées.**

> **Version** : 5.14.1 · **Date** : 2026-05-30 · **Statut** : Production

---

## Bienvenue sur la documentation officielle

Oomus CampaignID est une plateforme GovTech Enterprise SaaS conçue pour la génération, la distribution, la vérification et la gouvernance de cartes d'identité sanitaire numériques sécurisées. Elle repose sur une identité numérique MPI (Master Patient Index) souveraine, interopérable HL7 FHIR R4, et pensée pour fonctionner dans des environnements à faible connectivité.

---

## Ce que fait Oomus CampaignID

Oomus CampaignID permet aux programmes de santé nationaux et aux organisations humanitaires de :

- **Générer** des cartes de santé numériques sécurisées à grande échelle (jusqu'à 10 millions de bénéficiaires)
- **Distribuer** les cartes par WhatsApp, SMS et Google Wallet depuis une interface unifiée
- **Vérifier** l'authenticité des cartes hors ligne — sans connexion Internet requise
- **Dédupliquer** automatiquement les identités grâce au moteur MPI probabiliste
- **Synchroniser** les données depuis DHIS2 Tracker en temps réel ou selon un calendrier configurable
- **Gouverner** les accès, les quotas et les approbations grâce à un système RBAC institutionnel complet
- **Émettre** des Sovereign Wallet Passes — passes digitaux signés HMAC-SHA256, synchronisables hors ligne
- **Produire** des cartes PVC physiques en deux qualités d'impression : Standard PVC et Offset Industriel
- **Facturer** avec une traçabilité complète — chaque débit génère automatiquement une facture formelle signée
- **Administrer** la plateforme entière depuis un panneau admin unifié

---

## Capacités clés

- **Card Studio** — Éditeur visuel avec 11 modèles de cartes dont le template **Sovereign** (navy/or premium), aperçu PNG temps réel, export YAML/JSON, options DPI 300/450/600
- **Identités numériques souveraines** — 1 citoyen = 1 identifiant de santé numérique MPI, inter-programmes, à vie, cross-campagnes
- **Intégration DHIS2** — Synchronisation automatique, mapping d'attributs, génération de cartes depuis les enrollments, guard IA données sensibles
- **Distribution multicanal** — WhatsApp (Meta Graph API v25.0), SMS (Orange API), Google Wallet
- **Vérification hors ligne** — Portail statique, registre SHA-256, WebCrypto, multilingue (FR/EN/WO)
- **Sovereign Wallet** — Passes digitaux signés HMAC-SHA256, bundle offline XOR-SHA256, sync appareils, révocation auditée
- **Simulation financière** — Estimation proforma 6 étapes, workflow d'approbation admin, génération PDF/Excel institutionnel
- **Cartes PVC physiques** — Standard PVC (350 FCFA/carte) et Offset Industriel (650 FCFA/carte), timeline de suivi avec notifications
- **Billing Infrastructure v6** — Registre comptable centralisé : chaque débit génère Transaction + Invoice signée **HMAC-SHA256** (v5.14.1) + AuditLog immuable. Numéro unique `INV-{TYPE}-{YYYYMM}-{6chars}`.
- **Dashboard opérationnel** — KPIs temps réel, analytics avancées, santé de l'infrastructure, alertes
- **Centre Billing admin** — Vue consolidée abonnements, devis, factures, tarification, simulations
- **Panneau Admin v5.14** — 9 pages de gouvernance : programmes, DHIS2, portails vérification, analytics, fraude IA, campagnes, jobs, PVC, MPI souverain
- **UI enterprise v5.14** — Design Stripe-inspired, IBM Plex Sans + Mono, dark hero, responsive, dark mode natif

---

## Pour qui

| Public cible | Cas d'usage typique |
|---|---|
| Programmes nationaux de santé | Vaccination, paludisme, nutrition, HIV/PTME |
| Ministères de la Santé | Carte d'assurance maladie universelle, identité sanitaire nationale |
| Agences humanitaires & ONG | Identification des réfugiés, distribution de moustiquaires (MILD) |
| Agences gouvernementales | Identité nationale reliée aux services de santé |
| Programmes de santé agricole | Carte agriculteur / santé rurale |

---

## Plans disponibles

| Plan | Identités/mois | SMS/mois | WhatsApp/mois | Modules inclus |
|---|---|---|---|---|
| **Essential** (`starter`) | 10 000 | 50 000 | 10 000 | SMS Gateway, Studio |
| **Regional Command** (`regional_ops`) | 100 000 | 250 000 | 100 000 | + WhatsApp, Sync Terrain, Geo, IA |
| **National Infrastructure** (`national_campaign`) | 1 000 000 | 3 000 000 | 1 000 000 | + Anomalies IA, Wallet, MPI, Portail Vérif. |
| **Sovereign Cloud** (`sovereign_enterprise`) | Illimité | Illimités | Illimités | Tous modules + hébergement souverain |

> Les modules inclus sont activés **automatiquement** à la souscription du plan — aucune configuration manuelle requise.

---

## Stack technique

| Composant | Technologie |
|---|---|
| Frontend | Next.js 16 + React 19 + TypeScript 5 · IBM Plex Sans/Mono · Lucide Icons |
| Backend | FastAPI 0.115 + Python 3.12 · SQLAlchemy 2 (async) · Pydantic v2 |
| Base de données | PostgreSQL 16 (asyncpg) — 44+ tables |
| Files de tâches | Celery 5 + Redis 7 |
| Stockage fichiers | MinIO / S3-compatible |
| Interopérabilité | HL7 FHIR R4 |
| Sécurité billing | SHA-256 (signatures factures), AES-256-GCM (QR), HMAC-SHA256 (Wallet) |

---

## Démarrer maintenant

Rendez-vous sur [Démarrage rapide](getting-started/quick-start.md) pour créer votre premier programme et générer vos premières cartes en moins de 10 minutes.

Consultez les [Plans & Fonctionnalités](getting-started/plans-and-pricing.md) pour choisir le plan adapté à votre programme.

---

> **Oomus CampaignID** — *Une identité. Un citoyen. Un système de santé souverain.*
