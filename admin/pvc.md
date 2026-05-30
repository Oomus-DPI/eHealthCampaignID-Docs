# Commandes PVC — Vue admin

La section **Commandes PVC** permet aux administrateurs de gérer le cycle de vie complet des commandes de cartes physiques en PVC passées par les programmes. Chaque commande génère automatiquement une facture traçable dans le Billing Ledger.

---

## Types d'impression disponibles

| Type | Label | Prix unitaire (défaut) | Quantité min. | Délai |
|---|---|---|---|---|
| `standard_pvc` | Cartes PVC Standard | 450 FCFA | 50 cartes | 7 jours |
| `offset_industriel` | Offset Industriel HD | 1 200 FCFA | 500 cartes | 21 jours |

> Les prix et délais sont configurables par l'admin via `PUT /api/v1/pvc/admin/prices` et persistent immédiatement en base de données.

---

## Cycle de vie d'une commande

```
pending → confirmed → printing → shipped → delivered
       ↘                                 ↘ cancelled
```

| Statut | Label affiché | Description |
|---|---|---|
| `pending` | Commande reçue | Commande créée, en attente de validation admin |
| `confirmed` | Commande validée | Confirmée par l'admin, en préparation |
| `printing` | Impression en cours | Fichiers envoyés à l'imprimeur |
| `shipped` | Expédiée | Colis remis au transporteur |
| `delivered` | Livrée | Réception confirmée |
| `cancelled` | Annulée | Commande annulée |

Chaque transition de statut est enregistrée dans `status_history` avec horodatage, note et flag de notification client.

---

## Toutes les commandes — Vue admin

```bash
GET /api/v1/pvc/admin/all?status=pending&print_type=standard_pvc&limit=100
Authorization: Bearer <ADMIN_TOKEN>
```

Paramètres disponibles :

| Paramètre | Type | Description |
|---|---|---|
| `status` | string | Filtrer par statut |
| `print_type` | string | `standard_pvc` ou `offset_industriel` |
| `limit` | integer | Nombre maximum de résultats (défaut 100) |

**Champs d'une commande :**

| Champ | Description |
|---|---|
| `id` | Identifiant unique `pvc_01HXYZ…` |
| `programme_id` | Programme propriétaire |
| `campaign_id` | Campagne associée (optionnel) |
| `print_type` | Type d'impression |
| `quantity` | Nombre de cartes commandées |
| `unit_price_fcfa` | Prix unitaire au moment de la commande |
| `total_fcfa` | Montant total débité |
| `delivery_address` | Adresse de livraison |
| `delivery_contact` | Nom du contact de livraison |
| `delivery_phone` | Téléphone de livraison |
| `status` | Statut actuel |
| `status_history` | Historique complet des transitions |
| `tracking_number` | Numéro de suivi transporteur |
| `estimated_delivery_at` | Date estimée de livraison |
| `notes` | Notes admin + référence facture |

---

## Mettre à jour le statut d'une commande

```bash
PATCH /api/v1/pvc/{order_id}/status
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "status": "shipped",
  "tracking_number": "DHL-1Z999AA10123456784",
  "note": "Colis remis à DHL Dakar — 30 mai 2026",
  "notify_client": true
}
```

- `tracking_number` est ajouté automatiquement dans la note d'historique
- `notify_client: true` déclenche une notification au programme (email / webhook)

---

## Configuration des prix (admin)

### Consulter les prix actuels

```bash
GET /api/v1/pvc/admin/prices
Authorization: Bearer <ADMIN_TOKEN>
```

### Modifier les prix

```bash
PUT /api/v1/pvc/admin/prices
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "standard_pvc": {
    "unit_price_fcfa": 480,
    "min_qty": 100,
    "lead_days": 7
  },
  "offset_industriel": {
    "unit_price_fcfa": 1100,
    "min_qty": 500,
    "lead_days": 21
  }
}
```

Les clés autorisées sont : `unit_price_fcfa`, `min_qty`, `lead_days`, `label`, `description`.

> **Validation :** `unit_price_fcfa` doit être positif, `min_qty > 0`, `lead_days ≥ 0`. Les prix sont vérifiés avant persistance.

---

## Facturation automatique

Chaque commande PVC déclenche automatiquement :

1. Un débit du solde FCFA du programme (`programme.balance_fcfa -= total_fcfa`)
2. Une transaction de type `pvc_order` dans le ledger
3. Une facture numérotée avec les détails de la commande

Le numéro de facture est enregistré dans le champ `notes` de la commande.

**Vérifier la facture d'une commande :**

```bash
GET /api/v1/billing/invoices?tx_type=pvc_order
Authorization: Bearer <PROGRAMME_TOKEN>
```

---

## Indicateurs admin

| KPI | Calcul |
|---|---|
| **Commandes en attente** | COUNT(PvcCardOrder WHERE status='pending') |
| **Cartes en impression** | SUM(quantity WHERE status='printing') |
| **Revenu PVC ce mois** | SUM(total_fcfa WHERE created_at >= début du mois) |
| **Délai moyen de livraison** | AVG(delivered_at - created_at) en jours |

---

## Prochaines étapes

- [Card Studio](../features/card-studio.md) — Personnaliser les cartes avant impression
- [Facturation](../api-reference/billing.md) — Référence API Billing Ledger
- [Vue d'ensemble Admin](overview.md)
