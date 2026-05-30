# Analytics Plateforme — Vue admin

La section **Analytics Plateforme** donne aux administrateurs une vue agrégée de l'ensemble des métriques d'usage de la plateforme : activité de génération, consommation de ressources, revenu et patterns comportementaux de tous les programmes.

---

## Différence avec le dashboard utilisateur

| Aspect | Dashboard utilisateur | Analytics admin |
|---|---|---|
| **Périmètre** | Son programme uniquement | Tous les programmes |
| **Granularité** | Temps réel (30 s) | Agrégé (journalier, mensuel) |
| **Contenu** | KPI opérationnels (quota, jobs) | KPI business (revenu, usage global) |
| **Accès** | Tout utilisateur authentifié | `is_admin = true` uniquement |

---

## Statistiques globales de la plateforme

```bash
GET /api/v1/admin/stats
Authorization: Bearer <ADMIN_TOKEN>
```

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

## Métriques d'usage par programme

### Activité de génération

| Métrique | Source DB | Description |
|---|---|---|
| **Cartes générées** | `SUM(GenerationJob.card_count WHERE status='completed')` | Total cartes produites |
| **Taux de succès** | `completed / total_jobs × 100` | Fiabilité du moteur |
| **DPI moyen** | `AVG(GenerationJob.dpi)` | Résolution préférée |
| **Volume moyen par job** | `AVG(GenerationJob.card_count)` | Taille typique des campagnes |

### Consommation financière

| Métrique | Source DB | Description |
|---|---|---|
| **Revenus FCFA** | `SUM(Transaction.amount WHERE amount < 0)` | Débits cumulés |
| **Rechargements validés** | `SUM(RechargeRequest.amount WHERE status='validated')` | Flux entrants |
| **Soldes actifs** | `SUM(Programme.balance_fcfa WHERE status='active')` | Trésorerie client totale |

---

## Répartition par plan tarifaire

Le tableau de répartition affiche la distribution des comptes par plan :

| Plan | Programmes | % |
|---|---|---|
| `starter` | 28 | 59,6 % |
| `regional_ops` | 11 | 23,4 % |
| `national_campaign` | 6 | 12,8 % |
| `sovereign_enterprise` | 2 | 4,3 % |

---

## Tendances temporelles

### Cartes générées — Courbe mensuelle

Agrégation mensuelle des cartes produites sur les 12 derniers mois :

```bash
GET /api/v1/analytics/platform/cards-trend?granularity=monthly&months=12
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "labels": ["Jan 2026", "Fév 2026", "Mar 2026"],
  "series": {
    "cards_total": [125000, 240000, 310000],
    "jobs_total": [18, 34, 47],
    "success_rate": [94.4, 97.1, 98.3]
  }
}
```

### Revenus — Tendance

```bash
GET /api/v1/analytics/platform/revenue-trend?months=6
Authorization: Bearer <ADMIN_TOKEN>
```

---

## Usage des modules enterprise

Suivi de l'adoption des modules premium par les programmes :

| Module | Programmes actifs | Activations ce mois |
|---|---|---|
| AI Campaign Optimizer | 4 | +1 |
| Geospatial Command Center | 3 | 0 |
| Health Trust Score | 5 | +2 |
| Sovereign Wallet | 2 | 0 |
| AI Fraud Detection | 6 | +1 |

---

## Analytics DHIS2

Métriques d'intégration DHIS2 agrégées sur la plateforme :

| Métrique | Description |
|---|---|
| **Connexions actives** | Programmes avec une connexion DHIS2 valide |
| **Bénéficiaires synchronisés** | Total bénéficiaires importés depuis DHIS2 |
| **Dernière synchronisation** | Date de la sync la plus récente (tous programmes) |
| **Erreurs de sync** | Synchronisations ayant échoué (30 derniers jours) |

---

## Distribution multicanal — Analytics

| Canal | Messages envoyés | Taux de succès | Taux d'ouverture |
|---|---|---|---|
| **SMS** | 1 200 000 | 97,3 % | — |
| **WhatsApp** | 340 000 | 99,1 % | 84,2 % |
| **Email** | 85 000 | 96,8 % | 42,1 % |

---

## Export des données analytiques

Les données analytics peuvent être exportées pour traitement externe ou reporting institutionnel :

| Format | Endpoint | Contenu |
|---|---|---|
| **CSV** | `GET /api/v1/analytics/platform/export.csv` | Toutes les métriques en colonnes |
| **JSON** | `GET /api/v1/analytics/platform/export.json` | Structure complète |
| **Excel** | `GET /api/v1/analytics/platform/export.xlsx` | Onglets par catégorie |

---

## Alertes et seuils

L'admin peut configurer des seuils d'alerte sur les indicateurs clés :

| Indicateur | Seuil par défaut | Action |
|---|---|---|
| Taux d'échec jobs > | 10 % sur 24h | Alerte email admin |
| Programme solde < | 5 000 FCFA | Notification programme |
| Jobs bloqués > | 30 min en `pending` | Alerte worker |
| Tentatives scan frauduleuses > | 50/min | Blocage IP automatique |

---

## Prochaines étapes

- [Détection Fraude IA (admin)](fraud.md) — Alertes et anomalies de sécurité
- [Dashboard & Analytics (utilisateur)](../features/dashboard-analytics.md) — Vue programme
- [Référence API — Modules Enterprise](../api-reference/enterprise-modules.md)
