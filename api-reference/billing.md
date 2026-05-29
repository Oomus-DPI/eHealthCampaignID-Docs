# Facturation

Endpoints de facturation, ledger comptable, modules et gestion du plan.

**Base URL :** `https://api.oomus.health`
**Authentification :** Tous les endpoints nécessitent `Authorization: Bearer <TOKEN>`

---

## GET /billing/balance

Retourne le solde, le plan et les quotas du compte authentifié.

**Réponse 200 OK :**

```json
{
  "account_id": "prog_01HXYZ123",
  "plan": "regional_ops",
  "plan_display_name": "Regional Command",
  "balance_fcfa": 45000,
  "monthly_subscription_fcfa": 75000,
  "quota": {
    "identities_total": 100000,
    "identities_used": 23450,
    "identities_remaining": 76550,
    "sms_total": 250000,
    "sms_used": 12300,
    "sms_remaining": 237700,
    "whatsapp_total": 100000,
    "whatsapp_used": 4500,
    "whatsapp_remaining": 95500,
    "verifications_total": 50000,
    "storage_gb_total": 50
  },
  "quota_reset_date": "2026-06-01T00:00:00Z",
  "currency": "XOF"
}
```

| Champ | Type | Description |
| --- | --- | --- |
| `balance_fcfa` | integer | Crédit disponible |
| `quota.identities_total` | integer | Quota mensuel (cartes Studio + DHIS2 cumulées) |
| `quota.sms_total` | integer | Quota SMS mensuel |
| `quota.whatsapp_total` | integer | Quota WhatsApp mensuel |
| `quota.verifications_total` | integer | Quota vérifications mensuel |

---

## GET /billing/transactions

Retourne l'historique des transactions du compte.

**Paramètres de requête :**

| Paramètre | Type | Description |
| --- | --- | --- |
| `limit` | integer | Nombre de résultats (défaut : 50) |
| `offset` | integer | Décalage pour pagination |

**Réponse 200 OK :**

```json
[
  {
    "id": "txn_01HXYZ111",
    "tx_type": "debit",
    "amount": -16000,
    "description": "Génération 2000 cartes — Campagne DKR-VAC (1800 facturées @ 8 FCFA, 200 dans le quota)",
    "card_count": 2000,
    "reference": "JOB:job_01|INV:a3f4c2...",
    "created_at": "2026-05-14T14:30:00Z"
  }
]
```

---

## GET /billing/ledger

Registre comptable complet — toutes les transactions avec leurs factures associées et flag d'audit.

**Paramètres de requête :**

| Paramètre | Type | Description |
| --- | --- | --- |
| `limit` | integer | Nombre d'entrées (défaut : 50) |
| `offset` | integer | Décalage pour pagination |
| `tx_type` | string | Filtrer par type de transaction |

**Réponse 200 OK :**

```json
{
  "programme_id": "prog_01HXYZ",
  "total_entries": 12,
  "debits_with_invoice": 10,
  "debits_without_invoice": 2,
  "entries": [
    {
      "id": "txn_01HXYZ111",
      "description": "Commande PVC 100 cartes Standard",
      "amount": -35000,
      "tx_type": "debit",
      "is_debit": true,
      "invoice_id": "a3f4c2d1-...",
      "audited": true,
      "invoice": {
        "invoice_number": "INV-PVC-202605-AB1234",
        "status": "paid",
        "total_fcfa": 35000,
        "issued_at": "2026-05-14T14:30:00Z",
        "invoice_type": "pvc_order",
        "signature_hash": "sha256:abc123..."
      },
      "created_at": "2026-05-14T14:30:00Z"
    }
  ]
}
```

| Champ | Type | Description |
| --- | --- | --- |
| `debits_with_invoice` | integer | Débits liés à une facture formelle |
| `debits_without_invoice` | integer | Débits sans facture (crédits, rechargements) |
| `audited` | boolean | `true` si une facture est associée ou si c'est un crédit |
| `invoice.signature_hash` | string | SHA-256 de l'intégrité de la facture |

---

## GET /billing/my-usage

Consommation mensuelle réelle (identités, SMS, WhatsApp) depuis les enregistrements `EngineUsageRecord`.

**Réponse 200 OK :**

```json
{
  "period": "2026-05",
  "identities_used": 23450,
  "studio_cards": 18200,
  "dhis2_cards": 5250,
  "sms_sent": 12300,
  "whatsapp_sent": 4500,
  "storage_gb_used": 3.2
}
```

---

## GET /billing/my-modules

Modules actifs du programme — fusion des modules inclus dans le plan et des modules activés manuellement.

**Réponse 200 OK :**

