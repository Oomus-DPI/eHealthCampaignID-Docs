# Facturation

Cette section documente les endpoints publics de facturation et de gestion du plan.

**Base URL :** `https://api.oomus.health`  
**Authentification :** Tous les endpoints nécessitent `Authorization: Bearer <TOKEN>`

---

## GET /billing/balance

Retourne le solde et les quotas du compte authentifié.

**Réponse 200 OK :**

```json
{
  "account_id": "usr_01HXYZ123ABC",
  "plan": "regional_ops",
  "plan_display_name": "Regional Ops",
  "balance_fcfa": 45000,
  "monthly_subscription_fcfa": 75000,
  "price_per_card_fcfa": 8,
  "quota": {
    "beneficiaries_total": 100000,
    "beneficiaries_used": 23450,
    "beneficiaries_remaining": 76550,
    "sms_total": 500000,
    "sms_used": 12300,
    "sms_remaining": 487700,
    "whatsapp_total": 100000,
    "whatsapp_used": 4500,
    "whatsapp_remaining": 95500,
    "storage_gb_total": 50,
    "storage_gb_used": 3.2,
    "storage_gb_remaining": 46.8
  },
  "quota_reset_date": "2026-06-01T00:00:00Z",
  "overage_price_per_beneficiary_fcfa": 15,
  "currency": "XOF"
}
```

| Champ | Type | Description |
|---|---|---|
| `balance_fcfa` | integer | Crédit disponible en FCFA |
| `monthly_subscription_fcfa` | integer | Montant de l'abonnement mensuel |
| `price_per_card_fcfa` | integer | Prix unitaire par carte générée |
| `quota.beneficiaries_total` | integer | Quota mensuel total de bénéficiaires |
| `quota.beneficiaries_used` | integer | Bénéficiaires consommés ce mois |
| `overage_price_per_beneficiary_fcfa` | integer | Prix par bénéficiaire en dépassement |

---

## GET /billing/transactions

Retourne l'historique des transactions du compte.

**Paramètres de requête (optionnels) :**

| Paramètre | Type | Description |
|---|---|---|
| `type` | string | Filtrer par type (`debit`, `credit`, `refund`, `subscription`) |
| `from` | date | Date de début (ISO 8601) |
| `to` | date | Date de fin (ISO 8601) |
| `limit` | integer | Nombre de résultats (défaut : 50, max : 200) |
| `offset` | integer | Décalage pour pagination |

**Réponse 200 OK :**

```json
{
  "total": 45,
  "items": [
    {
      "transaction_id": "txn_01HXYZ111",
      "type": "debit",
      "amount_fcfa": -16000,
      "description": "Génération 2000 cartes — Campagne DKR-VAC",
      "balance_after_fcfa": 45000,
      "created_at": "2026-05-14T14:30:00Z"
    },
    {
      "transaction_id": "txn_01HXYZ110",
      "type": "credit",
      "amount_fcfa": 100000,
      "description": "Rechargement manuel",
      "balance_after_fcfa": 61000,
      "created_at": "2026-05-10T09:00:00Z"
    },
    {
      "transaction_id": "txn_01HXYZ109",
      "type": "refund",
      "amount_fcfa": 800,
      "description": "Remboursement job échoué job_01HXYZ789",
      "balance_after_fcfa": 61000,
      "created_at": "2026-05-09T16:20:00Z"
    }
  ]
}
```

---

## GET /billing/quota-plans

Retourne la liste des plans tarifaires disponibles.

**Réponse 200 OK :**

