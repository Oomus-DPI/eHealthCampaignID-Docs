# Démarrage rapide

Ce guide vous permet de créer votre première campagne et de générer vos premières cartes en moins de 10 minutes.

**URL de l'application** : [https://app.oomus.health](https://app.oomus.health)  
**URL API (production)** : `https://api.oomus.health`  
**URL API (développement)** : `http://localhost:8000`

---

## Étape 1 — Créer un compte programme

### Via l'interface web

1. Accédez à [https://app.oomus.health/register](https://app.oomus.health/register)
2. Renseignez le nom de votre organisation, votre e-mail et un mot de passe sécurisé
3. Choisissez votre plan lors de l'inscription (vous pouvez changer ultérieurement)
4. Confirmez votre e-mail, puis connectez-vous

### Via l'API

```bash
# Créer un compte
curl -X POST https://api.oomus.health/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "contact@programme-sante.sn",
    "password": "VotreMotDePasse!",
    "full_name": "Programme National de Vaccination",
    "organization": "Ministère de la Santé du Sénégal"
  }'
```

**Réponse :**
```json
{
  "id": "usr_01HXYZ123ABC",
  "email": "contact@programme-sante.sn",
  "full_name": "Programme National de Vaccination",
  "organization": "Ministère de la Santé du Sénégal",
  "role": "programme_admin",
  "plan": "starter",
  "created_at": "2026-05-15T09:00:00Z"
}
```

---

## Étape 2 — S'authentifier

```bash
# Obtenir un token d'accès JWT
curl -X POST https://api.oomus.health/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "contact@programme-sante.sn",
    "password": "VotreMotDePasse!"
  }'
```

**Réponse :**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 1800
}
```

> Conservez votre `access_token`. Il expire après **30 minutes**. Utilisez le `refresh_token` (valable **30 jours**) pour en obtenir un nouveau sans re-saisir vos identifiants.

Dans tous les exemples suivants, remplacez `<VOTRE_TOKEN>` par la valeur de votre `access_token`.

---

## Étape 3 — Configurer votre première campagne

### Via l'interface web

1. Depuis le Dashboard, cliquez sur **"Nouvelle campagne"**
2. Renseignez le formulaire :
   - **Nom** : ex. `Vaccination EPI - Dakar 2026`
   - **Type** : ex. `vaccination`
   - **Préfixe** : ex. `DKR-VAC` (3–8 caractères, majuscules)
   - **Langue** : Français / English / Wolof
   - **Modèle de carte** : sélectionnez parmi les 11 modèles disponibles (ex. `vaccination`)
3. Cliquez sur **"Créer la campagne"**

### Via l'API

```bash
curl -X POST https://api.oomus.health/campaigns/ \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Vaccination EPI - Dakar 2026",
    "campaign_type": "vaccination",
    "prefix": "DKR-VAC",
    "language": "fr",
    "template_id": "vaccination"
  }'
```

**Réponse :**
```json
{
  "id": "camp_01HXYZ456DEF",
  "name": "Vaccination EPI - Dakar 2026",
  "status": "draft",
  "prefix": "DKR-VAC",
  "template_id": "vaccination",
  "created_at": "2026-05-15T09:05:00Z"
}
```

---

## Étape 4 — Concevoir votre carte (Card Studio)

### Via l'interface web

1. Depuis la campagne, cliquez sur l'onglet **"Card Studio"**
2. L'éditeur visuel s'ouvre avec votre modèle sélectionné
3. Personnalisez :
   - **Logos** : uploadez jusqu'à 3 logos (programme, partenaire, gouvernement)
   - **Couleurs** : choisissez la couleur primaire et secondaire de votre programme
   - **Champs dynamiques** : configurez les champs recto (nom, date, numéro) et verso (informations complémentaires)
   - **QR code** : activez le QR et choisissez son style
4. Cliquez sur **"Aperçu PNG"** pour visualiser le résultat avant génération
5. Sauvegardez le design

---

## Étape 5 — Générer les cartes

### Lancer un job de génération

```bash
curl -X POST https://api.oomus.health/campaigns/camp_01HXYZ456DEF/jobs \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "beneficiaries": [
      {
        "first_name": "Aminata",
        "last_name": "Diallo",
        "date_of_birth": "2020-03-15",
        "beneficiary_id": "BEN-001"
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
  }'
```

**Réponse (job créé) :**
```json
{
  "job_id": "job_01HXYZ789GHI",
  "campaign_id": "camp_01HXYZ456DEF",
  "status": "pending",
  "total_cards": 2,
  "created_at": "2026-05-15T09:10:00Z"
}
```

### Suivre la progression

La génération est **asynchrone**. Suivez la progression via polling ou WebSocket :

```bash
# Polling (vérification du statut)
curl -X GET https://api.oomus.health/jobs/job_01HXYZ789GHI \
  -H "Authorization: Bearer <VOTRE_TOKEN>"
```

**Réponse (en cours) :**
```json
{
  "job_id": "job_01HXYZ789GHI",
  "status": "generating",
  "progress": 65,
  "cards_generated": 1,
  "total_cards": 2,
  "started_at": "2026-05-15T09:10:05Z"
}
```

**Réponse (terminé) :**
```json
{
  "job_id": "job_01HXYZ789GHI",
  "status": "completed",
  "progress": 100,
  "cards_generated": 2,
  "total_cards": 2,
  "artifacts": {
    "pdf_url": "https://api.oomus.health/jobs/job_01HXYZ789GHI/download/pdf",
    "zip_url": "https://api.oomus.health/jobs/job_01HXYZ789GHI/download/zip",
    "verification_portal_url": "https://api.oomus.health/jobs/job_01HXYZ789GHI/download/portal"
  },
  "completed_at": "2026-05-15T09:10:48Z"
}
```

### Connexion WebSocket (temps réel)

```javascript
// Connexion WebSocket pour progression en temps réel
const ws = new WebSocket(
  "wss://api.oomus.health/ws/jobs/job_01HXYZ789GHI?token=<VOTRE_TOKEN>"
);

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(`Progression : ${data.progress}% — ${data.status}`);
};
```

---

## Étape 6 — Distribuer les cartes

Une fois la génération terminée, distribuez les cartes depuis l'interface ou via l'API :

- **WhatsApp** : depuis l'onglet "Distribution" de votre campagne, envoyez les cartes avec un message personnalisé
- **SMS** : envoyez un lien de téléchargement par SMS
- **Google Wallet** : émission de passes individuels ou en bulk
- **Téléchargement** : téléchargez le PDF et le ZIP pour impression ou archivage

---

## Résumé des étapes

| Étape | Action | Interface |
|---|---|---|
| 1 | Créer un compte programme | Web / API |
| 2 | S'authentifier, obtenir un JWT | API |
| 3 | Créer une campagne | Web / API |
| 4 | Concevoir la carte (Card Studio) | Web uniquement |
| 5 | Lancer la génération | Web / API |
| 6 | Distribuer (WhatsApp, SMS, Google Wallet) | Web / API |

---

## Prochaines étapes

- [Card Studio](../features/card-studio.md) — Maîtriser l'éditeur visuel
- [Identité MPI souveraine](../features/mpi-sovereign-identity.md) — Activer la déduplication
- [Intégration DHIS2](../features/dhis2-integration.md) — Synchroniser depuis DHIS2 Tracker
- [Plans & Tarification](plans-and-pricing.md) — Adapter votre quota à votre programme
