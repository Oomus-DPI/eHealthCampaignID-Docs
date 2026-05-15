# API MPI — Identité souveraine

Cette section documente les endpoints de l'API MPI (Master Patient Index) d'Oomus CampaignID.

**Base URL :** `https://api.oomus.health`  
**Authentification :** Requis sauf `GET /mpi/verify/{token}` (endpoint public)

---

## POST /mpi/register

Enregistre un nouveau bénéficiaire dans le registre MPI souverain.

**Corps de la requête :**

```json
{
  "first_name": "Aminata",
  "last_name": "Diallo",
  "date_of_birth": "1995-03-15",
  "gender": "F",
  "region_code": "DKR",
  "country_code": "SN",
  "phone_number": "+221771234567",
  "external_ids": {
    "dhis2_uid": "abcDEF123gh"
  }
}
```

| Champ | Type | Obligatoire | Description |
|---|---|---|---|
| `first_name` | string | Oui | Prénom |
| `last_name` | string | Oui | Nom de famille |
| `date_of_birth` | date | Recommandé | Format ISO 8601 (YYYY-MM-DD) |
| `gender` | string | Recommandé | `M`, `F`, ou `U` (unknown) |
| `region_code` | string | Oui | Code région 3 lettres (ex: `DKR`, `THI`, `ZIG`) |
| `country_code` | string | Oui | Code pays ISO alpha-2 (ex: `SN`, `CI`, `ML`) |
| `phone_number` | string | Non | Numéro au format E.164 |
| `external_ids` | object | Non | Identifiants dans d'autres systèmes |

**Réponse 201 Created :**

```json
{
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "status": "created",
  "confidence": "new_identity",
  "created_at": "2026-05-15T09:00:00Z"
}
```

### Réponse 409 — Doublon détecté

Si le moteur probabiliste détecte un doublon probable (score ≥ 0.95), l'enregistrement est bloqué et le MPI existant est retourné :

```json
{
  "status": "duplicate_detected",
  "error": "Un bénéficiaire correspondant existe déjà dans le registre.",
  "existing_mpi_id": "SN-DKR-25-4HBNM7KL",
  "similarity_score": 0.97,
  "matched_fields": ["first_name", "last_name", "date_of_birth"],
  "action": "use_existing_or_review"
}
```

**Flux de résolution d'un doublon :**
1. Utilisez `existing_mpi_id` pour lier le nouveau système à l'identité existante
2. Ou appelez `POST /mpi/merge` si vous êtes certain que c'est la même personne
3. Ou appelez `POST /mpi/register` avec `force_create: true` si vous êtes certain que c'est une personne différente

---

## GET /mpi/search

Recherche dans le registre MPI.

**Paramètres de requête :**

| Paramètre | Type | Description |
|---|---|---|
| `q` | string | Recherche textuelle (nom, prénom) |
| `mpi_id` | string | Recherche par identifiant MPI exact |
| `region_code` | string | Filtrer par région |
| `date_of_birth` | date | Filtrer par date de naissance |
| `programme` | string | Filtrer par programme associé |
| `limit` | integer | Nombre de résultats (défaut : 20) |
| `offset` | integer | Décalage pour pagination |

**Réponse 200 OK :**

```json
{
  "total": 3,
  "items": [
    {
      "mpi_id": "SN-DKR-26-9XQ7LM2A",
      "first_name": "Aminata",
      "last_name": "Diallo",
      "date_of_birth": "1995-03-15",
      "gender": "F",
      "region_code": "DKR",
      "programme_count": 2,
      "created_at": "2026-05-15T09:00:00Z"
    }
  ]
}
```

---

## GET /mpi/{mpi_id}

Retourne les détails complets d'une identité MPI.

**Réponse 200 OK :**

```json
{
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "first_name": "Aminata",
  "last_name": "Diallo",
  "date_of_birth": "1995-03-15",
  "gender": "F",
  "region_code": "DKR",
  "country_code": "SN",
  "phone_number": "+221771234567",
  "programmes": [
    {
      "programme_type": "vaccination",
      "linked_at": "2026-01-10T00:00:00Z"
    },
    {
      "programme_type": "nutrition",
      "linked_at": "2026-03-05T00:00:00Z"
    }
  ],
  "external_ids": {
    "dhis2_uid": "abcDEF123gh"
  },
  "status": "active",
  "created_at": "2026-05-15T09:00:00Z",
  "updated_at": "2026-05-15T09:00:00Z"
}
```

---

## PATCH /mpi/{mpi_id}

Met à jour les attributs d'une identité MPI.

