# Gestion des campagnes

Les campagnes sont l'unité centrale d'Oomus CampaignID. Chaque campagne regroupe un ensemble de cartes générées pour un programme, une période et une cible de bénéficiaires donnés.

---

## Création d'une campagne — Assistant wizard

La création d'une campagne se fait via un assistant en 5 étapes :

### Étape 1 — Informations générales

| Champ | Description | Exemple |
|---|---|---|
| **Nom** | Nom descriptif de la campagne | `Vaccination EPI - Dakar 2026` |
| **Type** | Type de programme de santé | `vaccination` |
| **Préfixe** | Code alphanumérique unique (3–8 car., maj.) | `DKR-VAC` |
| **Description** | Description optionnelle | `Campagne pilote district Dakar` |

### Étape 2 — Langue

Sélectionnez la langue principale de la campagne :
- **Français** (`fr`)
- **English** (`en`)
- **Wolof** (`wo`)

La langue affecte les libellés des champs sur les cartes, les messages de distribution (SMS/WhatsApp) et le portail de vérification généré.

### Étape 3 — Sélection du modèle

Choisissez parmi les 11 modèles disponibles dans le Card Studio. Le modèle peut être personnalisé après la création de la campagne.

### Étape 4 — Configuration DPI

Sélectionnez la résolution cible selon votre usage final (affichage numérique, impression laser, PVC). Les options disponibles dépendent de votre plan.

### Étape 5 — Révision et validation

Récapitulatif complet avant création. Vérifiez les paramètres — le préfixe et le type ne peuvent plus être modifiés après création.

---

## Types de campagne

| Type | Description |
|---|---|
| `vaccination` | Programme de vaccination (EPI, routine, campagne de masse) |
| `mild` | Distribution de moustiquaires imprégnées longue durée |
| `nutrition` | Suivi nutritionnel (enfants, femmes enceintes/allaitantes) |
| `antenatal` | Soins prénataux et consultations maternelles |
| `hiv` | Programme HIV/PTME (données sensibles — accès restreint) |
| `assurance` | Assurance maladie universelle |
| `refugee` | Identification humanitaire des réfugiés |
| `identity` | Identité sanitaire nationale |
| `lab` | Examens biologiques et suivi de laboratoire |
| `cps` | Protection sociale communautaire |
| `farmercard` | Santé agricole et rurale |

---

## Génération de cartes

### Génération asynchrone

La génération de cartes est un processus **asynchrone** géré par un moteur de files de tâches (Celery + Redis). Cela signifie que votre requête est acceptée immédiatement et traitée en arrière-plan, sans bloquer votre interface.

**Processus de génération :**

```
Requête API → Validation quota → Création job → File Celery
→ Worker génère les cartes (PNG, PDF, QR)
→ Assemblage ZIP + portail vérification
→ Mise à jour statut → Notification WebSocket
```

### Suivi de progression en temps réel — WebSocket

Connectez-vous au WebSocket de progression pour recevoir les mises à jour en direct :

```
wss://api.oomus.health/ws/jobs/{job_id}?token={access_token}
```

Les événements WebSocket envoyent :

```json
{
  "event": "progress",
  "job_id": "job_01HXYZ789GHI",
  "status": "generating",
  "progress": 45,
  "cards_generated": 450,
  "total_cards": 1000,
  "eta_seconds": 28
}
```

### Artefacts générés

À la fin d'un job réussi, les artefacts suivants sont disponibles :

| Artefact | Description | Format |
|---|---|---|
| **PDF** | Toutes les cartes recto/verso en un seul fichier | PDF multi-pages |
| **ZIP** | Cartes individuelles (une carte = un PNG) | Archive ZIP |
| **Portail de vérification** | Portail offline avec registre SHA-256 | HTML + JSON statique |
| **Manifeste** | Liste structurée des bénéficiaires et codes | JSON |

---

## Cycle de vie d'un job

### Statuts de campagne

```
draft → preview_ready → generating → completed
                                   ↘ failed
```

| Statut | Description |
|---|---|
| `draft` | Campagne créée, design non finalisé |
| `preview_ready` | Design validé, prêt pour génération |
| `generating` | Job de génération en cours |
| `completed` | Génération terminée avec succès |
| `failed` | Génération échouée (voir logs) |

### Statuts de job

| Statut | Description |
|---|---|
| `pending` | Job créé, en attente de traitement |
| `processing` | Worker en cours de traitement |
| `generating` | Génération active des cartes |
| `packaging` | Assemblage PDF, ZIP, portail |
| `completed` | Job terminé, artefacts disponibles |
| `failed` | Échec (remboursement automatique du quota) |
| `cancelled` | Annulé manuellement avant traitement |

---

## Remboursement automatique en cas d'échec

En cas d'échec d'un job de génération, le quota consommé est **automatiquement remboursé** sur votre compte. Aucune démarche manuelle n'est nécessaire.

Le remboursement inclut :
- Le quota bénéficiaires décompté au lancement
- Les éventuels crédits SMS/WhatsApp pré-alloués au job

Un journal de remboursement est disponible dans **Facturation > Transactions**.

---

## Options DPI par campagne

Chaque job de génération peut spécifier une résolution DPI indépendante :

| DPI | Facteur de coût | Usage recommandé |
|---|---|---|
| 300 | ×1.0 | Partage numérique (WhatsApp, e-mail) |
| 450 | ×1.4 | Impression laser, badges plastifiés |
| 600 | ×2.0 | Impression PVC haute qualité |

---

## Gestion des campagnes multiples

Un compte programme peut gérer un nombre illimité de campagnes simultanées (dans la limite du quota total de bénéficiaires du plan).

Fonctionnalités de gestion :

- **Filtrage** par statut, type, date, préfixe
- **Recherche** par nom ou identifiant
- **Duplication** d'une campagne (copie des paramètres, nouveau préfixe)
- **Archivage** des campagnes terminées
- **Export** de la liste des campagnes en CSV

---

## Exemple complet via API

```bash
# 1. Créer la campagne
curl -X POST https://api.oomus.health/campaigns/ \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "MILD Distribution - Thiès 2026",
    "campaign_type": "mild",
    "prefix": "THS-MILD",
    "language": "fr",
    "template_id": "mild"
  }'

# 2. Lancer la génération
curl -X POST https://api.oomus.health/campaigns/camp_01HXYZ/jobs \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "beneficiaries": [...],
    "dpi": 300,
    "include_mpi_id": true
  }'

# 3. Vérifier le statut
curl -X GET https://api.oomus.health/jobs/job_01HXYZ \
  -H "Authorization: Bearer <VOTRE_TOKEN>"

# 4. Télécharger les artefacts (une fois "completed")
curl -X GET https://api.oomus.health/jobs/job_01HXYZ/download/zip \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -o "cartes_thies_2026.zip"
```

---

## Prochaines étapes

- [Card Studio](card-studio.md) — Personnaliser le design des cartes
- [Distribution multicanal](distribution.md) — Envoyer les cartes générées
- [Vérification hors ligne](verification.md) — Utiliser le portail de vérification
- [Référence API — Campagnes & Jobs](../api-reference/campaigns-and-jobs.md)
