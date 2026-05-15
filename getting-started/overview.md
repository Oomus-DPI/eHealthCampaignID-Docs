# Vue d'ensemble

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

### Distribution multicanal
- **WhatsApp** : Meta Graph API v25.0 — image de carte + message personnalisé
- **SMS** : Orange SMS API OAuth2 — couverture Afrique de l'Ouest
- **Google Wallet** : pass générique avec QR, nom, programme, date — émission individuelle ou bulk (100 cartes)

### Moteur de simulation et gouvernance
- Estimation proforma complète avant engagement
- Workflow d'approbation admin multi-niveaux
- Génération de contrats (PDF, Excel, JSON/YAML)

### Sécurité et conformité
- Authentification JWT + bcrypt, RBAC institutionnel
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
| Identité nationale santé | Carte d'identité sanitaire | Google Wallet |

---

## Architecture de haut niveau

```
┌─────────────────────────────────────────────────────┐
│                   Oomus CampaignID                  │
├──────────────┬──────────────────┬───────────────────┤
│  Card Studio │  Campaign Engine │  MPI Sovereign    │
│  (11 modèles)│  (Celery async)  │  Identity         │
├──────────────┴──────────────────┴───────────────────┤
│              DHIS2 Tracker Integration               │
├──────────────┬──────────────────┬───────────────────┤
│  WhatsApp    │  SMS (Orange)    │  Google Wallet    │
│  (Meta v25)  │  (OAuth2)        │  (Generic Pass)   │
├──────────────┴──────────────────┴───────────────────┤
│  Verification Portal (Offline) │  HL7 FHIR R4       │
└─────────────────────────────────────────────────────┘
```

---

## Prochaines étapes

- [Démarrage rapide](quick-start.md) — Créez votre première campagne en 5 étapes
- [Plans & Tarification](plans-and-pricing.md) — Choisissez le plan adapté à votre programme
- [Identité MPI souveraine](../features/mpi-sovereign-identity.md) — Comprendre le système d'identité numérique