```json
{
  "plans": [
    {
      "alias": "starter",
      "display_name": "Starter",
      "monthly_price_fcfa": 25000,
      "beneficiaries_quota": 10000,
      "sms_quota": 50000,
      "whatsapp_quota": 10000,
      "storage_gb": 10,
      "features": ["card_generation", "distribution", "mpi", "dhis2_basic"]
    },
    {
      "alias": "regional_ops",
      "display_name": "Regional Ops",
      "monthly_price_fcfa": 75000,
      "beneficiaries_quota": 100000,
      "sms_quota": 500000,
      "whatsapp_quota": 100000,
      "storage_gb": 50,
      "features": ["card_generation", "distribution", "mpi", "dhis2_full", "rbac_3_levels"]
    },
    {
      "alias": "national_campaign",
      "display_name": "National Campaign",
      "monthly_price_fcfa": 250000,
      "beneficiaries_quota": 1000000,
      "sms_quota": 5000000,
      "whatsapp_quota": 1000000,
      "storage_gb": 500,
      "features": ["card_generation", "distribution", "mpi", "dhis2_full", "rbac_4_levels", "advanced_analytics", "enhanced_sla"]
    },
    {
      "alias": "sovereign_enterprise",
      "display_name": "Sovereign Enterprise",
      "monthly_price_fcfa": 750000,
      "beneficiaries_quota": 10000000,
      "sms_quota": 50000000,
      "whatsapp_quota": 10000000,
      "storage_gb": 5120,
      "features": ["all_features", "sovereign_hosting", "dedicated_support", "critical_sla"]
    }
  ]
}
```

---

## POST /billing/change-plan

Change le plan tarifaire du compte.

**Corps de la requête :**

```json
{
  "plan_alias": "national_campaign"
}
```

Les alias disponibles sont : `starter`, `regional_ops`, `national_campaign`, `sovereign_enterprise`.

**Réponse 200 OK :**

```json
{
  "previous_plan": "regional_ops",
  "new_plan": "national_campaign",
  "effective_date": "2026-05-15T10:00:00Z",
  "message": "Plan changé avec succès. Le nouveau quota est effectif immédiatement."
}
```

**Erreurs :**

| Code | Description |
|---|---|
| `400` | Alias de plan invalide |
| `409` | Plan déjà actif |

---

## POST /billing/recharge-requests

Soumet une demande de rechargement de solde.

**Corps de la requête :**

```json
{
  "amount_fcfa": 200000,
  "payment_method": "bank_transfer",
  "reference": "BON-COMMANDE-2026-0042",
  "notes": "Rechargement pour campagne nationale vaccination"
}
```

**Réponse 202 Accepted :**

```json
{
  "request_id": "rch_01HXYZ999",
  "status": "pending",
  "amount_fcfa": 200000,
  "payment_method": "bank_transfer",
  "created_at": "2026-05-15T10:05:00Z",
  "message": "Votre demande de rechargement est en attente de traitement par l'équipe de facturation."
}
```

---

## GET /billing/premium-modules

Retourne la liste des modules premium disponibles et leur statut d'activation.

**Réponse 200 OK :**

```json
{
  "modules": [
    {
      "id": "sms_gateway",
      "name": "SMS Gateway",
      "description": "Accès étendu aux passerelles SMS supplémentaires",
      "price_fcfa_monthly": 15000,
      "status": "inactive"
    },
    {
      "id": "advanced_analytics",
      "name": "Advanced Analytics",
      "description": "Tableaux de bord analytiques avancés, exports CSV/Excel",
      "price_fcfa_monthly": 20000,
      "status": "active",
      "activated_at": "2026-04-01T00:00:00Z"
    },
    {
      "id": "ai_fraud_detection",
      "name": "AI Fraud Detection",
      "description": "Détection d'anomalies en temps réel par IA (IsolationForest)",
      "price_fcfa_monthly": 30000,
      "status": "inactive"
    },
    {
      "id": "sovereign_hosting",
      "name": "Sovereign Hosting",
      "description": "Hébergement dédié en infrastructure nationale ou régionale",
      "price_fcfa_monthly": null,
      "status": "quote_required"
    }
  ]
}
```

---

## POST /billing/premium-modules/activate

Active un module premium.

**Corps de la requête :**

```json
{
  "module_id": "advanced_analytics"
}
```

**Réponse 200 OK :**

```json
{
  "module_id": "advanced_analytics",
  "status": "active",
  "activated_at": "2026-05-15T10:10:00Z",
  "billing_note": "Le module sera facturé au prochain cycle mensuel."
}
```

**Erreurs :**

| Code | Description |
|---|---|
| `400` | Module inconnu ou non compatible avec votre plan |
| `402` | Solde insuffisant pour activer le module |
| `409` | Module déjà actif |

---

## Prochaines étapes

- [Plans & Tarification](../getting-started/plans-and-pricing.md) — Comparatif des plans
- [Authentification](authentication.md) — Gestion des tokens
- [Campagnes & Jobs](campaigns-and-jobs.md) — Utiliser votre quota
