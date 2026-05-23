# Authentification

Oomus CampaignID utilise une authentification **JWT (JSON Web Token)** basée sur HS256. Toutes les requêtes API (sauf les endpoints publics documentés) nécessitent un token d'accès valide dans l'en-tête `Authorization`.

**Base URL :** `https://api.oomus.health`

---

## Durée de vie des tokens

| Token | Durée de validité |
|---|---|
| **Access token** | 30 minutes |
| **Refresh token** | 30 jours |

L'access token doit être inclus dans chaque requête API. Lorsqu'il expire, utilisez le refresh token pour obtenir un nouvel access token sans re-saisir vos identifiants.

---

## Endpoints

### POST /auth/register

Crée un nouveau compte programme.

**Corps de la requête :**

```json
{
  "email": "ceo@oomus.org",
  "password": "VotreMotDePasse!Secure",
  "full_name": "Programme National de Vaccination",
  "organization": "Ministère de la Santé du Sénégal"
}
```

**Réponse 201 Created :**

```json
{
  "id": "usr_01HXYZ123ABC",
  "email": "ceo@oomus.org",
  "full_name": "Programme National de Vaccination",
  "organization": "Ministère de la Santé du Sénégal",
  "role": "programme_admin",
  "plan": "starter",
  "is_active": true,
  "created_at": "2026-05-15T09:00:00Z"
}
```

**Erreurs possibles :**

| Code | Description |
|---|---|
| `400` | Corps de requête invalide (champs manquants, format incorrect) |
| `409` | Un compte avec cette adresse e-mail existe déjà |
| `422` | Erreur de validation (mot de passe trop faible, e-mail invalide) |

---

### POST /auth/login

Authentifie un utilisateur et retourne les tokens JWT.

**Corps de la requête :**

```json
{
  "email": "ceo@oomus.org",
  "password": "VotreMotDePasse!Secure"
}
```

**Réponse 200 OK :**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c3JfMDFIWFlaMTIzQUJDIiwiZXhwIjoxNzQ3MzA2NDAwfQ...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c3JfMDFIWFlaMTIzQUJDIiwidHlwZSI6InJlZnJlc2gifQ...",
  "token_type": "bearer",
  "expires_in": 1800
}
```

**Erreurs possibles :**

| Code | Description |
|---|---|
| `401` | Identifiants incorrects |
| `403` | Compte désactivé |
| `422` | Format de la requête invalide |

**Exemple cURL :**

```bash
curl -X POST https://api.oomus.health/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "ceo@oomus.org",
    "password": "VotreMotDePasse!Secure"
  }'
