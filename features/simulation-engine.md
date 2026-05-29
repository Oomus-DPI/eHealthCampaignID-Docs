# Moteur de simulation v2

Le moteur de simulation v2 d'Oomus CampaignID adopte une approche **capability-first** : il recommande le plan adapté à votre programme, génère les documents proforma contractuels et suit un workflow d'approbation administrateur — sans jamais exposer les coûts unitaires techniques côté client.

---

## Pourquoi simuler ?

Avant de lancer un programme national de distribution de cartes, les décideurs ont besoin de réponses à des questions opérationnelles et institutionnelles :

- Quelle capacité d'infrastructure est nécessaire pour couvrir 500 000 bénéficiaires ?
- Quel plan garantit un SLA 99,9% sur 10 régions ?
- Quelles capacités sont incluses dans mon plan ?
- Comment produire les documents pour l'approbation budgétaire institutionnelle ?

---

## 5 modes de simulation

Chaque mode est conçu pour un périmètre d'intervention spécifique :

| Mode | Échelle bénéficiaires | Plan recommandé | Cas d'usage |
| --- | --- | --- | --- |
| **Rapide** | < 20 000 | Essential | Projet pilote, district, ONG locale |
| **Régionale** | 20 000 – 200 000 | Regional Command | Programme régional multi-districts |
| **Nationale** | 200 000 – 2 000 000 | National Infrastructure | Campagne gouvernementale nationale |
| **Multi-pays** | 2 000 000 – 20 000 000 | Sovereign Cloud | Fédération souveraine, programme continental |
| **Souveraine** | > 5 000 000 | Sovereign Cloud | Cloud dédié par pays |

---

## Moteur de recommandation de plan

Le moteur `recommend_plan()` calcule le plan optimal selon :

- Volume de bénéficiaires
- Nombre de régions couvertes
- Mode de simulation sélectionné
- Besoin de souveraineté des données

**Exemple de résultat :**

```json
{
  "plan": "national_campaign",
  "label": "National Infrastructure",
  "confidence": 94,
  "reasons": [
    "Volume de 450 000 bénéficiaires sur 12 régions",
    "Infrastructure nationale dédiée requise",
    "Capacités de communication à grande échelle"
  ],
  "capacities": {
    "identities": "1 000 000",
    "omnichannel_comms": "5 000 000 / mois",
    "verifications": "500 000 / mois",
    "storage": "500 Go"
  },
  "sla": "99.9%"
}
```

> Les prix unitaires ne sont jamais inclus dans la réponse du moteur — approche capability-first.

---

## Wizard de simulation — 6 étapes

### Étape 1 — Mode de simulation

Sélection du mode parmi les 5 types. Chaque carte de mode affiche :
- Échelle cible (nombre de bénéficiaires)
- Plan recommandé
- Description du cas d'usage

### Étape 2 — Paramètres

- Nombre de bénéficiaires cibles
- Nombre de régions
- Nombre d'agents de terrain
- Durée de la campagne
- Pays de déploiement
- Volumes SMS / WhatsApp / QR souhaités
- Niveau SLA requis

### Étape 3 — Infrastructure

| Option | Description |
|---|---|
| **Infrastructure partagée** | Ressources mutualisées entre programmes (coût réduit) |
| **Infrastructure dédiée** | Ressources exclusives pour votre programme (performance garantie) |

L'infrastructure dédiée est requise pour les SLA Premium et Critique.

### Étape 4 — Gouvernance RBAC

Configuration du modèle de gouvernance :
- Nombre d'organisations et d'utilisateurs
- Niveaux d'approbation requis (jusqu'à 10 niveaux configurables)
- Rétention des logs d'audit
- Périmètre géographique (national, régional, district)

### Étape 5 — Recommandation de plan

La carte de recommandation affiche :
- Plan recommandé avec confidence score
- Raisons de la recommandation
- Capacités incluses (identités, communications, vérifications, stockage)
- SLA garanti
- Modules complémentaires suggérés

### Étape 6 — Résultats & Devis

Deux parcours disponibles :
1. **Débit automatique** — si solde suffisant, activation immédiate du plan
2. **Devis institutionnel** — envoi à l'équipe commerciale Oomus, réponse sous 24h

Documents exportables :

| Document | Format | Contenu |
| --- | --- | --- |
| **Proforma PDF** | PDF | Devis institutionnel, capacités, SLA, workflow |
| **Simulation Excel** | XLSX (4 onglets) | Détail capacités, planning, gouvernance, coûts |
| **Configuration JSON** | JSON | Paramètres techniques complets |
| **Configuration YAML** | YAML | Version lisible de la configuration |

---

## Options DPI — Multiplicateurs

| DPI | Facteur | Usage |
| --- | --- | --- |
| 300 dpi | ×1.0 (référence) | Affichage numérique, distribution mobile |
| 450 dpi | ×1.4 | Impression laser, badges |
| 600 dpi | ×2.0 | Impression PVC haute qualité |

---

## Niveaux SLA

| Niveau | Disponibilité | Réponse P1 | Multiplicateur |
| --- | --- | --- | --- |
| **Standard** | 99,0% | 8h ouvrables | ×1.0 |
| **Enhanced** | 99,5% | 4 heures | ×1.3 |
| **Premium** | 99,9% | 1 heure | ×1.8 |
| **Critical** | 99,99% | 15 minutes | ×2.5 |

---

## Workflow d'approbation administrateur

```
Soumission simulation
        │
        ▼
   [submitted]
        │
        ▼ (administrateur Oomus examine)
   ┌────┴──────────────────────┐
   │                           │
   ▼                           ▼
[approved]           [modification_requested]
   │                           │
   │              (Programme corrige et resoumet)
   │
   ▼
[provisioning] → [provisioned]
```

| Statut | Description |
|---|---|
| `submitted` | Simulation soumise pour approbation |
| `approved` | Simulation approuvée, provisionnement lancé |
| `rejected` | Simulation rejetée (motif communiqué) |
| `modification_requested` | Modifications demandées par l'admin |
| `provisioning` | Infrastructure en cours de déploiement |
| `provisioned` | Programme opérationnel |

---

## Configuration admin du moteur

Les administrateurs peuvent configurer les paramètres internes du moteur de simulation via `PUT /api/v1/admin/billing/engine-config`. Ces valeurs sont stockées en `PlatformSetting` (persistées en DB, effectives immédiatement) :

| Paramètre | Description |
|---|---|
| `infra_base_fcfa_per_worker` | Coût base worker infrastructure |
| `bsp_sms_fcfa_per_msg` | Coût BSP par SMS |
| `bsp_whatsapp_fcfa_per_msg` | Coût BSP par message WhatsApp |
| `autoscaling_factor` | Multiplicateur autoscaling |
| `sovereign_factor` | Multiplicateur cloud souverain |
| `verify_platform_monthly` | Coût mensuel portail vérification |

---

## Prochaines étapes

- [Plans & Fonctionnalités](../getting-started/plans-and-pricing.md) — Choisir le bon plan
- [Facturation](../api-reference/billing.md) — API facturation et ledger
- [Dashboard & Analytics](dashboard-analytics.md) — Suivre l'utilisation réelle
