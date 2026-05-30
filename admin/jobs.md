# Jobs de Génération — Vue admin

La section **Jobs de Génération** permet aux administrateurs de superviser tous les jobs de génération de cartes lancés sur la plateforme, indépendamment du programme propriétaire. Elle expose l'état en temps réel du moteur de génération asynchrone (Celery + Redis).

---

## Endpoint admin

```bash
GET /api/v1/admin/jobs?limit=50
Authorization: Bearer <ADMIN_TOKEN>
```

Retourne les N derniers jobs triés par date de création décroissante. La limite par défaut est 50 ; elle peut être augmentée jusqu'à 500.

---

## Champs d'un job

| Champ | Type | Description |
|---|---|---|
| `id` | string | Identifiant unique `job_01HXYZ…` |
| `campaign_id` | string | Campagne parente |
| `programme_id` | string | Programme propriétaire |
| `status` | string | État du job (voir ci-dessous) |
| `card_count` | integer | Nombre de cartes à générer |
| `cards_generated` | integer | Cartes déjà produites |
| `progress` | float | Progression 0–100 % |
| `dpi` | integer | Résolution (300 / 450 / 600) |
| `error_message` | string | Message d'erreur si `failed` |
| `created_at` | datetime | Date de création |
| `started_at` | datetime | Début de traitement par un worker |
| `completed_at` | datetime | Date de complétion |

---

## Cycle de vie d'un job

```
pending → processing → generating → packaging → completed
                                              ↘ failed
                    ↘ cancelled
```

| Statut | Description |
|---|---|
| `pending` | Job créé, en attente d'un worker Celery |
| `processing` | Worker a pris en charge le job |
| `generating` | Génération active des cartes (PNG, QR) |
| `packaging` | Assemblage PDF, ZIP, portail de vérification |
| `completed` | Job terminé — artefacts disponibles |
| `failed` | Échec — quota automatiquement remboursé |
| `cancelled` | Annulé manuellement avant traitement |

---

## Indicateurs globaux

En haut de la page admin, les compteurs agrégés affichent :

| KPI | Calcul |
|---|---|
| **Jobs actifs** | COUNT(status IN ['pending','processing','generating','packaging']) |
| **Jobs complétés** | COUNT(status='completed') sur les 30 derniers jours |
| **Jobs en échec** | COUNT(status='failed') sur les 30 derniers jours |
| **Taux de succès** | completed / (completed + failed) × 100 |
| **Cartes générées aujourd'hui** | SUM(card_count WHERE status='completed' AND date=today) |

---

## Filtres disponibles

| Filtre | Paramètre | Valeurs |
|---|---|---|
| Par statut | `?status=failed` | `pending`, `processing`, `generating`, `packaging`, `completed`, `failed`, `cancelled` |
| Par programme | `?programme_id=prog_01HXYZ` | |
| Par campagne | `?campaign_id=camp_01HXYZ` | |
| Par DPI | `?dpi=600` | `300`, `450`, `600` |
| Jobs en échec uniquement | `?status=failed` | |
| Plage de dates | `?from=2026-01-01&to=2026-06-30` | |

---

## Remboursement automatique de quota

En cas d'échec d'un job (`status=failed`), le système rembourse automatiquement le quota consommé :

1. Le quota bénéficiaires décompté au lancement est recrédité
2. Les crédits SMS/WhatsApp pré-alloués sont restitués
3. Une transaction de type `refund` est inscrite dans le ledger du programme

**Aucune intervention manuelle n'est requise.** L'admin peut vérifier les remboursements effectués dans le ledger via `GET /api/v1/billing/transactions?type=refund`.

---

## Jobs bloqués — Procédure d'investigation

Un job reste en `pending` ou `processing` trop longtemps ? Étapes d'investigation :

1. **Vérifier le worker Celery** — s'assurer que les workers sont actifs (`celery inspect active`)
2. **Vérifier Redis** — connexion et longueur de la file (`redis-cli llen celery`)
3. **Consulter les logs** — `docker logs oomus-worker-1 --tail 100`
4. **Forcer l'échec** si nécessaire (via l'interface de gestion Celery ou en base de données)

---

## Suivi WebSocket (supervision)

Pour suivre en temps réel un job spécifique :

```
wss://api.oomus.health/ws/jobs/{job_id}?token={access_token}
```

Exemple de message WebSocket :

```json
{
  "event": "progress",
  "job_id": "job_01HXYZ789GHI",
  "status": "generating",
  "progress": 72,
  "cards_generated": 720,
  "total_cards": 1000,
  "eta_seconds": 14
}
```

---

## Artefacts disponibles après complétion

| Artefact | Endpoint | Format |
|---|---|---|
| **PDF multi-pages** | `GET /api/v1/jobs/{id}/download/pdf` | PDF |
| **Archive ZIP** | `GET /api/v1/jobs/{id}/download/zip` | ZIP |
| **Portail de vérification** | `GET /api/v1/jobs/{id}/download/portal` | HTML + JSON |
| **Manifeste JSON** | `GET /api/v1/jobs/{id}/manifest` | JSON |

---

## Prochaines étapes

- [Toutes les Campagnes (admin)](campaigns.md) — Vue transversale des campagnes
- [Analytics Plateforme (admin)](analytics.md) — Métriques d'usage agrégées
- [Référence API — Campagnes & Jobs](../api-reference/campaigns-and-jobs.md)
