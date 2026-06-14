# Oomus CampaignID

**Infrastructure souveraine de santé publique numérique pour les programmes nationaux en Afrique et dans les pays à ressources limitées.**

> **Version** : 8.0 · **Date** : 2026-06-14 · **Statut** : Production

---

## Bienvenue sur la documentation officielle

Oomus CampaignID est une plateforme GovTech Enterprise SaaS conçue pour la génération, la distribution, la vérification et la gouvernance de cartes d'identité sanitaire numériques sécurisées. Elle repose sur une identité numérique MPI (Master Patient Index) souveraine, interopérable HL7 FHIR R4, et pensée pour fonctionner dans des environnements à faible connectivité.

---

## Ce que fait Oomus CampaignID

Oomus CampaignID permet aux programmes de santé nationaux et aux organisations humanitaires de :

- **Générer** des cartes de santé numériques sécurisées à grande échelle
- **Distribuer** les cartes par WhatsApp, SMS et Google Wallet depuis une interface unifiée
- **Vérifier** l'authenticité des cartes hors ligne — sans connexion Internet requise
- **Dédupliquer** automatiquement les identités grâce au moteur MPI probabiliste
- **Synchroniser** les données depuis DHIS2 Tracker en temps réel
- **Gouverner** les accès, les quotas et les approbations via un système RBAC institutionnel
- **Émettre** des Sovereign Wallet Passes — passes digitaux signés, synchronisables hors ligne
- **Produire** des cartes PVC physiques en deux qualités d'impression
- **Facturer** avec une traçabilité complète — chaque débit génère automatiquement une facture formelle signée
- **Accéder** à l'identité souveraine depuis l'application mobile **OOMUS Wallet** (iOS + Android) — bilingue FR/EN
- **Vérifier** son identité avec le wizard KYC 5 étapes (CNI + selfie biométrique) — score de confiance MPI mis à jour en temps réel
- **Consulter** le Registre d'Identités Souveraines — niveaux IAL 0–3, fonctionnalités débloquées, intégrité cryptographique

---

## Capacités clés

- **Card Studio** — Éditeur visuel avec 11 modèles de cartes, aperçu temps réel, export, options DPI 300/450/600
- **Identités numériques souveraines** — 1 citoyen = 1 identifiant de santé numérique MPI, inter-programmes, à vie
- **Intégration DHIS2** — Synchronisation automatique, mapping d'attributs, génération depuis les enrollments
- **Distribution multicanal** — WhatsApp (Meta Graph API), SMS (Orange API), Google Wallet
- **Vérification hors ligne** — Portail statique, cryptographie côté navigateur, multilingue (FR/EN/WO)
- **Sovereign Wallet** — Passes digitaux signés, bundle offline, sync appareils, révocation auditée
- **Simulation financière** — Estimation proforma, workflow d'approbation admin, génération PDF/Excel
- **Cartes PVC physiques** — Standard et Offset Industriel, timeline de suivi
- **Billing Infrastructure** — Registre comptable centralisé avec factures signées et audit immuable
- **Dashboard opérationnel** — KPIs temps réel, analytics avancées, alertes
- **Sécurité Enterprise** — 2FA TOTP, brute-force protection, JTI token revocation, RBAC institutionnel
- **Application mobile bilingue** — OOMUS Wallet React Native / Expo SDK 54 — FR/EN, biométrie, offline-first
- **KYC Wizard 5 étapes** — vérification d'identité CNI + selfie, score de confiance MPI propagé en temps réel via WebSocket
- **Portefeuille Citoyen temps réel** — activité live, `ACTION_LABELS` humains, filtres, toasts animés
- **Registre d'Identités Souveraines** — niveaux IAL 0–3, empreinte cryptographique, signalement d'erreur

---

## Pour qui

| Public cible | Cas d'usage typique |
| --- | --- |
| Programmes nationaux de santé | Vaccination, paludisme, nutrition, HIV/PTME |
| Ministères de la Santé | Carte d'assurance maladie universelle, identité sanitaire nationale |
| Agences humanitaires & ONG | Identification des réfugiés, distribution de moustiquaires |
| Agences gouvernementales | Identité nationale reliée aux services de santé |
| Programmes de santé agricole | Carte agriculteur / santé rurale |

---

## Plans disponibles

| Plan | Identités/mois | SMS/mois | WhatsApp/mois |
| --- | --- | --- | --- |
| **Essential** | 10 000 | 50 000 | 10 000 |
| **Regional Command** | 100 000 | 250 000 | 100 000 |
| **National Infrastructure** | 1 000 000 | 3 000 000 | 1 000 000 |
| **Sovereign Cloud** | Illimité | Illimités | Illimités |

---

## Stack technique

| Composant | Technologie |
| --- | --- |
| Frontend | Next.js + React + TypeScript |
| Backend | FastAPI + Python + SQLAlchemy |
| Base de données | PostgreSQL 16 |
| Files de tâches | Celery + Redis |
| Stockage fichiers | S3-compatible |
| Interopérabilité | HL7 FHIR R4 |
| **Application mobile** | React Native 0.81.5 · Expo SDK 54 · TypeScript · i18next (FR/EN) |
| **Temps réel mobile** | WebSocket + Redis pubsub · SyncContext · `publish_wallet_event()` |
| **KYC Trust Bridge** | `kyc_trust_bridge.py` · `MpiTrustScore` · `MpiVerificationEvent` |

---

## Démarrer maintenant

Rendez-vous sur [Démarrage rapide](getting-started/quick-start.md) pour créer votre premier programme et générer vos premières cartes.

Consultez les [Plans & Fonctionnalités](getting-started/plans-and-pricing.md) pour choisir le plan adapté à votre programme.

---

---

## Application mobile — OOMUS Wallet

L'application mobile **OOMUS Wallet** (React Native / Expo SDK 54) permet à chaque citoyen d'accéder à son identité souveraine MPI, ses passes de santé et de contrôler le partage de ses données depuis son téléphone.

**Fonctionnalités clés :**

- Connexion OTP SMS → PIN 6 chiffres → biométrie (Face ID / Fingerprint)
- Carte d'identité souveraine `SovereignCard` avec QR HMAC-SHA256
- **Wizard KYC 5 étapes** — CNI + selfie biométrique, score MPI mis à jour en temps réel après finalisation
- **Portefeuille Citoyen** — activité live WebSocket, `ACTION_LABELS` humains, filtres Tous/Portefeuille/Santé/Identité
- **Registre d'Identités Souveraines** — niveaux IAL 0–3, empreinte cryptographique, signalement d'erreur, services débloqués
- Gestion des passes de santé (vaccination, assurance, ordonnance…) avec overlay `Niveau IAL{n} requis`
- Consentements de partage avec GrantModal 2 étapes
- Journal d'activité complet — labels humains, fallback `'Événement système'`
- Module Assurance maladie — couvertures, vérification d'actes, historique
- **Bilingue FR/EN** — sélecteur de langue dans Paramètres, persistance AsyncStorage
- Mode hors ligne (SyncContext) — passes et identité en cache, sync auto au retour réseau

**Démarrage :**

```bash
cd mobile
npm install
npx expo start          # QR code Expo Go
npx expo run:ios        # Simulateur iOS
npx expo run:android    # Émulateur Android
```

**Variables d'environnement :**

```bash
EXPO_PUBLIC_API_URL=https://api.oomus.org   # URL du backend
```

---

> **Oomus CampaignID** — *Une identité. Un citoyen. Un système de santé souverain.*
