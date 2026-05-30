# DHIS2 Intégration — Vue admin

La section **DHIS2 Intégration (admin)** permet de superviser l'état de toutes les connexions DHIS2 configurées sur la plateforme, de monitorer les synchronisations et d'intervenir en cas d'erreur. Le module DHIS2 d'Oomus adopte une approche **safe-by-design** : toutes les opérations sont en lecture seule depuis DHIS2 — aucune écriture n'est effectuée dans le système source sans consentement explicite.

---

## Architecture DHIS2 safe (dhis2_safe)

```
DHIS2 Instance
     │
     │  (lecture seule — HMAC-signed requests)
     ▼
dhis2_safe module
     │
     ├─ Validation des credentials (chiffrement AES-256 au repos)
     ├─ Rate limiting (max 1 000 req/min par instance)
     ├─ Circuit breaker (seuil d'erreurs configurable)
     └─ Audit trail complet de chaque sync
          │
          ▼
     MPI / Campaign Engine
```

> Les credentials DHIS2 (URL, username, password) sont chiffrés en AES-256-GCM avant stockage en base. L'admin peut vérifier leur présence mais ne peut jamais les lire en clair.

---

## État des connexions DHIS2

```bash
GET /api/v1/dhis2/admin/connections
Authorization: Bearer <ADMIN_TOKEN>
```

**Réponse :**

```json
[
  {
    "programme_id": "prog_01HXYZ",
    "programme_name": "MSAS — Direction EPI",
    "dhis2_url": "https://dhis2.sante.gouv.sn",
    "status": "active",
    "last_sync_at": "2026-05-30T08:42:00Z",
    "last_sync_records": 12400,
    "error_count_24h": 0
  }
]
```

| Statut | Description |
|---|---|
| `active` | Connexion valide, synchronisations réussies |
| `degraded` | Erreurs intermittentes (circuit breaker partiellement ouvert) |
| `down` | Circuit breaker ouvert — instance DHIS2 inaccessible |
| `unconfigured` | Programme n'a pas encore configuré de connexion DHIS2 |

---

## Métriques de synchronisation globales

```bash
GET /api/v1/dhis2/admin/sync-stats?days=7
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "period_days": 7,
  "total_syncs": 340,
  "successful_syncs": 332,
  "failed_syncs": 8,
  "total_records_imported": 4500000,
  "avg_sync_duration_seconds": 14.2,
  "active_connections": 12
}
```

---

## Synchronisations récentes

```bash
GET /api/v1/dhis2/admin/recent-syncs?limit=50
Authorization: Bearer <ADMIN_TOKEN>
```

Chaque entrée de synchronisation inclut :

| Champ | Description |
|---|---|
| `sync_id` | Identifiant unique de la synchronisation |
| `programme_id` | Programme ayant déclenché la sync |
| `triggered_by` | `manual` / `scheduled` / `campaign_start` |
| `status` | `success` / `partial` / `failed` |
| `records_fetched` | Bénéficiaires récupérés depuis DHIS2 |
| `records_created` | Nouveaux enregistrements créés dans le MPI |
| `records_updated` | Enregistrements mis à jour |
| `duration_seconds` | Durée de la synchronisation |
| `error_message` | Message d'erreur si échec |
| `started_at` / `completed_at` | Horodatages |

---

## Diagnostiquer une synchronisation en échec

Étapes d'investigation recommandées :

1. **Vérifier les logs de sync** :
   ```bash
   GET /api/v1/dhis2/admin/sync-logs/{sync_id}
   ```
2. **Tester la connectivité** vers l'instance DHIS2 :
   ```bash
   POST /api/v1/dhis2/admin/test-connection/{programme_id}
   ```
3. **Vérifier le circuit breaker** — s'il est ouvert, attendre le cooldown (30 min par défaut) ou forcer la réinitialisation :
   ```bash
   POST /api/v1/dhis2/admin/reset-circuit-breaker/{programme_id}
   ```
4. **Consulter l'audit trail** :
   ```bash
   GET /api/v1/security/audit-logs?resource_type=dhis2_sync
   ```

---

## Causes d'erreur courantes

| Code d'erreur | Description | Résolution |
|---|---|---|
| `DHIS2_AUTH_FAILED` | Credentials invalides ou expirés | Demander au programme de reconfigurer ses credentials |
| `DHIS2_TIMEOUT` | Instance DHIS2 trop lente (> 30 s) | Réduire le `page_size` de la sync |
| `DHIS2_RATE_LIMITED` | Trop de requêtes vers l'instance | Augmenter l'intervalle entre syncs |
| `MPI_CONFLICT` | Doublon détecté non résolu | Résoudre dans le [Registre MPI](mpi.md) |
| `PROGRAM_NOT_FOUND` | Programme DHIS2 introuvable | Vérifier l'ID programme dans la config |

---

## Configuration globale DHIS2 (admin)

L'admin peut configurer les paramètres globaux du module DHIS2 via les `PlatformSetting` :

```bash
PUT /api/v1/admin/settings
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "dhis2.max_page_size": 1000,
  "dhis2.request_timeout_seconds": 30,
  "dhis2.circuit_breaker_threshold": 5,
  "dhis2.circuit_breaker_cooldown_minutes": 30
}
```

---

## Données DHIS2 importées — Champs mappés

| Champ DHIS2 | Champ MPI | Transformation |
|---|---|---|
| `firstName` | `first_name` | Normalisation UTF-8, trim |
| `lastName` | `last_name` | Normalisation UTF-8, trim |
| `birthDate` | `birth_date` | ISO 8601 → date |
| `trackedEntityInstance` | `dhis2_tei_id` | Lien MpiLink type=`dhis2` |
| `organisationUnit.code` | `district_code` | Mapping DHIS2 → code district |
| `gender` | `gender` | `MALE`/`FEMALE`/`OTHER` |
| `phoneNumber` | `phone` | E.164 si disponible |

---

## Prochaines étapes

- [Guide DHIS2 (utilisateur)](../integrations/dhis2.md)
- [Registre MPI Souverain (admin)](mpi.md)
- [Référence API — DHIS2](../api-reference/dhis2.md)
- [Architecture safe DHIS2](../security/overview.md)
