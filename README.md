# Oomus CampaignID

**Infrastructure souveraine de santé publique numérique pour les programmes nationaux en Afrique et dans les pays à ressources limitées.**

---

## Bienvenue sur la documentation officielle

Oomus CampaignID est une plateforme GovTech Enterprise SaaS (v5.1) conçue pour la génération, la distribution, la vérification et la gouvernance de cartes d'identité sanitaire numériques sécurisées. Elle repose sur une identité numérique MPI (Master Patient Index) souveraine, interopérable HL7 FHIR R4, et pensée pour fonctionner dans des environnements à faible connectivité.

---

## Ce que fait Oomus CampaignID

Oomus CampaignID permet aux programmes de santé nationaux et aux organisations humanitaires de :

- **Générer** des cartes de santé numériques sécurisées à grande échelle (jusqu'à 10 millions de bénéficiaires)
- **Distribuer** les cartes par WhatsApp, SMS et Google Wallet depuis une interface unifiée
- **Vérifier** l'authenticité des cartes hors ligne — sans connexion Internet requise
- **Déduplicater** automatiquement les identités grâce au moteur MPI probabiliste
- **Synchroniser** les données depuis DHIS2 Tracker en temps réel ou selon un calendrier configurable
- **Gouverner** les accès, les quotas et les approbations grâce à un système RBAC institutionnel complet

---

## Capacités clés

- **Card Studio** — Éditeur visuel avec 11 modèles de cartes, aperçu PNG, export YAML/JSON, options DPI 300/450/600
- **Identité MPI souveraine** — 1 citoyen = 1 identifiant de santé numérique, inter-programmes, à vie
- **Intégration DHIS2** — Synchronisation automatique, 7 templates de cartes (vital/emerald/pulse/mothercare/shield/nomad/aero), génération depuis les enrollments
- **Distribution multicanal** — WhatsApp (Meta Graph API v25.0), SMS (Orange API), Google Wallet
- **Vérification hors ligne** — Portail statique, registre SHA-256, WebCrypto, multilingue (FR/EN/WO)
- **Moteur de simulation** — Estimation proforma, workflow d'approbation admin, génération de contrats
- **Dashboard opérationnel** — KPIs temps réel, analytics IA (prédictions 30j, anomalies, score santé), alertes
- **Administration globale** — 6 pages admin cross-programmes (analytics, fraude, campagnes, jobs, PVC, MPI) avec charts interactifs
- **Sécurité enterprise** — AES-256-GCM, chaîne d'audit immuable, 2FA TOTP, détection d'anomalies IA, garde données sensibles
- **Branding programme** — Logo personnalisé par programme, couleur primaire, nom affiché — persistés en base et affichés dans toute l'interface
- **Interface adaptative** — Responsive mobile (< 900 px : sidebar tiroir, header hamburger), mode sombre natif (Paramètres → Préférences)

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
| Frontend | Next.js 15 — responsive, dark mode, ThemeContext |
| Backend | FastAPI 0.115 + SQLAlchemy 2 (async) |
| Base de données | PostgreSQL 15+ (asyncpg) |
| Files de tâches | Celery 5 + Redis 7 |
| Stockage fichiers | MinIO / S3-compatible (logos, PDFs, portails) |
| Interopérabilité | HL7 FHIR R4 |
| Authentification | JWT HS256 + 2FA TOTP |

---

## Démarrer maintenant

Rendez-vous sur [Démarrage rapide](getting-started/quick-start.md) pour créer votre premier programme et générer vos premières cartes en moins de 10 minutes.

Consultez les [Plans & Tarification](getting-started/plans-and-pricing.md) pour choisir le plan adapté à votre programme.

---

> **Oomus CampaignID** — *Une identité. Un citoyen. Un système de santé souverain.*
