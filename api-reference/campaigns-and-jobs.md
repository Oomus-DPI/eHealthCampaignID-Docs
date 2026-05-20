# Campagnes & Jobs

Cette section documente les endpoints pour gérer les campagnes et les jobs de génération de cartes.

**Base URL :** `https://api.oomus.health`  
**Authentification :** Toutes les requêtes nécessitent `Authorization: Bearer <TOKEN>`

---

## Campagnes

### POST /campaigns/

Crée une nouvelle campagne.

**Corps de la requête :**

```json
{
  "name": "Vaccination EPI - Dakar 2026",
  "campaign_type": "vaccination",
  "prefix": "DKR-VAC",
  "language": "fr",
  "template_id": "vaccination",
  "description": "Campagne pilote district Dakar Plateau"
}
```

| Champ | Type | Obligatoire | Description |
|---|---|---|---|
| `name` | string | Oui | Nom de la campagne (2–200 caractères) |
| `campaign_type` | string | Oui | Type parmi les types supportés |
| `prefix` | string | Oui | Code unique 3–8 caractères majuscules |
| `language` | string | Oui | `fr`, `en`, ou `wo` |
| `template_id` | string | Oui | Identifiant du modèle de carte |
| `description` | string | Non | Description optionnelle |

**Réponse 201 Created :**

```json
{
  "id": "camp_01HXYZ456DEF",
  "name": "Vaccination EPI - Dakar 2026",
  "campaign_type": "vaccination",
  "prefix": "DKR-VAC",
  "language": "fr",
  "template_id": "vaccination",
  "status": "draft",
  "total_cards": 0,
  "created_at": "2026-05-15T09:05:00Z",
  "updated_at": "2026-05-15T09:05:00Z"
}
```

**Erreurs :**

| Code | Description |
|---|---|
| `400` | Préfixe déjà utilisé dans votre organisation |
| `402` | Quota de bénéficiaires épuisé |
| `404` | Plan non trouvé ou inactif |
| `422` | Erreur de validation (champs manquants ou invalides) |

---

### GET /campaigns/

Retourne la liste des campagnes du compte authentifié.

**Paramètres de requête (optionnels) :**

| Paramètre | Type | Description |
|---|---|---|
| `status` | string | Filtrer par statut (`draft`, `completed`, etc.) |
| `campaign_type` | string | Filtrer par type |
| `limit` | integer | Nombre de résultats (défaut : 20, max : 100) |
| `offset` | integer | Décalage pour pagination |

**Réponse 200 OK :**

```json
{
  "total": 12,
  "limit": 20,
  "offset": 0,
  "items": [
    {
      "id": "camp_01HXYZ456DEF",
      "name": "Vaccination EPI - Dakar 2026",
      "campaign_type": "vaccination",
      "prefix": "DKR-VAC",
      "status": "completed",
      "total_cards": 1250,
      "created_at": "2026-05-15T09:05:00Z"
    }
  ]
}
```

---

### GET /campaigns/{campaign_id}

Retourne les détails d'une campagne spécifique.

**Réponse 200 OK :**

```json
{
  "id": "camp_01HXYZ456DEF",
  "name": "Vaccination EPI - Dakar 2026",
  "campaign_type": "vaccination",
  "prefix": "DKR-VAC",
  "language": "fr",
  "template_id": "vaccination",
  "status": "completed",
  "total_cards": 1250,
  "jobs": [
    {
      "job_id": "job_01HXYZ789GHI",
      "status": "completed",
      "cards_generated": 1250,
      "created_at": "2026-05-15T09:10:00Z"
    }
  ],
  "created_at": "2026-05-15T09:05:00Z",
  "updated_at": "2026-05-15T09:30:00Z"
}
```

---

### PATCH /campaigns/{campaign_id}

Met à jour une campagne existante (nom, description uniquement — le préfixe et le type sont immuables).

**Corps de la requête :**

```json
{
  "name": "Vaccination EPI - Dakar 2026 (Révisé)",
  "description": "Version révisée après validation terrain"
}
```

---

### DELETE /campaigns/{campaign_id}

Archive une campagne (ne supprime pas les cartes générées).

**Réponse 204 No Content**

---

## Jobs de génération

### POST /campaigns/{campaign_id}/jobs

Lance un nouveau job de génération de cartes.

**Corps de la requête :**

```json
{
  "beneficiaries": [
    {
      "first_name": "Aminata",
      "last_name": "Diallo",
      "date_of_birth": "2020-03-15",
      "beneficiary_id": "BEN-001",
      "phone_number": "+221771234567"
    },
    {
      "first_name": "Moussa",
      "last_name": "Sow",
      "date_of_birth": "2019-07-22",
      "beneficiary_id": "BEN-002"
    }
  ],
  "dpi": 300,
  "include_mpi_id": true
}
```

| Champ | Type | Obligatoire | Description |
|---|---|---|---|
| `beneficiaries` | array | Oui | Liste des bénéficiaires (max 10 000 par job) |
| `dpi` | integer | Non | 300, 450 ou 600 (défaut : 300) |
| `include_mpi_id` | boolean | Non | Inclure l'identifiant MPI sur la carte (défaut : true) |

