# Référence API — Modules Enterprise

Tous les endpoints enterprise requièrent un token JWT valide via l'en-tête :
```
Authorization: Bearer <access_token>
```

Le token s'obtient via `POST /api/v1/auth/login`. Voir [Authentification](./authentication.md).

---

## Module 1 — AI Campaign Optimizer

**Base URL** : `/api/v1/ai`

### `GET /ai/insights`

Rapport d'analyse IA complet pour le programme courant.

**Réponse 200 :**
```json
{
  "health": {
    "score": 94.4,
    "level": "HIGH",
    "components": {
      "job_success": 86.1,
      "quota_health": 100.0,
      "balance_health": 99.9
    }
  },
  "card_forecast": {
    "predictions": [
      { "date": "2026-05-17", "value": 51.98 },
      { "date": "2026-05-18", "value": 54.58 }
    ],
    "confidence": 0.82,
    "r_squared": 0.82,
    "slope": 2.6
  },
  "spend_forecast": { "...": "même structure" },
  "quota_saturation": {
    "will_saturate": false,
    "days_until_saturation": null,
    "saturation_pct": 12.3
  },
  "card_anomalies": [
    { "index": 14, "value": 312.0, "zscore": 2.8 }
  ],
  "spend_anomalies": [],
  "recommendations": [
    {
      "category": "quota",
      "priority": "high",
      "title": "Quota mensuel à 75%",
      "body": "Consommation : 78% du quota mensuel...",
      "action_url": "billing"
    }
  ],
  "metrics": {
    "quota_pct": 12.3,
    "cards_used_month": 370,
    "quota_total": 3000,
    "job_success_rate": 86.1,
    "balance_fcfa": 1248500,
    "days_until_reset": 14
  }
}
```

---

### `GET /ai/health-score`

Score de santé composite uniquement (sans les séries temporelles).

**Réponse 200 :**
```json
{
  "score": 94.4,
  "level": "HIGH",
  "components": {
    "job_success": 86.1,
    "quota_health": 100.0,
    "balance_health": 99.9
  }
}
```

---

### `GET /ai/forecast`

Prévisions OLS cartes et dépenses sur 30 jours.

**Réponse 200 :**
```json
{
  "card_forecast": {
    "predictions": [ { "date": "YYYY-MM-DD", "value": 0.0 } ],
    "confidence": 0.0,
    "r_squared": 0.0,
    "slope": 0.0
  },
  "spend_forecast": { "...": "même structure en FCFA" }
}
```

> Si l'historique contient moins de 3 points : `{ "predictions": [], "confidence": 0.0, "r_squared": 0.0, "slope": 0.0 }`

---

### `GET /ai/recommendations`

Liste des recommandations contextuelles actives.

**Réponse 200 :**
```json
{
  "recommendations": [
    {
      "category": "quota | budget | risk | dhis2 | distribution",
      "priority": "critical | high | medium | low",
      "title": "string",
      "body": "string",
      "action_url": "billing | monitor | dhis2 | null"
    }
  ],
  "count": 1
}
```

---

### `GET /ai/anomalies`

Points aberrants détectés par Z-score (seuil 2,2σ) sur les 90 derniers jours.

**Réponse 200 :**
```json
{
  "card_anomalies": [
    { "index": 14, "value": 312.0, "zscore": 2.83 }
  ],
  "spend_anomalies": [],
  "total_anomalies": 1
}
```

---

## Module 2 — Geospatial Command Center

**Base URL** : `/api/v1/geo`

### `GET /geo/overview`

Vue d'ensemble géographique du programme courant.

**Réponse 200 :**
```json
{
  "programme_id": "uuid",
  "country": "Sénégal",
  "period": "2026-05",
  "total_campaigns": 90,
  "total_jobs": 72,
  "total_cards": 325,
  "success_rate": 86.1,
  "active_regions": 3,
  "coverage_score": 3
}
```

---

### `GET /geo/regions`

Stats par région dérivées des préfixes campagnes.

**Réponse 200 :**
```json
{
  "regions": [
    {
      "region": "Dakar",
      "country": "Sénégal",
      "period": "2026-05",
      "campaigns_count": 12,
      "cards_generated": 108,
      "beneficiaries_count": 91,
      "coverage_pct": 33.2,
      "distribution_success_rate": 86.1,
      "risk_score": 35.0
    }
  ],
  "count": 3
}
```

---

### `GET /geo/heatmap`

Matrice heatmap normalisée pour une métrique donnée.

**Paramètres :**
| Paramètre | Type | Valeurs | Défaut |
|---|---|---|---|
| `metric` | string | `cards_generated`, `coverage_pct`, `risk_score`, `distribution_success_rate`, `beneficiaries_count` | `cards_generated` |

