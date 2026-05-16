# Authentification

Oomus CampaignID utilise une authentification **JWT (JSON Web Token)** basée sur HS256. Toutes les requêtes API (sauf les endpoints publics documentés) nécessitent un token d'accès valide dans l'en-tête `Authorization`.

**Base URL :** `https://api.oomus.health`

---

## Durée de vie des tokens

| Token | Durée de validité |
| --- | --- |
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
  "email": "contact@programme-sante.sn",
  "password": "VotreMotDePasse!Secure",
  "full_name": "Programme National de Vaccination",
  "organization": "Ministère de la Santé du Sénégal"
}
```

**Réponse 201 Created :**

```json
{
  "id": "usr_01HXYZ123ABC",
  "email": "contact@programme-sante.sn",
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
| --- | --- |
| `400` | Corps de requête invalide (champs manquants, format incorrect) |
| `409` | Un compte avec cette adresse e-mail existe déjà |
| `422` | Erreur de validation (mot de passe trop faible, e-mail invalide) |

---

### POST /auth/login

Authentifie un utilisateur et retourne les tokens JWT.

**Corps de la requête :**

```json
{
  "email": "contact@programme-sante.sn",
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
| --- | --- |
| `401` | Identifiants incorrects |
| `403` | Compte désactivé |
| `422` | Format de la requête invalide |

**Exemple cURL :**

```bash
curl -X POST https://api.oomus.health/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "contact@programme-sante.sn",
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
| --- | --- |
| `401` | Refresh token invalide ou expiré |

---

### GET /auth/me

Retourne le profil de l'utilisateur authentifié.

**En-tête requis :**

```text
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Réponse 200 OK :**

```json
{
  "id": "usr_01HXYZ123ABC",
  "email": "contact@programme-sante.sn",
  "full_name": "Programme National de Vaccination",
  "organization": "Ministère de la Santé du Sénégal",
  "role": "programme_admin",
  "plan": "starter",
  "is_active": true,
  "created_at": "2026-05-15T09:00:00Z",
  "last_login": "2026-05-15T09:00:00Z"
}
```

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

**Réponse 200 OK :**

```json
{
  "message": "Mot de passe modifié avec succès."
}
```

**Erreurs possibles :**

| Code | Description |
| --- | --- |
| `401` | Mot de passe actuel incorrect |
| `422` | Nouveau mot de passe ne respecte pas les critères de sécurité |

---

### POST /auth/logo

Upload du logo de votre programme. Le logo est stocké de façon isolée par programme et affiché dans la sidebar, l'avatar et les aperçus de cartes.

**En-têtes requis :**

```text
Authorization: Bearer <VOTRE_TOKEN>
Content-Type: multipart/form-data
```

**Corps (multipart) :**

| Champ | Type | Description |
| --- | --- | --- |
| `file` | File | Image PNG, JPG, WebP ou SVG — max 2 Mo |

**Exemple cURL :**

```bash
curl -X POST https://api.oomus.health/auth/logo \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -F "file=@logo_programme.png"
```

**Réponse 200 OK :** profil du programme mis à jour avec `logo_url` renseigné.

**Erreurs possibles :**

| Code | Description |
| --- | --- |
| `400` | Format de fichier non supporté ou taille > 2 Mo |
| `401` | Token manquant ou invalide |

---

### GET /auth/logo/{programme_id}

Sert le logo d'un programme. Endpoint public — aucune authentification requise.

**Paramètre URL :**

| Paramètre | Description |
| --- | --- |
| `programme_id` | UUID du programme |

**Réponse 200 OK :** fichier image avec le `Content-Type` approprié (`image/png`, `image/jpeg`, etc.).

**Réponse 404 :** aucun logo uploadé pour ce programme.

---

### POST /auth/2fa/setup

Initialise l'authentification à deux facteurs TOTP pour le compte courant. Retourne un QR code à scanner avec Google Authenticator, Authy ou équivalent.

**En-tête requis :** `Authorization: Bearer <VOTRE_TOKEN>`

**Réponse 200 OK :**

```json
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qr_code_url": "otpauth://totp/CampaignID%3Acontact%40programme.sn?secret=JBSWY3DPEHPK3PXP&issuer=OomusCampaignID"
}
```

Affichez `qr_code_url` sous forme de QR code dans votre application, ou saisissez `secret` manuellement dans votre gestionnaire TOTP.

---

### POST /auth/2fa/verify

Confirme l'activation du 2FA en soumettant le premier code TOTP généré par l'application. Le 2FA est activé uniquement après cette vérification.

**Corps de la requête :**

```json
{
  "code": "123456"
}
```

**Réponse 200 OK :**

```json
{
  "message": "Authentification à deux facteurs activée.",
  "two_factor_enabled": true
}
```

**Erreurs possibles :**

| Code | Description |
| --- | --- |
| `400` | Code TOTP invalide ou expiré |
| `401` | Token manquant ou invalide |

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

```text
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
