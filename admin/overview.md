# Panneau Admin — Vue d'ensemble

Le Panneau Administrateur d'Oomus CampaignID est une interface réservée aux utilisateurs dotés du rôle `is_admin = true`. Il centralise la gouvernance opérationnelle de la plateforme : supervision des programmes, validation des rechargements, configuration du moteur de facturation et suivi en temps réel de l'activité globale.

---

## Accès et authentification

L'accès au panneau admin requiert :

1. Un compte Programme avec `is_admin = true` (accordé manuellement par un super-admin)
2. Un token JWT valide obtenu via `POST /api/v1/auth/login`
3. Toutes les requêtes admin passent par le middleware `get_current_admin` — un token utilisateur ordinaire est systématiquement rejeté avec `403 Forbidden`

> Le flag admin ne peut pas être auto-accordé. Seul un admin existant peut élever un autre compte via `PATCH /api/v1/admin/programmes/{id}`.

---

## Tableau de bord — Statistiques globales

La page d'accueil du panneau admin affiche les indicateurs globaux de la plateforme en temps réel :

| KPI | Endpoint source | Description |
|---|---|---|
| **Programmes total** | `GET /api/v1/admin/stats` | Nombre de comptes programmes créés |
| **Campagnes total** | idem | Nombre total de campagnes sur la plateforme |
| **Jobs total** | idem | Nombre total de jobs de génération lancés |
| **Cartes générées** | idem | Somme des cartes produites tous programmes confondus |
| **Revenu FCFA** | idem | Somme des débits (transactions négatives) = chiffre d'affaires |

**Requête :**

```bash
curl -X GET https://api.oomus.health/api/v1/admin/stats \
  -H "Authorization: Bearer <ADMIN_TOKEN>"
```

**Réponse :**

```json
{
  "programmes_total": 47,
  "campaigns_total": 312,
  "jobs_total": 891,
  "cards_total": 2450000,
  "revenue_fcfa": 18750000.0
}
```

---

## Sections du panneau admin

| Section | Description |
|---|---|
| [Programmes & Comptes](overview.md#gestion-des-programmes) | Créer, modifier, suspendre ou supprimer des comptes programmes |
| [DHIS2 Intégration](dhis2.md) | Superviser les connexions DHIS2 et les synchronisations |
| [Portails de Vérification](verification-portals.md) | Gérer les portails de vérification déployés |
| [Analytics Plateforme](analytics.md) | Métriques d'usage agrégées sur toute la plateforme |
| [Détection Fraude IA](fraud.md) | Alertes d'anomalies et tableaux de bord de sécurité |
| [Toutes les Campagnes](campaigns.md) | Vue transversale de toutes les campagnes (tous programmes) |
| [Jobs de Génération](jobs.md) | Supervision des jobs en cours et historique global |
| [Commandes PVC](pvc.md) | Gestion des commandes d'impression physique PVC |
| [Registre MPI Souverain](mpi.md) | Administration du Master Patient Index |

---

## Gestion des programmes

### Lister tous les programmes

```bash
GET /api/v1/admin/programmes
```

Retourne la liste complète des comptes programmes triés par date de création décroissante.

### Créer un compte programme (admin)

```bash
POST /api/v1/admin/programmes
```

```json
{
  "name": "MSAS - Direction EPI",
  "email": "epi@sante.gouv.sn",
  "password": "TemporaryPass123!",
  "country": "SN",
  "phone": "+221771234567"
}
```

### Modifier un programme

```bash
PATCH /api/v1/admin/programmes/{programme_id}
```

Champs modifiables :

| Champ | Type | Description |
|---|---|---|
| `plan` | string | Plan tarifaire (`starter`, `regional_ops`, `national_campaign`, `sovereign_enterprise`) |
| `status` | string | Statut du compte (`active`, `suspended`, `pending`) |
| `is_admin` | boolean | Élever ou révoquer les droits admin (auto-révocation bloquée) |
| `name` | string | Nom de l'organisation |
| `email` | string | Email de connexion |
| `country` | string | Code pays ISO 3166-1 alpha-2 |
| `phone` | string | Numéro de téléphone |

> **Sécurité :** Un admin ne peut pas révoquer son propre flag `is_admin` — protection contre le verrouillage accidentel.

### Supprimer un programme

```bash
DELETE /api/v1/admin/programmes/{programme_id}
```

Restrictions :
- Impossible de supprimer son propre compte
- Impossible de supprimer un autre compte admin

---

## Gestion des rechargements

Les demandes de rechargement (solde FCFA) soumises par les programmes sont traitées manuellement par les admins.

### File des demandes en attente

```bash
GET /api/v1/admin/recharge-requests?status=pending
```

### Valider un rechargement

```bash
PATCH /api/v1/admin/recharge-requests/{request_id}/validate
```

```json
{
  "notes": "Virement Orange Money reçu - réf #OM2026051234"
}
```

Effet immédiat :
1. `programme.balance_fcfa` est crédité du montant de la demande
2. Une transaction de type `topup` est créée dans le ledger
3. Le statut de la demande passe à `validated`

### Rejeter un rechargement

```bash
PATCH /api/v1/admin/recharge-requests/{request_id}/reject
```

```json
{
  "notes": "Preuve de paiement non reçue après 48h"
}
```

---

## Configuration du moteur de facturation

L'admin peut ajuster les paramètres internes du moteur de simulation et de facturation, stockés en base de données (effectifs immédiatement) :

```bash
PUT /api/v1/admin/billing/engine-config
```

```json
{
  "infra_base_fcfa_per_worker": 15000,
  "bsp_sms_fcfa_per_msg": 8,
  "bsp_whatsapp_fcfa_per_msg": 14,
  "autoscaling_factor": 1.35,
  "sovereign_factor": 2.2
}
```

Voir la liste complète des paramètres dans [Moteur de simulation — Configuration admin](../features/simulation-engine.md#configuration-admin-du-moteur).

---

## Rôles et ségrégation des accès

| Action | Utilisateur ordinaire | Admin |
|---|---|---|
| Gérer ses propres campagnes | ✅ | ✅ |
| Voir les campagnes d'autres programmes | ❌ | ✅ |
| Valider des rechargements | ❌ | ✅ |
| Modifier un plan tarifaire | ❌ | ✅ |
| Configurer le moteur de facturation | ❌ | ✅ |
| Supprimer un programme | ❌ | ✅ (non-admin seulement) |
| Créer un compte admin | ❌ | ✅ |

---

## Prochaines étapes

- [DHIS2 Intégration (admin)](dhis2.md)
- [Portails de Vérification (admin)](verification-portals.md)
- [Jobs de Génération (admin)](jobs.md)
- [Référence API — Authentification](../api-reference/authentication.md)