**Réponse 200 :**
```json
{
  "cells": [
    {
      "region": "Dakar",
      "value": 108.0,
      "intensity": 1.0,
      "color": "#08BCF1"
    },
    {
      "region": "Thiès",
      "value": 54.0,
      "intensity": 0.5,
      "color": "#08BCF180"
    }
  ],
  "max_value": 108.0,
  "metric": "cards_generated",
  "label": "Cartes générées"
}
```

---

### `GET /geo/risk-zones`

Régions dont le score de risque dépasse 30 ou la couverture est inférieure à 40%.

**Réponse 200 :**
```json
{
  "zones": [
    {
      "region": "Kédougou",
      "risk_score": 65.0,
      "risk_level": "HIGH",
      "coverage_pct": 12.5,
      "distribution_success_rate": 86.1,
      "cards_generated": 36
    }
  ],
  "count": 1
}
```

---

### `GET /geo/live-events`

Flux des événements de distribution en temps réel.

**Paramètres :**
| Paramètre | Type | Défaut |
|---|---|---|
| `limit` | integer | 20 |

**Réponse 200 :**
```json
{
  "events": [
    {
      "id": "uuid",
      "region": "Dakar",
      "channel": "whatsapp | sms | qr",
      "event_type": "sent | delivered | failed",
      "count": 3,
      "created_at": "2026-05-17T10:23:00Z"
    }
  ],
  "count": 12
}
```

---

### `GET /geo/admin/overview` *(Admin)*

Vue agrégée cross-programmes (réservé aux admins).

**Réponse 200 :**
```json
{
  "total_programmes": 5,
  "total_cards": 8420,
  "total_regions": 12,
  "global_success_rate": 84.2,
  "programmes": [ { "...": "..." } ]
}
```

---

## Module 3 — Health Trust Score

**Base URL** : `/api/v1/mpi/trust`

### `GET /mpi/trust/me`

Trust Score de l'utilisateur courant (basé sur `programme.id`).

**Réponse 200 :**
```json
{
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "score": 72.5,
  "level": "MEDIUM",
  "computed_at": "2026-05-17T08:00:00Z",
  "components": {
    "kyc_verified": true,
    "has_nin": false,
    "programmes_count": 2
  }
}
```

---

### `GET /mpi/trust/{mpi_id}`

Trust Score pour un MPI ID spécifique.

**Paramètres :**
| Paramètre | Type | Description |
|---|---|---|
| `mpi_id` | string | Identifiant MPI (ex. `SN-DKR-26-9XQ7LM2A`) |

**Réponse 200 :** Même structure que `/mpi/trust/me`.

---

### `POST /mpi/trust/{mpi_id}/compute`

Recalculer le Trust Score à la demande.

**Réponse 200 :**
```json
{
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "score": 85.0,
  "level": "HIGH",
  "computed_at": "2026-05-17T10:30:00Z"
}
```

---

### `GET /mpi/trust/{mpi_id}/history`

Historique d'évolution du Trust Score.

**Réponse 200 :**
```json
{
  "history": [
    { "score": 55.0, "level": "LOW", "computed_at": "2026-04-01T..." },
    { "score": 72.5, "level": "MEDIUM", "computed_at": "2026-05-01T..." }
  ],
  "count": 2
}
```

---

### `GET /mpi/trust/{mpi_id}/risks`

Signaux de risque actifs pour un MPI ID.

**Réponse 200 :**
```json
{
  "risks": [
    {
      "risk_type": "probable_duplicate | demographic_inconsistency | missing_nin",
      "severity": "HIGH | MEDIUM | LOW",
      "description": "Un doublon probable (score 82/100) a été identifié.",
      "detected_at": "2026-05-10T..."
    }
  ],
  "count": 1
}
```

---

### `POST /mpi/trust/admin/recompute` *(Admin)*

Recalcul batch de tous les Trust Scores.

**Réponse 200 :**
```json
{
  "recomputed": 142,
  "duration_ms": 3420
}
```

---

## Module 4 — Sovereign Wallet

**Base URL** : `/api/v1/wallet`

### `POST /wallet/pass`

Créer ou récupérer un pass (idempotent sur `mpi_id`).

**Corps :**
```json
{
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "holder_name": "Fatou Diallo",
  "holder_id": "1234567890123",
  "campaign_id": "uuid",
  "campaign_name": "CPS Dakar 2026",
  "pass_type": "health | vaccination | distribution",
  "expires_at": "2027-05-17T00:00:00Z"
}
```