**Réponse 202 Accepted :**

```json
{
  "job_id": "job_01HXYZ789GHI",
  "campaign_id": "camp_01HXYZ456DEF",
  "status": "pending",
  "total_cards": 2,
  "quota_consumed": 2,
  "created_at": "2026-05-15T09:10:00Z"
}
```

**Erreurs :**

| Code | Description |
|---|---|
| `402` | Quota de bénéficiaires insuffisant pour cette génération |
| `404` | Campagne non trouvée ou plan introuvable |
| `422` | Données bénéficiaires invalides (champs manquants, format incorrect) |

---

### GET /jobs/{job_id}

Retourne le statut et la progression d'un job.

**Réponse 200 OK (job en cours) :**

```json
{
  "job_id": "job_01HXYZ789GHI",
  "campaign_id": "camp_01HXYZ456DEF",
  "status": "generating",
  "progress": 65,
  "cards_generated": 1,
  "total_cards": 2,
  "eta_seconds": 12,
  "started_at": "2026-05-15T09:10:05Z"
}
```

**Réponse 200 OK (job terminé) :**

```json
{
  "job_id": "job_01HXYZ789GHI",
  "campaign_id": "camp_01HXYZ456DEF",
  "status": "completed",
  "progress": 100,
  "cards_generated": 2,
  "total_cards": 2,
  "artifacts": {
    "pdf_url": "https://api.oomus.health/jobs/job_01HXYZ789GHI/download/pdf",
    "zip_url": "https://api.oomus.health/jobs/job_01HXYZ789GHI/download/zip",
    "verification_portal_url": "https://api.oomus.health/jobs/job_01HXYZ789GHI/download/portal",
    "manifest_url": "https://api.oomus.health/jobs/job_01HXYZ789GHI/download/manifest"
  },
  "started_at": "2026-05-15T09:10:05Z",
  "completed_at": "2026-05-15T09:10:48Z"
}
```

**Réponse 200 OK (job échoué) :**

```json
{
  "job_id": "job_01HXYZ789GHI",
  "status": "failed",
  "error": "Échec de la génération : données de template incomplètes",
  "quota_refunded": 2,
  "failed_at": "2026-05-15T09:10:30Z"
}
```

---

### GET /jobs/{job_id}/download/{artifact}

Télécharge un artefact de génération.

Valeurs de `{artifact}` : `pdf`, `zip`, `portal`, `manifest`

**Réponse 200 OK :** Contenu binaire du fichier avec les en-têtes Content-Type et Content-Disposition appropriés.

---

### WebSocket — Progression en temps réel

Connectez-vous au WebSocket pour recevoir les mises à jour en direct sans polling :

```
wss://api.oomus.health/ws/jobs/{job_id}?token={access_token}
```

**Événements reçus :**

```json
{
  "event": "progress",
  "job_id": "job_01HXYZ789GHI",
  "status": "generating",
  "progress": 45,
  "cards_generated": 1,
  "total_cards": 2,
  "eta_seconds": 18
}
```

```json
{
  "event": "completed",
  "job_id": "job_01HXYZ789GHI",
  "status": "completed",
  "progress": 100,
  "cards_generated": 2,
  "artifacts": { ... }
}
```

```json
{
  "event": "failed",
  "job_id": "job_01HXYZ789GHI",
  "status": "failed",
  "error": "Erreur lors de la génération",
  "quota_refunded": 2
}
```

---

## Exemple complet : lancer une génération et suivre son statut

```bash
# Étape 1 : S'authentifier
TOKEN=$(curl -s -X POST https://api.oomus.health/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "mon@email.sn", "password": "MonMDP!"}' \
  | jq -r '.access_token')

# Étape 2 : Créer la campagne
CAMP_ID=$(curl -s -X POST https://api.oomus.health/campaigns/ \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Génération",
    "campaign_type": "vaccination",
    "prefix": "TEST01",
    "language": "fr",
    "template_id": "vaccination"
  }' | jq -r '.id')

# Étape 3 : Lancer le job
JOB_ID=$(curl -s -X POST https://api.oomus.health/campaigns/$CAMP_ID/jobs \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "beneficiaries": [
      {"first_name": "Test", "last_name": "User", "beneficiary_id": "T001"}
    ],
    "dpi": 300,
    "include_mpi_id": true
  }' | jq -r '.job_id')

echo "Job lancé : $JOB_ID"

# Étape 4 : Polling jusqu'à completion
while true; do
  STATUS=$(curl -s https://api.oomus.health/jobs/$JOB_ID \
    -H "Authorization: Bearer $TOKEN" | jq -r '.status')
  echo "Statut : $STATUS"
  if [ "$STATUS" = "completed" ] || [ "$STATUS" = "failed" ]; then
    break
  fi
  sleep 5
done

# Étape 5 : Télécharger le ZIP
curl -X GET https://api.oomus.health/jobs/$JOB_ID/download/zip \
  -H "Authorization: Bearer $TOKEN" \
  -o "cartes.zip"
```

---

## Prochaines étapes

- [Facturation](billing.md) — Gérer votre quota
- [DHIS2](dhis2.md) — Générer depuis DHIS2
- [Authentification](authentication.md) — Gestion des tokens