**Corps de la requête (champs optionnels) :**

```json
{
  "phone_number": "+221779876543",
  "region_code": "THI"
}
```

**Réponse 200 OK :** Identité MPI mise à jour (même schéma que GET /mpi/{mpi_id})

---

## POST /mpi/match

Vérifie la correspondance d'un jeu d'attributs avec le registre MPI et retourne le score de similarité.

**Corps de la requête :**

```json
{
  "first_name": "Aminata",
  "last_name": "Dialo",
  "date_of_birth": "1995-03-15",
  "gender": "F",
  "region_code": "DKR"
}
```

**Réponse 200 OK :**

```json
{
  "matches": [
    {
      "mpi_id": "SN-DKR-26-9XQ7LM2A",
      "similarity_score": 0.94,
      "matched_fields": ["last_name_fuzzy", "date_of_birth", "gender", "region_code"],
      "confidence": "probable"
    }
  ]
}
```

---

## POST /mpi/{mpi_id}/link-dhis2

Lie un UID DHIS2 à une identité MPI existante.

**Corps de la requête :**

```json
{
  "dhis2_uid": "abcDEF123gh",
  "dhis2_instance_url": "https://dhis2.sante.gov.sn"
}
```

**Réponse 200 OK :**

```json
{
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "dhis2_uid": "abcDEF123gh",
  "linked_at": "2026-05-15T10:00:00Z"
}
```

---

## GET /mpi/resolve-dhis2/{dhis2_uid}

Résout un UID DHIS2 vers l'identité MPI correspondante.

**Réponse 200 OK :**

```json
{
  "dhis2_uid": "abcDEF123gh",
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "resolved_at": "2026-05-15T10:00:00Z"
}
```

**Réponse 404 :** UID DHIS2 non lié à un MPI.

---

## POST /mpi/merge

Fusionne deux identités MPI en une (résolution manuelle de doublon).

**Corps de la requête :**

```json
{
  "source_mpi_id": "SN-DKR-26-8ZPLK9NQ",
  "target_mpi_id": "SN-DKR-26-9XQ7LM2A",
  "reason": "Doublon confirmé — même personne, deux enregistrements",
  "keep_target": true
}
```

`target_mpi_id` devient l'identité canonique. `source_mpi_id` est désactivé et redirige vers la cible.

**Réponse 200 OK :**

```json
{
  "canonical_mpi_id": "SN-DKR-26-9XQ7LM2A",
  "merged_mpi_id": "SN-DKR-26-8ZPLK9NQ",
  "merged_at": "2026-05-15T10:05:00Z",
  "programmes_migrated": 1
}
```

---

## GET /mpi/{mpi_id}/card

Retourne les données de la carte associée à une identité MPI.

**Réponse 200 OK :**

```json
{
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "card_code": "DKR-VAC-9XQ7LM2A",
  "campaign_id": "camp_01HXYZ456DEF",
  "status": "valid",
  "issued_at": "2026-05-15T09:10:48Z"
}
```

---

## GET /mpi/verify/{token} *(public — sans authentification)*

Vérifie un jeton QR de carte. Cet endpoint est **public** et ne nécessite aucun token d'authentification.

```bash
# Exemple — aucun en-tête Authorization requis
curl -X GET https://api.oomus.health/mpi/verify/a3f8c2e1d4b7f9e2a1c3...
```

**Réponse 200 OK (valide) :**

```json
{
  "status": "valid",
  "programme_count": 2,
  "verified_at": "2026-05-15T10:30:00Z"
}
```

**Réponse 200 OK (révoquée) :**

```json
{
  "status": "revoked",
  "verified_at": "2026-05-15T10:30:00Z"
}
```

**Réponse 404 (inconnue) :**

```json
{
  "status": "not_found",
  "message": "Ce jeton ne correspond à aucune carte connue."
}
```

> Cet endpoint ne retourne **jamais** de données personnelles (nom, MPI ID, date de naissance). Il confirme uniquement la validité cryptographique du jeton.

---

## GET /mpi/qr-config

Retourne la configuration QR active (style, taille, position).

**Réponse 200 OK :**

```json
{
  "enabled": true,
  "position": "bottom-right",
  "size": "standard",
  "token_type": "opaque_sha256"
}
```

---

## Prochaines étapes

- [Identité souveraine MPI](../features/mpi-sovereign-identity.md) — Documentation fonctionnelle
- [HL7 FHIR R4](../integrations/fhir-r4.md) — Interopérabilité
- [Protection des données](../security/data-protection.md)
