# Oomus CampaignID

**Infrastructure souveraine de santé publique numérique pour les programmes nationaux en Afrique et dans les pays à ressources limitées.**

> **Version** : 5.6 · **Date** : 2026-05-19 · **Statut** : Production

---

## Bienvenue sur la documentation officielle

Oomus CampaignID est une plateforme GovTech Enterprise SaaS conçue pour la génération, la distribution, la vérification et la gouvernance de cartes d'identité sanitaire numériques sécurisées. Elle repose sur une identité numérique MPI (Master Patient Index) souveraine, interopérable HL7 FHIR R4, et pensée pour fonctionner dans des environnements à faible connectivité.

---

## Ce que fait Oomus CampaignID

Oomus CampaignID permet aux programmes de santé nationaux et aux organisations humanitaires de :

- **Générer** des cartes de santé numériques sécurisées à grande échelle (jusqu'à 10 millions de bénéficiaires)
- **Distribuer** les cartes par WhatsApp, SMS et Google Wallet depuis une interface unifiée
- **Vérifier** l'authenticité des cartes hors ligne — sans connexion Internet requise
- **Déduplicater** automatiquement les identités grâce au moteur MPI probabiliste
- **Synchroniser** les données depuis DHIS2 Tracker en temps réel ou selon un calendrier configurable
- **Gouverner** les accès, les quotas et les approbations grâce à un système RBAC institutionnel complet
- **Émettre** des Sovereign Wallet Passes — passes digitaux signés HMAC-SHA256, synchronisables hors ligne
- **Administrer** la plateforme entière depuis un panneau admin unifié (8 sections de gouvernance)

---

## Capacités clés

- **Card Studio** — Éditeur visuel avec 11 modèles de cartes dont le nouveau template **Sovereign** (navy/or premium), aperçu PNG, export YAML/JSON, options DPI 300/450/600
- **Identité MPI souveraine** — 1 citoyen = 1 identifiant de santé numérique, inter-programmes, à vie
- **Intégration DHIS2** — Synchronisation automatique, mapping d'attributs, génération de cartes depuis les enrollments, guard IA données sensibles
- **Distribution multicanal** — WhatsApp (Meta Graph API v25.0), SMS (Orange API), Google Wallet
- **Vérification hors ligne** — Portail statique, registre SHA-256, WebCrypto, multilingue (FR/EN/WO)
- **Portail de vérification public** — Route standalone `/verify?p=…&s=…` — vérification QR depuis n'importe quel appareil sans authentification
- **Sovereign Wallet** — Passes digitaux signé HMAC-SHA256, bundle offline XOR-SHA256, sync appareils, révocation auditée
- **Moteur de simulation** — Estimation proforma, workflow d'approbation admin, génération de contrats
- **Dashboard opérationnel** — KPIs temps réel, analytics avancées, santé de l'infrastructure, alertes
- **Panneau admin v5.4** — 8 pages de gouvernance complètes : DHIS2, Portails Vérif., Analytics, Fraude IA, Campagnes, Jobs, PVC, MPI
- **UI mature v5.6** — Page Campagnes (table épurée, pas d'affichage des coûts sauf dépassement quota) et page Connexion redesignées avec une esthétique professionnelle sobre
- **Logo programme persistant** — Upload `POST /auth/logo` sauvegarde le logo en base (data URI), profil latéral et sidebar affichent le logo réel du programme
- **Authentification 2FA** — Endpoints TOTP alignés : `two_factor_enabled` / `totp_secret` correctement exposés dans `ProgrammeOut`
- **Sécurité enterprise** — AES-256-GCM, chaîne d'audit immuable, IsolationForest, garde données sensibles

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

## Stack technique

| Composant | Technologie |
|---|---|
| Frontend | Next.js 16.2.4 + React 19 + TypeScript 5 |
| Backend | FastAPI 0.115 + Python 3.12 |
| Base de données | PostgreSQL 16 + Alembic (migrations 0001–0005) |
| Files de tâches | Celery 5 + Redis 7 |
| Stockage fichiers | MinIO / S3-compatible |
| Interopérabilité | HL7 FHIR R4 |

---

## Démarrer maintenant

Rendez-vous sur [Démarrage rapide](getting-started/quick-start.md) pour créer votre premier programme et générer vos premières cartes en moins de 10 minutes.

Consultez les [Plans & Tarification](getting-started/plans-and-pricing.md) pour choisir le plan adapté à votre programme.

---

> **Oomus CampaignID** — *Une identité. Un citoyen. Un système de santé souverain.*