```json
[
  {
    "key": "sms_gateway",
    "label": "Distribution SMS Omnicanale",
    "included_in_plan": true,
    "plan": "regional_ops",
    "monthly_cost": 0,
    "is_active": true
  },
  {
    "key": "ai_fraud_detection",
    "label": "Détection d'Anomalies & Conformité",
    "included_in_plan": false,
    "monthly_cost": 60000,
    "is_active": true,
    "activated_at": "2026-04-01T00:00:00Z"
  }
]
```

| Champ | Type | Description |
| --- | --- | --- |
| `included_in_plan` | boolean | `true` si le module est inclus dans l'abonnement, `false` si acheté séparément |
| `monthly_cost` | integer | 0 si inclus dans le plan, sinon coût mensuel en FCFA |

---

## GET /billing/my-quota

Quotas mensuels détaillés (cartes, SMS, WhatsApp).

---

## GET /billing/quota-plans

Plans disponibles avec quotas et prix. Retourne les 4 plans actifs.

**Réponse 200 OK :**

```json
[
  {
    "plan": "starter",
    "display_name": "Essential",
    "monthly_subscription_fcfa": 25000,
    "studio_print_quota": 10000,
    "sms_quota": 50000,
    "whatsapp_quota": 10000,
    "verifications_quota": 5000,
    "storage_gb": 10
  },
  {
    "plan": "regional_ops",
    "display_name": "Regional Command",
    "monthly_subscription_fcfa": 75000,
    "studio_print_quota": 100000,
    "sms_quota": 250000,
    "whatsapp_quota": 100000,
    "verifications_quota": 50000,
    "storage_gb": 50
  },
  {
    "plan": "national_campaign",
    "display_name": "National Infrastructure",
    "monthly_subscription_fcfa": 250000,
    "studio_print_quota": 1000000,
    "sms_quota": 3000000,
    "whatsapp_quota": 1000000,
    "verifications_quota": 500000,
    "storage_gb": 500
  },
  {
    "plan": "sovereign_enterprise",
    "display_name": "Sovereign Cloud",
    "monthly_subscription_fcfa": 750000,
    "studio_print_quota": -1,
    "sms_quota": -1,
    "whatsapp_quota": -1,
    "verifications_quota": -1,
    "storage_gb": -1
  }
]
```

> `-1` signifie illimité. Les prix configurables admin via `PUT /billing/quota-plans/{plan}`.

---

## PUT /billing/quota-plans/{plan}

[Admin] Configure les quotas et prix d'un plan. Effectif immédiatement pour toutes les nouvelles opérations.

**Corps de la requête :**

```json
{
  "monthly_subscription_fcfa": 75000,
  "studio_print_quota": 100000,
  "sp_price_per_card": 10,
  "sp_dhis2_price": 10,
  "sp_quality_surcharge": 3
}
```

---

## POST /billing/change-plan

Change le plan du compte. Débite le prix mensuel complet du plan cible, active automatiquement les modules inclus, génère une facture signée.

**Corps de la requête :**

```json
{ "plan_alias": "national_campaign" }
```

**Réponse 200 OK :**

```json
{
  "previous_plan": "regional_ops",
  "new_plan": "national_campaign",
  "effective_date": "2026-05-15T10:00:00Z",
  "modules_activated": ["ai_fraud_detection", "sovereign_wallet", "mpi_registry"],
  "invoice_number": "INV-SUB-202605-XY1234"
}
```

---

## GET /billing/premium-modules

Catalogue de tous les modules premium disponibles.

---

## GET /billing/premium-modules/active

Modules activés pour le programme courant (plan + add-ons manuels).

---

## POST /billing/premium-modules/activate

Active un module premium. Génère automatiquement un débit + une facture formelle.

**Corps de la requête :**

```json
{ "module_key": "advanced_analytics" }
```

**Erreurs :**

| Code | Description |
| --- | --- |
| `400` | Module inconnu |
| `402` | Solde insuffisant |
| `409` | Module déjà actif |

---

## GET /billing/engine-usage

Consommation par moteur (Studio Print vs Campaign Delivery).

---

## GET /billing/v2/invoices

Liste des factures formelles émises pour le programme.

**Réponse 200 OK :**

```json
[
  {
    "id": "inv_01HXYZ",
    "invoice_number": "INV-GEN-202605-AB1234",
    "invoice_type": "generation",
    "status": "paid",
    "total_fcfa": 16000,
    "issued_at": "2026-05-14T14:30:00Z",
    "signature_hash": "sha256:abc123..."
  }
]
```

Types de factures : `subscription` · `module_activation` · `pvc_order` · `generation` · `platform_creation`

---

## GET /billing/v2/invoices/{id}/pdf

Télécharge le PDF d'une facture formelle (authentifié). Retourne `application/pdf`.

---

## Prochaines étapes

- [Plans & Fonctionnalités](../getting-started/plans-and-pricing.md) — Comparatif des plans
- [Authentification](authentication.md) — Gestion des tokens
- [Campagnes & Jobs](campaigns-and-jobs.md) — Utiliser votre quota
