# Product Requirements — OOMUS eHealth CampaignID v4.2

## Objectif produit

Permettre à un ministère de la santé, une ONG, un programme national ou un partenaire technique de **simuler, créer, générer, distribuer, vérifier et superviser** des cartes d'identité sanitaires sécurisées pour des campagnes de santé publique à grande échelle.

La plateforme est positionnée comme une **infrastructure opérationnelle souveraine** — pas un outil de formulaires, mais un système de commandement de campagne avec facturation quota, simulation avant déploiement, gouvernance RBAC institutionnelle et audit immuable.

---

## Fonctionnalités principales

### Simulation & planification

- Moteur de simulation deux modes avant tout déploiement réel.
- Wizard 6 étapes : moteur, volumes, infrastructure, gouvernance RBAC, modules premium, résultats.
- Proforma PDF (ReportLab) et Excel (openpyxl 4 feuilles) générés automatiquement.
- Export config JSON/YAML pour provisioning.
- Workflow de validation admin : submitted → approved / rejected / modification_requested → provisioned.

### Dashboard Operations Center (temps réel)

- Polling automatique toutes les 30 secondes.
- 5 KPI temps réel : solde, cartes générées, jobs actifs, taux de succès, campagnes.
- ActivityChart annuel : jobs, cartes générées, syncs DHIS2 par mois (Recharts).
- Quotas de consommation en temps réel : bénéficiaires, SMS, WhatsApp, vérifications.
- Flux d'événements live dérivé des jobs réels.
- Alertes actives : jobs échoués, quotas > 80 %, running jobs.

### Gestion des campagnes

- Création de campagne avec wizard 5 étapes.
- Upload de template `.yml` personnalisé.
- Statuts : `draft → preview_ready → generating → completed / failed`.
- Génération asynchrone Celery avec progression WebSocket temps réel.
- DPI 300 / 450 / 600 — multiplicateurs tarifaires × 1.0 / × 1.4 / × 2.0.
- Remboursement automatique si le job échoue.

### Card Studio

Éditeur no-code de cartes digitales — 11 templates disponibles :

| Template | Programme |
| -------- | --------- |
| `cps` | Chimio-Prévention Paludisme (recto + verso) |
| `mild` | Distribution moustiquaires MILD |
| `vaccination` | Carnet de vaccination |
| `antenatal` | Suivi prénatal CPN |
| `nutrition` | Programme nutrition ANJE |
| `hiv` | Suivi PTME / VIH |
| `lab` | Carte d'examen biologique |
| `assurance` | Assurance maladie |
| `refugee` | Carte réfugié |
| `identity` | Carte d'identité sanitaire |
| `farmercard` | Carte agriculteur |

Fonctionnalités : logos ×3, couleurs header, QR personnalisable, champs dynamiques recto/verso, aperçu PNG réel.

### Facturation quota (v4.2)

- 4 plans d'abonnement mensuel (voir [Billing & Quota Plans](../reference/billing-quota-plans.md)).
- SMS et WhatsApp inclus dans le quota — seuls les dépassements sont facturés.
- Coûts infrastructure auto-calculés depuis les usages réels.
- Aucun prix à l'unité exposé côté client.

### Intégration DHIS2 Tracker

- Configuration, test de connexion, liste des programmes DHIS2.
- Génération de cartes PNG depuis les données DHIS2 (Pillow natif, ≤ 6 attributs).
- Synchronisation automatique planifiable (10 / 15 / 30 / 60 / 180 min).
- Aperçu PNG grille 4 cartes avec données réelles + watermark.

### Distribution

- **WhatsApp** : Meta Graph API v25.0, Bearer token chiffré Fernet AES-256.
- **SMS** : Orange SMS API OAuth2 West Africa, bulk batch 20.
- **Google Wallet** : JWT RS256 individuel + bulk ≤ 100 cartes.
- Logs complets : canal, événement, latence ms, succès.

### Vérification publique

- Portail statique offline généré automatiquement à chaque lot.
- Vérification 100 % client-side — aucune donnée transmise.
- SHA-256 lookup dans `registry.json`.
- Trilingue : FR / EN / WO (Wolof).
- Modes : code unique · scanner QR · masse CSV · statistiques.

### Sécurité & gouvernance

- RBAC institutionnel : `super_admin`, `programme_admin`, `programme_user`, rôles personnalisés.
- Permissions granulaires assignables par programme.
- Workflows d'approbation multi-niveaux (jusqu'à 10 niveaux configurables).
- Journal d'audit immuable : acteur, action, ressource, IP, statut.

### Modules Premium (activation à la demande)

| Module | Clé |
| ------ | --- |
| SMS Gateway | `sms_gateway` |
| WhatsApp BSP | `whatsapp_bsp` |
| Analytics Avancés | `advanced_analytics` |
| IA Détection Fraude | `ai_fraud_detection` |
| Sync Offline | `offline_sync` |
| Cartes PVC Physiques | `physical_pvc_card` |
| Portail Vérification Dédié | `dedicated_verify_platform` |
| SLA Avancé | `advanced_sla` |
| Hébergement Souverain | `sovereign_hosting` |
| Provisioning Dédié | `dedicated_provisioning` |

---

## Utilisateurs cibles

| Rôle | Responsabilités |
| ---- | --------------- |
| Administrateur plateforme | Validation simulations, gestion programmes, tarification, rechargements |
| Gestionnaire de programme | Création campagnes, supervision jobs, facturation |
| Opérateur de campagne | Lancement générations, distribution, monitoring |
| Partenaire technique | Intégration API, configuration DHIS2, déploiement |
| Vérificateur public | Consultation portail public (sans authentification) |
| Équipe terrain | Scanner QR, vérification code unique |

---

## Critères d'acceptation

- Un utilisateur authentifié peut créer et simuler une campagne avant tout déploiement réel.
- Une simulation soumise passe par un workflow admin (approved / rejected / modification_requested).
- Une campagne sans template ne peut pas être générée.
- Un job expose son statut, sa progression et son coût en temps réel via WebSocket.
- Un job terminé expose des URLs de téléchargement signées (PDF, ZIP, registre, rapport).
- Un code voucher généré peut être vérifié sans authentification, offline, depuis le portail statique.
- Une campagne peut être alimentée depuis DHIS2 Tracker si l'intégration est configurée.
- Une carte peut être distribuée par WhatsApp, SMS ou Google Wallet.
- Le quota de consommation (SMS, bénéficiaires, vérifications) est visible en temps réel sur le dashboard.
- Les QR codes ne transmettent aucune donnée médicale sensible en clair.
- Tout accès admin est enregistré dans l'audit log immuable.
