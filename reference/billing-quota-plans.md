# Billing & Quota Plans — v4.2

OOMUS CampaignID utilise un modèle de facturation **quota opérationnel** : abonnement mensuel fixe incluant des volumes d'usage, avec dépassements auto-calculés. Aucun prix à l'unité n'est exposé côté client.

---

## Plans disponibles

| Plan | Clé API | Abonnement mensuel |
| ---- | ------- | ------------------ |
| Starter | `starter` | 25 000 FCFA |
| Regional Ops | `regional_ops` | 75 000 FCFA |
| National Campaign | `national_campaign` | 250 000 FCFA |
| Sovereign Enterprise | `sovereign_enterprise` | 750 000 FCFA |

> **Rétrocompatibilité** : les valeurs legacy `standard`, `pro` et `enterprise` sont automatiquement résolues vers `starter`, `regional_ops` et `national_campaign`.

---

## Quotas inclus par plan

| Ressource | Starter | Regional Ops | National Campaign | Sovereign Enterprise |
| --------- | ------- | ------------ | ----------------- | -------------------- |
| Bénéficiaires | 10 000 | 100 000 | 1 000 000 | 10 000 000 |
| SMS | 50 000 | 500 000 | 5 000 000 | 50 000 000 |
| WhatsApp | 10 000 | 100 000 | 1 000 000 | 10 000 000 |
| Vérifications QR | 5 000 | 50 000 | 500 000 | 5 000 000 |
| Stockage | 10 Go | 50 Go | 500 Go | 5 To |

---

## Dépassements (overages)

Lorsque la consommation dépasse le quota inclus, des frais de dépassement s'appliquent automatiquement. Les taux sont configurables par l'administrateur plateforme depuis l'interface de gouvernance (`PUT /billing/quota-plans/{plan}`).

Les dépassements sont calculés à partir des données réelles de consommation issues de `EngineUsageRecord`.

---

## Infrastructure auto-calculée

Les coûts d'infrastructure (workers Celery, stockage, DPI processing, DHIS2, vérification) sont calculés automatiquement depuis `INFRA_AUTO_RATES` multipliés par un `infra_factor` configurable par plan. L'administrateur ajuste uniquement le facteur, pas les taux bruts.

---

## Deux moteurs métier

Chaque plan donne accès aux deux moteurs indépendants :

**Campaign Delivery Mode**
Usage : DHIS2 Tracker, WhatsApp, SMS, QR temps réel, workers & queues.

**Studio Print Mode**
Usage : templates sécurisés, DPI 300 / 450 / 600, exports PDF/ZIP, production batch.

Les multiplicateurs DPI s'appliquent sur le coût de génération :

| DPI | Multiplicateur |
| --- | -------------- |
| 300 | × 1.0 |
| 450 | × 1.4 |
| 600 | × 2.0 |

---

## Simulation avant engagement

Avant de choisir un plan ou de lancer une campagne, utilisez le moteur de simulation pour obtenir une décomposition complète des coûts estimés, une proforma PDF/Excel et une analyse des risques de dépassement.

Voir le [Guide Simulation Wizard](../guides/simulation-wizard.md).

---

## Gouvernance admin

| Endpoint | Description |
| -------- | ----------- |
| `GET /billing/quota-plans` | Lister les 4 plans (public) |
| `PUT /billing/quota-plans/{plan}` | **[Admin]** Modifier quotas, overage rates, infra_factor |

Champs configurables par l'admin :

- `beneficiary_quota` — volume bénéficiaires inclus
- `sms_quota` — volume SMS inclus
- `whatsapp_quota` — volume WhatsApp inclus
- `verification_quota` — volume vérifications inclus
- `overage_beneficiary_fcfa` — coût FCFA par bénéficiaire supplémentaire
- `overage_sms_fcfa` — coût FCFA par SMS supplémentaire
- `overage_whatsapp_fcfa` — coût FCFA par message WhatsApp supplémentaire
- `infra_factor` — multiplicateur infrastructure (défaut : 1.0)
- `monthly_subscription_fcfa` — abonnement mensuel de base

---

## Changement de plan

```http
POST /api/v1/billing/change-plan
Authorization: Bearer <token>
Content-Type: application/json

{ "plan": "regional_ops" }
```

Réponse :

```json
{
  "plan": "regional_ops",
  "fee_charged": 50000,
  "is_upgrade": true
}
```

En cas d'upgrade, des frais de passage correspondant à la différence d'abonnement sont débités du solde. Un solde insuffisant bloque l'opération (`HTTP 400`).

---

## Rechargements

Les rechargements sont soumis via `POST /billing/recharge-requests` et validés par l'administrateur plateforme. Modes acceptés : Mobile Money, Virement bancaire, Chèque, Espèces.
