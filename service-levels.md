# Accord de niveau de service (SLA)

Ce document décrit les niveaux de service cibles d'Oomus CampaignID, les objectifs de temps de réponse et les priorités de support.

---

## Disponibilité de la plateforme

| Niveau | Disponibilité cible | Applicable à |
|---|---|---|
| **Standard** | 99,0% (~ 7,3h d'interruption/mois) | Plans Starter, Regional Ops |
| **Renforcé** | 99,5% (~ 3,6h d'interruption/mois) | Plan National Campaign |
| **Premium** | 99,9% (~ 44 min d'interruption/mois) | Plan Sovereign Enterprise (contractuel) |
| **Critique** | 99,99% (~ 4,4 min d'interruption/mois) | Sovereign Enterprise + module Advanced SLA |

> Les objectifs de disponibilité sont calculés hors maintenance planifiée (annoncée avec préavis de 72h minimum) et hors cas de force majeure.

---

## Objectifs de temps de réponse API

| Type de requête | Objectif (P95) | Description |
|---|---|---|
| **Lecture API** | < 200 ms | GET sur campagnes, bénéficiaires, statuts |
| **Écriture API** | < 500 ms | POST/PATCH sur campagnes, configuration |
| **WebSocket** | < 1 seconde | Connexion et premier événement de progression |
| **Génération de cartes** | < 60 secondes pour 100 cartes | Job Celery asynchrone |
| **Synchronisation DHIS2** | < 30 secondes pour 1 000 enrollments | Tâche de synchronisation |
| **Distribution WhatsApp/SMS** | < 5 secondes par message | Envoi via API externe |
| **Vérification hors ligne** | < 50 ms | Lookup local WebCrypto (aucune requête réseau) |

> Les temps de réponse pour la génération de cartes et la distribution dépendent partiellement des APIs tierces (Meta WhatsApp, Orange SMS, Google Wallet). Ces composants externes ont leurs propres SLA.

---

## Priorités de support

Les incidents sont classifiés en 4 niveaux de priorité :

### P1 — Critique (Service complètement indisponible)

| Critère | Description |
|---|---|
| **Définition** | La plateforme est totalement inaccessible pour tous les utilisateurs |
| **Exemples** | API inaccessible, base de données hors ligne, infrastructure effondrée |
| **Temps de réponse initial** | 1 heure (Premium/Critical SLA) · 4 heures (Enhanced) · 8 heures (Standard) |
| **Canal de contact** | E-mail urgent + téléphone (Enterprise) |

### P2 — Haute (Fonctionnalité majeure dégradée)

| Critère | Description |
|---|---|
| **Définition** | Une fonctionnalité essentielle est indisponible ou très dégradée |
| **Exemples** | Génération de cartes bloquée, distribution WhatsApp en échec, DHIS2 sync arrêtée |
| **Temps de réponse initial** | 4 heures (Premium) · 8 heures (Enhanced) · Jour ouvrable suivant (Standard) |
| **Canal de contact** | E-mail support prioritaire |

### P3 — Moyenne (Fonctionnalité mineure dégradée)

| Critère | Description |
|---|---|
| **Définition** | Une fonctionnalité secondaire est dégradée mais un contournement existe |
| **Exemples** | Analytics lentes, aperçu PNG défaillant, UI bug non bloquant |
| **Temps de réponse initial** | Jour ouvrable suivant (tous plans) |
| **Canal de contact** | E-mail support |

### P4 — Basse (Demande d'amélioration / question)

| Critère | Description |
|---|---|
| **Définition** | Question fonctionnelle, suggestion d'amélioration, documentation manquante |
| **Exemples** | "Comment configurer le mapping DHIS2 ?" / "Puis-je avoir ce champ sur la carte ?" |
| **Temps de réponse initial** | 3 jours ouvrables |
| **Canal de contact** | E-mail ou portail documentation |

---

## Niveaux de support par plan

| Plan | Niveau de support | Canal | Heures |
|---|---|---|---|
| **Starter** | Communautaire | Documentation + forum | 24/7 (auto-service) |
| **Regional Ops** | Standard | E-mail | Jours ouvrables (8h–18h UTC) |
| **National Campaign** | Prioritaire | E-mail + ticketing | Jours ouvrables (7h–20h UTC) |
| **Sovereign Enterprise** | Dédié | E-mail + téléphone + canal dédié | 24/7 |

**Contact support** : support@oomus.health

---

## SLA des dépendances tierces

Oomus CampaignID s'appuie sur des services tiers dont les niveaux de service sont indépendants et hors de notre contrôle :

| Service tiers | Usage dans Oomus | SLA tiers de référence |
|---|---|---|
| **DHIS2** | Synchronisation des enrollments | Dépend de votre instance DHIS2 |
| **WhatsApp / Meta Graph API** | Distribution des cartes | SLA Meta Enterprise |
| **Orange SMS API** | Distribution SMS | SLA Orange Developer |
| **Google Wallet API** | Émission de passes | SLA Google Cloud |
| **Stockage S3/MinIO** | Stockage des artefacts | SLA du fournisseur configuré |

En cas de dégradation d'un service tiers, Oomus CampaignID notifiera les programmes affectés et proposera des alternatives lorsqu'elles existent.

---

## Maintenance planifiée

Les opérations de maintenance planifiée sont communiquées avec un préavis minimum de :

| Type de maintenance | Préavis |
|---|---|
| Maintenance mineure (< 5 min d'impact) | 24 heures |
| Maintenance standard (5–30 min d'impact) | 72 heures |
| Maintenance majeure (> 30 min d'impact) | 7 jours |

Les maintenances sont planifiées en dehors des heures de pointe d'utilisation (généralement entre 22h et 4h UTC le dimanche).

**Page de statut** : status.oomus.health (en cours de déploiement)

---

## Exclusions du SLA

Les objectifs de disponibilité et de temps de réponse ne s'appliquent pas dans les cas suivants :

- Force majeure (catastrophes naturelles, coupures Internet nationales, conflits)
- Maintenance planifiée (annoncée selon les préavis ci-dessus)
- Attaques DDoS actives et non encore mitigées
- Indisponibilité causée par les actions du client (configuration incorrecte, abus d'API)
- Indisponibilité des services tiers (DHIS2, Meta, Orange, Google)

---

## Prochaines étapes

- [Plans & Tarification](getting-started/plans-and-pricing.md) — Choisir le niveau SLA adapté
- [Vue d'ensemble sécurité](security/overview.md)
- [Contact support](mailto:support@oomus.health)
