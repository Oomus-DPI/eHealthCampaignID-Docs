# Vue d'ensemble

> **Version** : 5.11 · **Date** : 2026-05-23

## Vision

Oomus CampaignID est né d'un constat simple : dans de nombreux pays d'Afrique subsaharienne et dans les pays à ressources limitées (PRFI), les systèmes de santé publique souffrent de trois lacunes majeures :

1. **Les cartes papier** — facilement perdues, falsifiées ou détériorées, sans possibilité de vérification croisée
2. **La fragmentation des systèmes** — chaque programme (vaccination, nutrition, paludisme, HIV) gère ses propres listes, ses propres cartes, créant des doublons massifs et une impossibilité de vision longitudinale du bénéficiaire
3. **L'absence de vérification hors ligne** — dans des zones à connectivité intermittente, les agents de santé ne peuvent pas vérifier l'authenticité d'une carte sans connexion Internet

Oomus CampaignID répond à ces trois défis par une infrastructure souveraine, interopérable et conçue pour fonctionner dans les conditions réelles du terrain.

---

## Le problème résolu

### Cartes papier → Cartes numériques sécurisées

Chaque carte générée par Oomus CampaignID est :
- Associée à un identifiant unique cryptographiquement sécurisé (CSPRNG Base36)
- Signée par un QR code à jeton opaque (non réversible, basé SHA-256)
- Vérifiable hors ligne via un portail statique téléchargeable
- Vérifiable en ligne via la route publique `/verify?p=…&s=…` — sans authentification
- Distribuable instantanément par WhatsApp, SMS ou Google Wallet

### Systèmes fragmentés → Identité MPI unifiée


Le Master Patient Index (MPI) souverain permet à chaque citoyen de disposer d'un identifiant de santé numérique unique, utilisable à travers tous les programmes de santé. Un même bénéficiaire peut recevoir une carte de vaccination, une carte nutrition et une carte d'assurance maladie — tous reliés à la même identité souveraine, avec déduplication automatique.

### Vérification impossible → Vérification hors ligne universelle

Le portail de vérification d'Oomus CampaignID est un artefact statique (HTML + JSON) qui fonctionne sans aucune connexion au serveur. L'agent de santé télécharge le portail une seule fois, puis peut vérifier des milliers de cartes en mode complètement hors ligne.

---

## Capacités clés

### Génération de cartes à grande échelle
- Génération asynchrone par lots (Celery + Redis)
- Jusqu'à 10 millions de bénéficiaires par programme
- Options DPI : 300 dpi (standard), 450 dpi (amélioré), 600 dpi (impression PVC)
- Export PDF (recto/verso) + archive ZIP + portail de vérification offline
- **11 templates visuels** dont le template **Sovereign** (boarding-pass 1011×375 px @ 300 DPI, configurable via `SovereignCardConfig`, icône empreinte digitale)

### Card Studio — Éditeur visuel de modèles
- 11 modèles prêts à l'emploi couvrant les principaux programmes de santé
- Personnalisation complète : logos, couleurs, champs dynamiques, QR code
- Aperçu PNG en temps réel
- Export des configurations en YAML/JSON

### Intégration DHIS2 Tracker
- Connexion directe à votre instance DHIS2 (v2.36+)
- Synchronisation automatique des enrollments (cron configurable)
- Mapping des attributs DHIS2 → champs de carte
- Résolution MPI automatique lors de la synchronisation
- Protection des données sensibles (garde IA — 7 catégories)

### Sovereign Wallet
- Passes digitaux signés HMAC-SHA256 — vérifiables hors ligne
- Bundle offline chiffré XOR-SHA256 pour synchronisation sur appareil
- Gestion multi-appareils avec historique de synchronisation
- Révocation auditée et horodatée
- Portail public de vérification de pass : `/verify?p=<payload>&s=<sig>`

### Configuration pays & devise

Oomus CampaignID inclut un référentiel de **40 pays africains** regroupés en 4 régions, chacun associé à sa devise ISO 4217 :

| Région | Pays couverts |
|---|---|
| **Afrique de l'Ouest** | Sénégal, Mali, Burkina Faso, Niger, Guinée, Côte d'Ivoire, Ghana, Togo, Bénin, Nigeria, et 5 autres |
| **Afrique Centrale** | Cameroun, Tchad, RCA, RDC, République du Congo, Gabon |
| **Afrique de l'Est** | Rwanda, Ouganda, Kenya, Tanzanie, Éthiopie, Somalie, et 3 autres |
| **Afrique Australe** | Afrique du Sud, Zambie, Zimbabwe, Botswana, Namibie, et 4 autres |

Sélectionner un pays dans les paramètres du programme remplit automatiquement la devise (XOF, XAF, NGN, KES, ZAR…). Les 21 devises uniques couvertes sont disponibles dans le menu Préférences.

