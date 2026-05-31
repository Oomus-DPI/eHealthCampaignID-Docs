# Oomus CampaignID

**Infrastructure souveraine de santé publique numérique pour les programmes nationaux en Afrique et dans les pays à ressources limitées.**

> **Version** : 5.16 · **Date** : 2026-05-31 · **Statut** : Production

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

---

## Pour qui

| Public cible | Cas d'usage typique |
|---|---|
| Programmes nationaux de santé | Vaccination, paludisme, nutrition, HIV/PTME |
| Ministères de la Santé | Carte d'assurance maladie universelle, identité sanitaire nationale |
| Agences humanitaires & ONG | Identification des réfugiés, distribution de moustiquaires |
| Agences gouvernementales | Identité nationale reliée aux services de santé |
| Programmes de santé agricole | Carte agriculteur / santé rurale |

---

## Plans disponibles

| Plan | Identités/mois | SMS/mois | WhatsApp/mois |
|---|---|---|---|
| **Essential** | 10 000 | 50 000 | 10 000 |
| **Regional Command** | 100 000 | 250 000 | 100 000 |
| **National Infrastructure** | 1 000 000 | 3 000 000 | 1 000 000 |
| **Sovereign Cloud** | Illimité | Illimités | Illimités |

---

## Stack technique

| Composant | Technologie |
|---|---|
| Frontend | Next.js + React + TypeScript |
| Backend | FastAPI + Python + SQLAlchemy |
| Base de données | PostgreSQL 16 |
| Files de tâches | Celery + Redis |
| Stockage fichiers | S3-compatible |
| Interopérabilité | HL7 FHIR R4 |

---

## Démarrer maintenant

Rendez-vous sur [Démarrage rapide](getting-started/quick-start.md) pour créer votre premier programme et générer vos premières cartes.

Consultez les [Plans & Fonctionnalités](getting-started/plans-and-pricing.md) pour choisir le plan adapté à votre programme.

---

> **Oomus CampaignID** — *Une identité. Un citoyen. Un système de santé souverain.*
