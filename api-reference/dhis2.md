# API DHIS2

Cette section documente les endpoints d'intégration DHIS2 d'Oomus CampaignID.

**Base URL :** `https://api.oomus.health`  
**Authentification :** Tous les endpoints nécessitent `Authorization: Bearer <TOKEN>`

---

## POST /dhis2/config

Configure la connexion DHIS2 pour le compte authentifié.

**Corps de la requête :**

```json
{
  "url": "https://dhis2.sante.gov.sn",
  "username": "oomus_readonly",
  "password": "MotDePasseDHIS2"
}
```

**Réponse 200 OK :**

```json
{
  "config_id": "dhis2cfg_01HXYZ",
  "url": "https://dhis2.sante.gov.sn",
  "username": "oomus_readonly",
  "status": "configured",
  "created_at": "2026-05-15T09:00:00Z"
}
```

> Le mot de passe est stocké chiffré et n'est jamais renvoyé dans les réponses API.

---

## POST /dhis2/test-connection

Teste la connexion à l'instance DHIS2 configurée.

**Corps de la requête :**

```json
{
  "url": "https://dhis2.sante.gov.sn",
  "username": "oomus_readonly",
  "password": "MotDePasseDHIS2"
}
```

**Réponse 200 OK :**

```json
{
  "status": "connected",
  "dhis2_version": "2.41.2",
  "server_date": "2026-05-15T09:00:00",
  "programmes_available": 7,
  "organisation_units": 248
}
```

**Réponse 401 :**
```json
{
  "status": "auth_failed",
  "error": "Identifiants DHIS2 invalides."
}
```

**Réponse 503 :**
```json
{
  "status": "unreachable",
  "error": "Instance DHIS2 inaccessible depuis les serveurs Oomus."
}
```

---

## GET /dhis2/preview-enrollments

Prévisualise les enrollments disponibles dans un programme DHIS2.

**Paramètres de requête :**

| Paramètre | Type | Obligatoire | Description |
|---|---|---|---|
| `programme_uid` | string | Oui | UID du programme DHIS2 |
| `org_unit` | string | Non | Filtrer par unité organisationnelle |
| `from_date` | date | Non | Date de début (ISO 8601) |
| `to_date` | date | Non | Date de fin (ISO 8601) |
| `limit` | integer | Non | Nombre d'enrollments (défaut : 10) |

**Réponse 200 OK :**

```json
{
  "programme_uid": "P3jJH5Tu5VC",
  "programme_name": "Programme National de Vaccination",
  "total_enrollments": 12450,
  "preview": [
    {
      "enrollment_id": "xyz123abc",
      "enrollment_date": "2026-01-15",
      "org_unit_name": "Centre de santé Plateau",
      "attributes": {
        "MMD_PER_NAM": "Aminata",
        "MMD_PER_LST": "Diallo",
        "MMD_PER_DOB": "1995-03-15"
      }
    }
  ]
}
```

---

## POST /dhis2/assign-codes

Configure le mapping des attributs DHIS2 vers les champs de carte Oomus.

**Corps de la requête :**

```json
{
  "programme_uid": "P3jJH5Tu5VC",
  "mappings": [
    {
      "dhis2_attribute_code": "MMD_PER_NAM",
      "card_field": "first_name"
    },
    {
      "dhis2_attribute_code": "MMD_PER_LST",
      "card_field": "last_name"
    },
    {
      "dhis2_attribute_code": "MMD_PER_DOB",
      "card_field": "date_of_birth"
    },
    {
      "dhis2_attribute_code": "MMD_PER_SEX",
      "card_field": "gender"
    },
    {
      "dhis2_attribute_display_name": "Phone Number",
      "card_field": "phone_number"
    }
  ]
}
```

**Réponse 200 OK :**

```json
{
  "programme_uid": "P3jJH5Tu5VC",
  "mappings_configured": 5,
  "unmapped_card_fields": ["health_center", "zone"],
  "updated_at": "2026-05-15T09:15:00Z"
}
```

---

## POST /dhis2/generate-cards

Lance la génération de cartes pour les enrollments d'un programme DHIS2.

**Corps de la requête :**

```json
{
  "programme_uid": "P3jJH5Tu5VC",
  "template": "pulse",
  "dpi": 300,
  "include_mpi_id": true,
  "enrollment_filter": {
    "enrollment_date_from": "2026-01-01",
    "enrollment_date_to": "2026-05-15",
    "org_unit": "DiszpKrYNg8",
    "enrollment_status": "ACTIVE"
  }
}
```