**Réponse 200/201 :**
```json
{
  "id": "uuid",
  "pass_id": "8A06586D268B79A3",
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "holder_name": "Fatou Diallo",
  "pass_type": "health",
  "status": "active",
  "qr_payload": "SN-DKR-26-9XQ7LM2A:8A06586D268B79A3:health",
  "qr_signature": "hmac-sha256-hex",
  "issued_at": "2026-05-17T10:00:00Z",
  "expires_at": "2027-05-17T00:00:00Z",
  "created_at": "2026-05-17T10:00:00Z"
}
```

---

### `GET /wallet/pass/{mpi_id}`

Pass actif d'un bénéficiaire par MPI ID.

**Réponse 200 :** Même structure que `POST /wallet/pass`.
**Réponse 404 :** Aucun pass actif pour ce MPI ID.

---

### `GET /wallet/passes`

Liste paginée des passes du programme.

**Paramètres :**
| Paramètre | Type | Valeurs | Défaut |
|---|---|---|---|
| `status` | string | `active`, `revoked`, `expired` | tous |
| `limit` | integer | 1–500 | 100 |
| `offset` | integer | ≥ 0 | 0 |

**Réponse 200 :**
```json
{
  "passes": [ { "...": "..." } ],
  "total": 42,
  "stats": {
    "total": 42,
    "active": 38,
    "revoked": 3,
    "expired": 1,
    "devices_registered": 12
  }
}
```

---

### `POST /wallet/pass/{pass_id}/revoke`

Révoquer un pass avec raison.

**Corps :**
```json
{
  "reason": "Bénéficiaire décédé | Doublon identifié | Erreur de données"
}
```

**Réponse 200 :**
```json
{
  "pass_id": "8A06586D268B79A3",
  "status": "revoked",
  "revoked_at": "2026-05-17T11:00:00Z",
  "reason": "Doublon identifié"
}
```

---

### `GET /wallet/pass/{pass_id}/bundle`

Bundle offline chiffré pour stockage IndexedDB.

**Réponse 200 :**
```json
{
  "pass_id": "8A06586D268B79A3",
  "bundle": {
    "algo": "xor-sha256",
    "encrypted": "<base64>",
    "hash": "<sha256-hex>",
    "expires_at": "2027-05-17T00:00:00Z"
  }
}
```

---

### `POST /wallet/device/register`

Enregistrer un appareil pour la synchronisation.

**Corps :**
```json
{
  "device_token": "unique-device-identifier",
  "device_name": "Tablet Terrain 01",
  "platform": "android | ios | web"
}
```

**Réponse 201 :**
```json
{
  "device_id": "uuid",
  "device_token": "unique-device-identifier",
  "registered_at": "2026-05-17T..."
}
```

---

### `GET /wallet/devices`

Appareils enregistrés du programme.

**Réponse 200 :**
```json
{
  "devices": [
    {
      "id": "uuid",
      "device_token": "...",
      "device_name": "Tablet Terrain 01",
      "platform": "android",
      "last_sync": "2026-05-16T..."
    }
  ],
  "count": 3
}
```

---

### `GET /wallet/sync`

Passes à synchroniser pour un appareil donné.

**Paramètres :**
| Paramètre | Type | Obligatoire |
|---|---|---|
| `device_token` | string | Oui |

**Réponse 200 :**
```json
{
  "passes": [ { "...": "..." } ],
  "sync_token": "uuid",
  "synced_at": "2026-05-17T..."
}
```

---

### `GET /wallet/sync/history`

Historique des synchronisations du programme.

**Réponse 200 :**
```json
{
  "history": [
    {
      "sync_token": "uuid",
      "device_token": "...",
      "passes_synced": 38,
      "synced_at": "2026-05-16T..."
    }
  ],
  "count": 12
}
```

---

### `GET /wallet/stats`

Statistiques du portefeuille du programme.

**Réponse 200 :**
```json
{
  "total": 42,
  "active": 38,
  "revoked": 3,
  "expired": 1,
  "devices_registered": 12
}
```

---

## Codes d'erreur communs

| Code | Signification |
|---|---|
| `401` | Token absent, invalide ou expiré |
| `403` | Permission insuffisante (ex. endpoint Admin) |
| `404` | Ressource introuvable (MPI ID, pass_id inconnu) |
| `422` | Paramètre obligatoire manquant (ex. `device_token` pour `/wallet/sync`) |
| `500` | Erreur interne — consulter les logs API |

---

## Voir aussi

- [Authentification](./authentication.md)
- [Campagnes & Jobs](./campaigns-and-jobs.md)
- [Facturation](./billing.md)
- [MPI](./mpi.md)
