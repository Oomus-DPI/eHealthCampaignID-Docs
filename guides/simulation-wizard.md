# Campaign Simulation Wizard — Guide

Le moteur de simulation permet d'**estimer les coûts et ressources d'une campagne avant tout déploiement réel**, de générer une proforma officielle PDF/Excel et de soumettre le dossier au workflow de validation admin.

---

## Vue d'ensemble du workflow

```text
Institution
    │
    ▼
[1] Simulation créée (status: draft → computed)
    │
    ▼
[2] Proforma générée (PDF + Excel + JSON + YAML)
    │
    ▼
[3] Soumission admin (status: submitted)
    │
    ├─► Approuvé (status: approved)
    │       │
    │       ▼
    │   [4] Provisioning déclenché (status: provisioned)
    │
    ├─► Modification demandée (status: modification_requested)
    │       │
    │       ▼
    │   Institution corrige et re-soumet
    │
    └─► Rejeté (status: rejected)
```

---

## Étapes du wizard (6 étapes)

### Étape 1 — Choix du moteur

Deux modes disponibles :

**Campaign Delivery Mode**
Pour les campagnes numériques connectées : DHIS2 Tracker, WhatsApp, SMS, vérification QR temps réel.

**Studio Print Mode**
Pour la production de cartes imprimables haute résolution : 300 / 450 / 600 DPI, exports PDF/ZIP sécurisés.

---

### Étape 2 — Paramètres campagne

| Paramètre | Description |
| --------- | ----------- |
| Bénéficiaires | Nombre total de bénéficiaires |
| Cartes | Volume de cartes à générer |
| DPI | Résolution : 300 (×1.0) / 450 (×1.4) / 600 (×2.0) |
| Durée | Durée estimée de la campagne (mois) |
| Pays | Pays de déploiement |
| SMS | Volume de SMS à envoyer |
| WhatsApp | Volume de messages WhatsApp |
| Vérifications QR | Volume de scans QR attendus |

---

### Étape 3 — Infrastructure

| Paramètre | Description |
| --------- | ----------- |
| Environnement | Mutualisé (standard) ou Dédié (+150 000 FCFA/mois) |
| SLA tier | Standard (×1.0) / Enhanced (×1.35) / Premium (×1.75) / Critical (×2.5) |
| Workers Celery | Nombre de workers dédiés |
| Plateforme vérification dédiée | 85 000 FCFA/mois si activée |

---

### Étape 4 — Gouvernance RBAC

| Paramètre | Description |
| --------- | ----------- |
| Organisations | Nombre d'organisations (au-delà de 1 : 10 000 FCFA/org/mois) |
| Utilisateurs | Au-delà de 5 : 1 500 FCFA/utilisateur/mois |
| Niveaux d'approbation | Au-delà de 2 : 8 000 FCFA/niveau/mois |
| Rétention audit | Au-delà de 90 j : 5 000 FCFA par 30 j additionnels |

---

### Étape 5 — Modules Premium

Sélection des modules à inclure dans le budget de la simulation.

| Module | Clé | Coût/mois |
| ------ | --- | --------- |
| SMS Gateway | `sms_gateway` | 25 000 FCFA |
| WhatsApp BSP | `whatsapp_bsp` | 75 000 FCFA |
| Analytics Avancés | `advanced_analytics` | 35 000 FCFA |
| IA Détection Fraude | `ai_fraud_detection` | 60 000 FCFA |
| Cartes PVC Physiques | `physical_pvc_card` | 0 + 350 FCFA/carte |
| Portail Vérification Dédié | `dedicated_verify_platform` | 85 000 FCFA |
| SLA Avancé | `advanced_sla` | 50 000 FCFA |
| Hébergement Souverain | `sovereign_hosting` | 150 000 FCFA |

---

### Étape 6 — Résultats & Documents

La simulation calcule automatiquement :

- **Coût total estimé** — décomposé par poste (abonnement, overages, infra, RBAC, modules).
- **Analyse quota** — usage % par ressource, risques de dépassement (warning > 80 %, critical > 100 %).
- **Recommandation de plan** — si les volumes dépassent 80 % du quota actuel.
- **Ressources estimées** — workers, stockage, workers DHIS2.

Documents générables :

| Format | Contenu |
| ------ | ------- |
| PDF (ReportLab A4) | Proforma OOMUS-branded, bannière statut workflow |
| Excel (openpyxl) | 4 feuilles : Proforma · Infrastructure · Gouvernance RBAC · Décomposition coûts |
| JSON | Snapshot complet simulation (métadonnées, engine, volumes, facturation, workflow) |
| YAML | Même structure, format YAML pour provisioning infra |

---

## API — Création et soumission

### 1. Créer une simulation

```http
POST /api/v1/simulation
Authorization: Bearer <token>
Content-Type: application/json

{
  "engine_mode": "campaign_delivery",
  "beneficiaries": 50000,
  "cards": 50000,
  "render_scale": 1.4,
  "duration_months": 3,
  "country": "SN",
  "sms_volume": 150000,
  "whatsapp_volume": 30000,
  "qr_verifications": 25000,
  "environment": "shared",
  "sla_tier": "standard",
  "num_orgs": 2,
  "num_users": 8,
  "approval_levels": 3,
  "audit_retention_days": 180,
  "premium_modules": ["advanced_analytics", "sms_gateway"]
}
```

### 2. Générer la proforma

```http
POST /api/v1/simulation/{id}/proforma
```

### 3. Soumettre pour validation

```http
POST /api/v1/simulation/{id}/submit
```

---

## Workflow admin

Une fois soumise, la simulation apparaît dans la file d'attente admin (`GET /simulation/admin/all?status=submitted`).

L'admin peut :

- **Approuver** : `POST /simulation/admin/{id}/approve` (notes optionnelles)
- **Rejeter** : `POST /simulation/admin/{id}/reject` (motif obligatoire)
- **Demander modification** : `POST /simulation/admin/{id}/request-modification` (instructions obligatoires — la simulation retourne au statut `modification_requested` côté institution)

Après approbation, le provisioning peut être déclenché : `POST /simulation/{id}/provision`.

---

## Analyse des risques de dépassement

La simulation inclut un objet `quota_analysis` avec les risques identifiés :

```json
{
  "quota_analysis": {
    "beneficiary_usage_pct": 87.5,
    "sms_usage_pct": 112.0,
    "overage_risks": [
      { "resource": "sms", "severity": "critical", "message": "SMS dépasse le quota de 12%" },
      { "resource": "beneficiaries", "severity": "warning", "message": "Bénéficiaires à 87.5% du quota" }
    ],
    "plan_recommendation": "national_campaign"
  }
}
```

Sévérités :

- `warning` — usage > 80 % du quota
- `critical` — usage > 100 % du quota (dépassement certain)