| Champ | Type | Obligatoire | Description |
|---|---|---|---|
| `programme_uid` | string | Oui | UID du programme DHIS2 |
| `template` | string | Oui | `vital`, `emerald`, `pulse`, `mothercare`, `shield`, `nomad`, `aero`, `horizon`, `aurora`, `sovereign` |
| `dpi` | integer | Non | 300, 450 ou 600 (défaut : 300) |
| `include_mpi_id` | boolean | Non | Inclure l'ID MPI sur la carte (défaut : `true`) |
| `enrollment_filter` | object | Non | Filtres sur les enrollments à traiter |
| `sovereign_config` | object | Non | Pour le template `sovereign` uniquement : `{"accent_hex": "#D4AF37", "bg_hex": "#0D1B2A", "bg_deep_hex": "#060E1A", "font_scale": 1.0, "max_attrs": 4}` |

**Réponse 202 Accepted :**

```json
{
  "task_id": "task_dhis2_01HXYZ",
  "programme_uid": "P3jJH5Tu5VC",
  "status": "pending",
  "estimated_cards": 1250,
  "created_at": "2026-05-15T09:20:00Z"
}
```

---

## GET /dhis2/tasks/{task_id}

Retourne le statut d'une tâche de génération DHIS2.

**Réponse 200 OK :**

```json
{
  "task_id": "task_dhis2_01HXYZ",
  "programme_uid": "P3jJH5Tu5VC",
  "status": "generating",
  "progress": 72,
  "cards_generated": 900,
  "total_cards": 1250,
  "mpi_resolved": 850,
  "mpi_created": 50,
  "mpi_pending_review": 12,
  "eta_seconds": 45,
  "started_at": "2026-05-15T09:20:05Z"
}
```

---

## GET /dhis2/preview-card-png

Génère un aperçu PNG d'une carte avec les données réelles d'un enrollment DHIS2.

**Paramètres de requête :**

| Paramètre | Type | Obligatoire | Description |
|---|---|---|---|
| `enrollment_id` | string | Oui | UID de l'enrollment DHIS2 |
| `template` | string | Oui | `vital`, `emerald`, `pulse`, `mothercare`, `shield`, `nomad`, `aero`, `horizon`, `aurora`, `sovereign` |
| `dpi` | integer | Non | 300, 450 ou 600 (défaut : 300) |

**Réponse 200 OK :** Image PNG (Content-Type: `image/png`)

```bash
curl -X GET "https://api.oomus.health/dhis2/preview-card-png?enrollment_id=xyz123&template=pulse&dpi=300" \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  --output apercu_carte.png
```

---

## POST /dhis2/sync-schedule

Configure le calendrier de synchronisation automatique.

**Corps de la requête :**

```json
{
  "programme_uid": "P3jJH5Tu5VC",
  "interval_minutes": 30,
  "enabled": true,
  "auto_generate_cards": false
}
```

Valeurs de `interval_minutes` : `10`, `15`, `30`, `60`, `180`

**Réponse 200 OK :**

```json
{
  "programme_uid": "P3jJH5Tu5VC",
  "interval_minutes": 30,
  "enabled": true,
  "next_sync": "2026-05-15T09:50:00Z",
  "auto_generate_cards": false
}
```

---

## POST /dhis2/send-card

Envoie une carte générée à un bénéficiaire via WhatsApp ou SMS.

**Corps de la requête :**

```json
{
  "enrollment_id": "xyz123abc",
  "channel": "whatsapp",
  "phone_number": "+221771234567",
  "message": "Bonjour {first_name}, voici votre carte de vaccination. Conservez ce message."
}
```

Valeurs de `channel` : `whatsapp`, `sms`

**Réponse 200 OK :**

```json
{
  "enrollment_id": "xyz123abc",
  "channel": "whatsapp",
  "status": "sent",
  "message_id": "meta_msg_01HXYZ",
  "sent_at": "2026-05-15T09:25:00Z"
}
```

---

## GET /dhis2/google-wallet/{enrollment_id}

Génère un lien Google Wallet pour un enrollment DHIS2.

**Réponse 200 OK :**

```json
{
  "enrollment_id": "xyz123abc",
  "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "jwt_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "valid_until": "2027-05-15T00:00:00Z"
}
```

---

## POST /dhis2/google-wallet/bulk

Génère des liens Google Wallet pour plusieurs enrollments (max 100 par requête).

**Corps de la requête :**

```json
{
  "enrollment_ids": ["xyz123abc", "def456ghi", "jkl789mno"],
  "programme_uid": "P3jJH5Tu5VC"
}
```

**Réponse 200 OK :**

```json
{
  "batch_id": "batch_01HXYZ",
  "total": 3,
  "processed": 3,
  "wallet_links": [
    {
      "enrollment_id": "xyz123abc",
      "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGci..."
    },
    {
      "enrollment_id": "def456ghi",
      "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGci..."
    }
  ]
}
```

---

## Prochaines étapes

- [Guide d'intégration DHIS2](../integrations/dhis2.md) — Guide pas-à-pas
- [Intégration DHIS2 — Fonctionnalités](../features/dhis2-integration.md)
- [Distribution multicanal](../features/distribution.md)
