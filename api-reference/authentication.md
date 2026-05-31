# Authentification

Oomus CampaignID utilise une authentification **JWT (JSON Web Token)**. Toutes les requêtes API (sauf les endpoints publics documentés) nécessitent un token d'accès valide dans l'en-tête `Authorization`.

**Base URL :** `https://api.oomus.health`

---

## Tokens d'accès

L'access token doit être inclus dans chaque requête API. Lorsqu'il expire, utilisez le refresh token pour obtenir un nouvel access token sans re-saisir vos identifiants.

---

## Endpoints

### POST /auth/register

Crée un nouveau compte programme.

**Corps de la requête :**

```json
{
  "email": "contact@programme.sn",
  "password": "VotreMotDePasse!Secure",
  "name": "Programme National de Vaccination",
  "country": "Sénégal"
}
```

**Réponse 201 Created :**

```json
{
  "id": "usr_01HXYZ123ABC",
  "email": "contact@programme.sn",
  "name": "Programme National de Vaccination",
  "role": "programme_admin",
  "plan": "starter",
  "status": "active",
  "created_at": "2026-05-15T09:00:00Z"
}
```

**Erreurs possibles :**

| Code | Description |
|---|---|
| `400` | Email déjà utilisé |
| `422` | Validation échouée (mot de passe trop faible, email invalide) |

---

### POST /auth/login

Authentifie un utilisateur et retourne les tokens JWT.

**Corps de la requête :**

```json
{
  "email": "contact@programme.sn",
  "password": "VotreMotDePasse!Secure"
}
```

**Réponse 200 OK (sans 2FA) :**

```json
{
  "access_token": "<access_token>",
  "refresh_token": "<refresh_token>",
  "token_type": "bearer",
  "requires_2fa": false,
  "programme": { "...": "..." }
}
```

**Réponse 200 OK (avec 2FA activée) :**

```json
{
  "access_token": "",
  "refresh_token": "",
  "token_type": "bearer",
  "requires_2fa": true,
  "totp_token": "<totp_pending_token>"
}
```

> Si `requires_2fa: true`, appelez `POST /auth/2fa/login` avec le `totp_token` reçu et le code TOTP.

**Erreurs possibles :**

| Code | Description |
|---|---|
| `401` | Identifiants incorrects |
| `403` | Compte suspendu |
| `429` | Trop de tentatives — réessayez plus tard |

---

### POST /auth/refresh

Obtient un nouvel access token à partir d'un refresh token valide.

**Corps de la requête :**

```json
{
  "refresh_token": "<refresh_token>"
}
```

**Réponse 200 OK :**

```json
{
  "access_token": "<nouvel_access_token>",
  "refresh_token": "<nouveau_refresh_token>",
  "token_type": "bearer"
}
```

**Erreurs possibles :**

| Code | Description |
|---|---|
| `401` | Refresh token invalide, expiré ou révoqué |

---

### POST /auth/logout

Révoque le token d'accès courant (déconnexion de la session active).

**Réponse 204 No Content**

---

### POST /auth/logout-all

Révoque toutes les sessions actives du compte (tous les appareils).

**Réponse 204 No Content**

---

### GET /auth/me

Retourne le profil de l'utilisateur authentifié.

**Réponse 200 OK :**

```json
{
  "id": "usr_01HXYZ123ABC",
  "email": "contact@programme.sn",
  "name": "Programme National de Vaccination",
  "country": "Sénégal",
  "role": "programme_admin",
  "plan": "starter",
  "status": "active",
  "balance_fcfa": 125000,
  "cards_generated": 4800,
  "is_admin": false,
  "permissions": [],
  "two_factor_enabled": false,
  "logo_url": null,
  "created_at": "2026-05-15T09:00:00Z"
}
```

---

### PATCH /auth/me

Met à jour le profil de l'utilisateur authentifié (nom, pays, téléphone).

---

### POST /auth/change-password

Modifie le mot de passe. Invalide toutes les sessions actives.

**Corps de la requête :**

```json
{
  "current_password": "AncienMotDePasse!",
  "new_password": "NouveauMotDePasse!Secure"
}
```

**Réponse 204 No Content**

---

### POST /auth/logo

Upload le logo du programme (PNG / JPEG / WEBP, max 512 KB).

**Content-Type :** `multipart/form-data`

| Champ | Type | Description |
|---|---|---|
| `file` | `File` | Image PNG, JPEG ou WEBP (≤ 512 KB) |

---

## Authentification à deux facteurs (2FA TOTP)

La 2FA est basée sur le standard TOTP (RFC 6238), compatible avec Google Authenticator, Authy et tout client TOTP standard.

### Flux complet

```
1. POST /auth/login      → { requires_2fa: true, totp_token: "..." }
2. POST /auth/2fa/login  → { access_token, refresh_token } (tokens définitifs)
```

### POST /auth/2fa/setup

Génère un secret TOTP et retourne l'URI de provisioning.

**Réponse 200 OK :**

```json
{
  "secret": "BASE32SECRETHERE",
  "provisioning_uri": "otpauth://totp/OOMUS%20DPI:contact%40programme.sn?secret=BASE32SECRETHERE&issuer=OOMUS%20DPI"
}
```

### POST /auth/2fa/verify-setup

Active la 2FA après confirmation du code.

```json
{ "secret": "BASE32SECRETHERE", "code": "123456" }
```

### DELETE /auth/2fa/disable

Désactive la 2FA. Requiert la confirmation du mot de passe actuel.

```json
{ "current_password": "VotreMotDePasse!" }
```

### POST /auth/2fa/login

Complète l'authentification 2FA avec le token intermédiaire et le code TOTP.

```json
{
  "totp_token": "<totp_pending_token>",
  "code": "123456"
}
```

**Réponse 200 OK :** tokens JWT définitifs (même schéma que `POST /auth/login`)

---

## Utilisation du token dans les requêtes

```bash
curl -X GET https://api.oomus.health/campaigns \
  -H "Authorization: Bearer <access_token>"
```

---

## Gestion de l'expiration

### Flux recommandé

```
1. Login  → stocker access_token + refresh_token (côté client)
2. Utiliser access_token pour les requêtes
3. Si réponse 401 :
   → POST /auth/refresh avec le refresh_token
   → Stocker le nouvel access_token
   → Réessayer la requête
4. Si refresh_token invalide :
   → Rediriger vers la page de connexion
```

---

## Prochaines étapes

- [Campagnes & Jobs](campaigns-and-jobs.md)
- [Facturation](billing.md)
- [MPI](mpi.md)