### Distribution multicanal
- **WhatsApp** : Meta Graph API v25.0 — image de carte + message personnalisé
- **SMS** : Orange SMS API OAuth2 — couverture Afrique de l'Ouest
- **Google Wallet** : pass générique avec QR, nom, programme, date — émission individuelle ou bulk (100 cartes)

### Moteur de simulation et gouvernance
- Estimation proforma complète avant engagement
- Workflow d'approbation admin multi-niveaux
- Génération de contrats (PDF, Excel, JSON/YAML)

### Panneau Admin v5.6 — Gouvernance complète
Le panneau d'administration offre une gestion opérationnelle complète de la plateforme :

| Section admin | Capacités |
|---|---|
| **Intégration DHIS2** | Gestion configs, toggle actif/inactif, sync manuelle, consultation enrollments |
| **Portails Vérification** | Création, statut, suppression des portails de vérification dédiés |
| **Analytics Plateforme** | KPIs globaux, taux de succès, top programmes par volume de cartes |
| **Détection Fraude IA** | Alertes IsolationForest, filtres par risque, répartition par type, dismiss |
| **Toutes les Campagnes** | Vue globale multi-programmes, filtres status/type/recherche, suppression |
| **Jobs de Génération** | Stats workers Celery, filtre statut, annulation de jobs actifs |
| **Commandes PVC** | Suivi fabrication et livraison, transitions de statut, saisie N° de suivi |
| **Registre MPI Souverain** | Stats globales, liste paginée, détail identité, vérification manuelle |

### Sécurité et conformité
- Authentification JWT HS256 (algorithme fixé, token versioning) + bcrypt, RBAC institutionnel
- Access token 8h, politique de mots de passe renforcée (10 chars + complexité)
- Isolation réseau Docker : PostgreSQL/Redis sans port externe, MinIO/Flower sur 127.0.0.1
- CORS strict, Content-Security-Policy, HSTS en production
- Chaîne d'audit immuable (SHA-256)
- Détection d'anomalies par IA (IsolationForest)
- Garde automatique des données sensibles (HIV, TB, biométrie, etc.)
- Option hébergement souverain

---

## Utilisateurs cibles

| Type d'organisation | Exemples |
|---|---|
| **Programmes nationaux de santé** | Direction nationale de l'immunisation, PNLP (paludisme), PNLS (HIV) |
| **Ministères de la Santé** | DSI santé, direction de l'assurance maladie universelle |
| **Agences humanitaires & ONG** | HCR, UNICEF, MSF, World Vision, Plan International |
| **Agences gouvernementales** | Agences nationales d'identification |
| **Collectivités locales** | Régions, districts sanitaires |

---

## Cas d'usage supportés

| Programme | Type de carte | Distribution |
|---|---|---|
| Vaccination (EPI) | Carte de vaccination numérique | WhatsApp + SMS |
| Paludisme / MILD | Carte de distribution moustiquaires | Google Wallet + SMS |
| Nutrition | Carte de suivi nutritionnel | WhatsApp |
| Santé maternelle (CPN/PTME) | Carte de suivi anténatal | WhatsApp + SMS |
| HIV/PTME | Carte de programme (données sensibles protégées) | SMS uniquement |
| Assurance maladie | Carte d'affiliation | Google Wallet + WhatsApp |
| Identification réfugiés | Carte d'identité humanitaire | SMS + hors ligne |
| Santé agricole | Carte agriculteur / santé rurale | SMS |
| Laboratoire | Carte de résultats (lab) | WhatsApp |
| Identité nationale santé | Carte d'identité sanitaire souveraine | Google Wallet + Sovereign Wallet |

---

## Architecture de haut niveau

```
┌──────────────────────────────────────────────────────────────────┐
│                       Oomus CampaignID v5.11                     │
├──────────────┬──────────────────┬───────────────────┬────────────┤
│  Card Studio │  Campaign Engine │  MPI Sovereign    │  Admin v5.4│
│  (11 modèles)│  (Celery async)  │  Identity         │  (8 pages) │
├──────────────┴──────────────────┴───────────────────┴────────────┤
│              DHIS2 Tracker Integration (safe read-only guard)     │
├──────────────┬──────────────────┬───────────────────┬────────────┤
│  WhatsApp    │  SMS (Orange)    │  Google Wallet    │  Sovereign │
│  (Meta v25)  │  (OAuth2)        │  (Generic Pass)   │  Wallet    │
├──────────────┴──────────────────┴───────────────────┴────────────┤
│  Verify Portal (Offline + /verify public) │  HL7 FHIR R4         │
└──────────────────────────────────────────────────────────────────┘
```

---

## Prochaines étapes

- [Démarrage rapide](quick-start.md) — Créez votre première campagne en 5 étapes
- [Plans & Tarification](plans-and-pricing.md) — Choisissez le plan adapté à votre programme
- [Identité MPI souveraine](../features/mpi-sovereign-identity.md) — Comprendre le système d'identité numérique
- [Modules Enterprise v5.2+](../features/enterprise-modules.md) — AI, Geo, Trust Score, Sovereign Wallet