```

---

### POST /auth/refresh

Obtient un nouvel access token à partir d'un refresh token valide.

**Corps de la requête :**

```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Réponse 200 OK :**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c3JfMDFIWFlaMTIzQUJDIiwiZXhwIjoxNzQ3MzA2NDAwfQ...",
  "token_type": "bearer",
  "expires_in": 1800
}
```

**Erreurs possibles :**

| Code | Description |
|---|---|
| `401` | Refresh token invalide ou expiré |

---

### GET /auth/me

Retourne le profil de l'utilisateur authentifié.

**En-tête requis :**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Réponse 200 OK :**

```json
{
  "id": "usr_01HXYZ123ABC",
  "email": "ceo@oomus.org",
  "name": "Programme National de Vaccination",
  "country": "Sénégal",
  "phone": "+221XXXXXXXXX",
  "role": "programme_admin",
  "plan": "starter",
  "status": "active",
  "balance_fcfa": 125000,
  "cards_generated": 4800,
  "is_admin": false,
  "permissions": [],
  "two_factor_enabled": false,
  "logo_url": "data:image/png;base64,iVBORw0K...",
  "created_at": "2026-05-15T09:00:00Z"
}
```

> **`logo_url`** — Data URI base64 du logo programme (`data:{type};base64,{b64}`), stocké directement en base. `null` si aucun logo uploadé.
> **`two_factor_enabled`** — `true` si la 2FA TOTP est activée sur ce compte.

---

### PATCH /auth/me

Met à jour le profil de l'utilisateur authentifié.

**Corps de la requête (champs optionnels) :**

```json
{
  "full_name": "Nouveau Nom",
  "organization": "Nouvelle Organisation"
}
```

**Réponse 200 OK :** profil mis à jour (même schéma que GET /auth/me)

---

### POST /auth/change-password

Modifie le mot de passe de l'utilisateur authentifié.

**Corps de la requête :**

```json
{
  "current_password": "AncienMotDePasse!",
  "new_password": "NouveauMotDePasse!Secure"
}
```

**Réponse 204 No Content** (succès silencieux)

**Erreurs possibles :**

| Code | Description |
|---|---|
| `400` | Mot de passe actuel incorrect ou nouveau mot de passe identique à l'ancien |

---

### POST /auth/logo

Upload le logo du programme (PNG / JPEG / WEBP, max 512 KB). Le logo est encodé en base64 et stocké en base de données dans `logo_url`. Il est automatiquement renvoyé dans `ProgrammeOut` (GET /auth/me) et affiché dans la sidebar et le profil.

**Content-Type :** `multipart/form-data`

| Champ | Type | Description |
|---|---|---|
| `file` | `File` | Image PNG, JPEG ou WEBP (≤ 512 KB) |

**Réponse 200 OK :** profil complet mis à jour (même schéma que GET /auth/me, avec `logo_url` renseigné)

**Erreurs possibles :**

| Code | Description |
|---|---|
| `400` | Format de fichier non supporté (seuls PNG/JPEG/WEBP acceptés) |
| `413` | Image trop volumineuse (max 512 KB) |

---

## Authentification à deux facteurs (2FA TOTP)

La 2FA est basée sur le standard TOTP (RFC 6238 — Time-based One-Time Password), compatible avec Google Authenticator, Authy et tout client TOTP standard.

### POST /auth/2fa/setup

Génère un nouveau secret TOTP et retourne l'URI de provisioning pour le scan QR.

**Réponse 200 OK :**

```json
{
  "secret": "BASE32SECRETHERE",
  "provisioning_uri": "otpauth://totp/OOMUS%20DPI:contact%40programme.sn?secret=BASE32SECRETHERE&issuer=OOMUS%20DPI"
}
```

Le secret retourné doit être transmis à `POST /auth/2fa/verify-setup` pour activer la 2FA. Il n'est pas encore enregistré en base à ce stade.

---

### POST /auth/2fa/verify-setup

Vérifie le code TOTP saisi et active la 2FA sur le compte.

**Corps de la requête :**

```json
{
  "secret": "BASE32SECRETHERE",
  "code": "123456"
}
```

**Réponse 200 OK :** profil mis à jour avec `two_factor_enabled: true`

**Erreurs possibles :**

| Code | Description |
|---|---|
| `400` | Code TOTP invalide (code expiré, mauvais secret) |
| `501` | Module `pyotp` non installé (backend) |

---

### DELETE /auth/2fa/disable

Désactive la 2FA sur le compte. Le `totp_secret` est effacé et `two_factor_enabled` passe à `false`.

**Réponse 200 OK :** profil mis à jour avec `two_factor_enabled: false`

---

### POST /auth/2fa/login

Deuxième étape d'authentification lorsque la 2FA est activée. À appeler après une connexion `POST /auth/login` réussie, en fournissant le code TOTP courant.

**Corps de la requête :**

```json
{
  "email": "ceo@oomus.org",
  "code": "123456"
}
```

**Réponse 200 OK :** tokens JWT (même schéma que `POST /auth/login`)

**Erreurs possibles :**

| Code | Description |
|---|---|
| `400` | 2FA non configurée sur ce compte |
| `401` | Code TOTP invalide |

---

## Utilisation du token dans les requêtes

Incluez l'access token dans l'en-tête `Authorization` de toutes vos requêtes API :

```bash
curl -X GET https://api.oomus.health/campaigns/ \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## Gestion de l'expiration

### Flux recommandé

```
1. Login → stocker access_token + refresh_token
2. Utiliser access_token pour les requêtes
3. Si réponse 401 (token expiré) :
   → Appeler POST /auth/refresh avec le refresh_token
   → Stocker le nouvel access_token
   → Retry la requête originale
4. Si refresh_token expiré (après 30 jours) :
   → Rediriger l'utilisateur vers la page de login
```

### Implémentation JavaScript (exemple)

```javascript
async function apiRequest(url, options = {}) {
  const token = localStorage.getItem("access_token");
  const response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${token}`,
    },
  });

  if (response.status === 401) {
    // Token expiré — tentative de rafraîchissement
    const refreshed = await refreshAccessToken();
    if (refreshed) {
      return apiRequest(url, options); // Retry
    } else {
      redirectToLogin();
    }
  }

  return response;
}
```

---

## Prochaines étapes

- [Campagnes & Jobs](campaigns-and-jobs.md)
- [Facturation](billing.md)
- [MPI](mpi.md)
