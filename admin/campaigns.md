# Toutes les Campagnes — Vue admin

La section **Toutes les Campagnes** offre aux administrateurs une vue transversale de l'ensemble des campagnes de la plateforme, tous programmes confondus. Elle permet de superviser l'état opérationnel global, d'identifier les campagnes en anomalie et d'intervenir si nécessaire.

---

## Vue d'ensemble

Contrairement à la vue programme qui ne montre que ses propres campagnes, la vue admin agrège toutes les campagnes de tous les comptes programmes. Elle est accessible via :

```bash
GET /api/v1/campaigns/
Authorization: Bearer <ADMIN_TOKEN>
```

Les campagnes sont triées par date de création décroissante.

---

## Colonnes de la table admin

| Colonne | Description |
|---|---|
| **ID** | Identifiant unique `camp_01HXYZ…` |
| **Nom** | Nom descriptif de la campagne |
| **Programme** | Compte programme propriétaire |
| **Type** | Type de campagne (vaccination, mild, nutrition…) |
| **Préfixe** | Code alphanumérique unique de la campagne |
| **Langue** | fr / en / wo |
| **Statut** | État du cycle de vie |
| **DPI** | Résolution cible configurée |
| **Cartes** | Nombre de cartes générées |
| **Créée le** | Date de création |

---

## Filtres disponibles

| Filtre | Paramètre API | Exemple |
|---|---|---|
| Par statut | `?status=completed` | `draft`, `generating`, `completed`, `failed` |
| Par type | `?campaign_type=vaccination` | Tous les types de programme |
| Par programme | `?programme_id=prog_01HXYZ` | Campagnes d'un programme spécifique |
| Par préfixe | `?prefix=DKR` | Recherche préfixe |
| Plage de dates | `?from=2026-01-01&to=2026-06-30` | ISO 8601 |

---

## Types de campagne

| Type | Description |
|---|---|
| `vaccination` | Programme de vaccination (EPI, routine, masse) |
| `mild` | Distribution de moustiquaires imprégnées longue durée |
| `nutrition` | Suivi nutritionnel |
| `antenatal` | Soins prénataux |
| `hiv` | Programme HIV/PTME (données sensibles) |
| `assurance` | Assurance maladie universelle |
| `refugee` | Identification humanitaire réfugiés |
| `identity` | Identité sanitaire nationale |
| `lab` | Examens biologiques |
| `cps` | Protection sociale communautaire |
| `farmercard` | Santé agricole et rurale |

---

## Cycle de vie des campagnes

```
draft → preview_ready → generating → completed
                                   ↘ failed
```

| Statut | Icône | Description |
|---|---|---|
| `draft` | ○ | Campagne créée, design non finalisé |
| `preview_ready` | ◎ | Design validé, prêt pour génération |
| `generating` | ⟳ | Job de génération en cours (point animé) |
| `completed` | ✓ | Génération terminée avec succès |
| `failed` | ✗ | Génération échouée |

---

## Actions disponibles (admin)

En tant qu'admin, vous pouvez effectuer les mêmes actions qu'un programme sur n'importe quelle campagne :

### Consulter le détail

```bash
GET /api/v1/campaigns/{campaign_id}
Authorization: Bearer <ADMIN_TOKEN>
```

### Modifier les métadonnées

```bash
PATCH /api/v1/campaigns/{campaign_id}
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "name": "Campagne corrigée — Dakar 2026",
  "description": "Description mise à jour par admin"
}
```

> Le préfixe et le type de campagne ne peuvent plus être modifiés après création.

### Archiver une campagne

Les campagnes terminées peuvent être archivées pour alléger la vue opérationnelle des programmes. L'archivage est réversible.

---

## Indicateurs de santé globaux

En haut de la page, quatre KPI agrégés donnent une vue instantanée de la santé opérationnelle :

| KPI | Calcul |
|---|---|
| **Total campagnes** | COUNT(Campaign) tous programmes |
| **En cours** | COUNT(Campaign WHERE status='generating') |
| **Terminées** | COUNT(Campaign WHERE status='completed') |
| **Taux d'échec** | COUNT(failed) / COUNT(total) × 100 |

---

## Campagnes sensibles — Programme HIV

Les campagnes de type `hiv` sont signalées d'un badge spécial dans la vue admin. Leur accès aux données bénéficiaires est soumis à des restrictions supplémentaires conformément à la politique de protection des données (voir [Protection des données](../security/data-protection.md)).

---

## Prochaines étapes

- [Jobs de Génération (admin)](jobs.md) — Superviser les jobs de toutes les campagnes
- [Gestion des campagnes (utilisateur)](../features/campaigns.md) — Vue utilisateur
- [Référence API — Campagnes & Jobs](../api-reference/campaigns-and-jobs.md)
