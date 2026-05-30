# Registre MPI Souverain — Vue admin

Le **Master Patient Index (MPI)** est l'infrastructure d'identité numérique souveraine d'Oomus CampaignID. Il attribue à chaque bénéficiaire un identifiant unique inter-opérable (`MPI_ID`) résistant aux doublons, aux changements de nom et aux transferts de district. La vue admin donne accès au registre global cross-programmes et aux opérations de fédération d'identités.

---

## Format du MPI_ID

```
{ISO_PAYS}-{CODE_DISTRICT}-{AA}-{BASE36_ALÉATOIRE}

Exemple : SN-DKR-26-9XQ7LM2A
          │   │    │   └─ 8 caractères Base36 (collision ≈ 1/2,8 milliards)
          │   │    └─ Année d'enregistrement (2 chiffres)
          │   └─ Code district 3 lettres
          └─ Code pays ISO 3166-1 alpha-2
```

---

## Statistiques admin globales

```bash
GET /api/v1/mpi/admin/stats/global
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "total_records": 1250000,
  "verified_records": 980000,
  "linked_records": 340000,
  "programmes_using_mpi": 12,
  "verifications_last_30d": 45000,
  "merge_operations_last_30d": 234
}
```

| Métrique | Description |
|---|---|
| `total_records` | Identités enregistrées dans le MPI (tous programmes) |
| `verified_records` | Identités vérifiées manuellement par un admin |
| `linked_records` | Identités liées à un autre registre (DHIS2, CRVS…) |
| `programmes_using_mpi` | Programmes actifs qui utilisent le MPI |
| `verifications_last_30d` | Vérifications d'identité (scans QR) sur 30 jours |
| `merge_operations_last_30d` | Fusions de doublons effectuées |

---

## Registre global — Recherche cross-programmes

```bash
GET /api/v1/mpi/admin/all?limit=100&verified=true
Authorization: Bearer <ADMIN_TOKEN>
```

La vue admin liste **toutes** les identités, indépendamment du programme propriétaire. Filtres disponibles :

| Paramètre | Description |
|---|---|
| `limit` | Nombre maximum d'enregistrements |
| `offset` | Pagination |
| `verified` | `true` / `false` — identités vérifiées uniquement |
| `programme_id` | Filtrer par programme |
| `country` | Code pays ISO |

---

## Vérification manuelle d'une identité

L'admin peut valider manuellement une identité MPI (ex. : après contrôle documentaire) :

```bash
POST /api/v1/mpi/admin/{mpi_id}/verify
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "verification_method": "document_check",
  "notes": "CIN vérifiée le 2026-05-30 — agent terrain Dakar"
}
```

**Effet :**
- `mpi_record.is_verified = true`
- `mpi_record.verified_by = admin_id`
- Audit log créé : `action=mpi.admin_verify`

---

## Fédération d'identités cross-programmes

La fédération permet de lier deux identités MPI appartenant à des programmes différents lorsqu'elles correspondent au même bénéficiaire réel.

### Recherche de doublons inter-programmes

```bash
GET /api/v1/mpi/admin/federation/duplicates?threshold=85
Authorization: Bearer <ADMIN_TOKEN>
```

Retourne les paires d'identités ayant un score de similarité ≥ `threshold` (défaut 85 %) entre programmes distincts.

### Recherche globale

```bash
GET /api/v1/mpi/admin/federation/search?q=Fatou+Diallo&country=SN
Authorization: Bearer <ADMIN_TOKEN>
```

Recherche textuelle dans l'ensemble du registre global.

### Lier deux identités

```bash
POST /api/v1/mpi/admin/federation/link
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "source_mpi_id": "SN-DKR-26-9XQ7LM2A",
  "target_mpi_id": "SN-THS-25-4KR8NP1B",
  "link_type": "same_person",
  "confidence": 97.2,
  "notes": "Même bénéficiaire — district transféré Dakar→Thiès"
}
```

Types de lien disponibles :

| Type | Description |
|---|---|
| `same_person` | Même bénéficiaire, registres dupliqués |
| `household_member` | Membre du même ménage |
| `related` | Relation familiale |

### Consulter les liens d'une identité

```bash
GET /api/v1/mpi/admin/federation/links/{mpi_id}
Authorization: Bearer <ADMIN_TOKEN>
```

### Supprimer un lien de fédération

```bash
DELETE /api/v1/mpi/admin/federation/links/{link_id}
Authorization: Bearer <ADMIN_TOKEN>
```

---

## Fusion de doublons (merge)

La fusion consolide deux enregistrements en un seul, en préservant l'historique complet :

```bash
POST /api/v1/mpi/{source_id}/merge
Authorization: Bearer <PROGRAMME_TOKEN>
```

```json
{
  "target_id": "SN-DKR-26-XXXXXXXX",
  "reason": "Enregistrement en double détecté — même bénéficiaire",
  "confidence": 95.0
}
```

**Règles de fusion :**
- L'enregistrement `source` est marqué `merged_into: target_id`
- Toutes les références (jobs, campagnes) sont redirigées vers `target`
- Un `MpiMergeLog` est créé pour la traçabilité
- L'opération est irréversible — seul un admin peut créer un enregistrement de démerge

---

## Audit trail

Toutes les opérations admin sur le MPI sont journalisées dans la table `AuditLog` :

| Action | Déclencheur |
|---|---|
| `mpi.register` | Enregistrement standard |
| `mpi.register_forced` | Enregistrement forcé (doublon confirmé distinct) |
| `mpi.admin_verify` | Vérification manuelle admin |
| `mpi.merge` | Fusion de doublons |
| `mpi.federation.link` | Lien de fédération créé |
| `mpi.federation.unlink` | Lien de fédération supprimé |

```bash
GET /api/v1/security/audit-logs?resource_type=mpi_registry
Authorization: Bearer <ADMIN_TOKEN>
```

---

## Intégration DHIS2 — Résolution d'identité

Lors d'une synchronisation DHIS2, le système résout automatiquement les identités :

1. Recherche dans le MPI par `(first_name, last_name, birth_date, district)` avec score ≥ 85 %
2. Si trouvé : retourne le `MPI_ID` existant + crée un lien `MpiLink` type `dhis2`
3. Si non trouvé : crée un nouvel enregistrement MPI et retourne le nouveau `MPI_ID`

---

## Prochaines étapes

- [Identité souveraine MPI (utilisateur)](../features/mpi-sovereign-identity.md)
- [Guide DHIS2](../integrations/dhis2.md)
- [Référence API — MPI](../api-reference/mpi.md)
- [Sécurité — Vérification hors ligne](../security/offline-verification.md)
