# Intégration Google Wallet

Oomus CampaignID permet d'émettre des passes Google Wallet pour les bénéficiaires disposant d'un smartphone Android ou d'un navigateur compatible. Le pass fonctionne hors ligne une fois téléchargé.

---

## Prérequis

Pour utiliser l'intégration Google Wallet, vous devez disposer de :

| Prérequis | Description |
|---|---|
| Compte Google Cloud | Projet Google Cloud actif |
| Service Account | Compte de service avec rôle Wallet Object Creator |
| Google Wallet API | API Google Wallet activée dans votre projet Google Cloud |
| Issuer ID | Identifiant d'émetteur Google Wallet (obtenu lors de l'inscription au programme) |
| Clé privée (JSON) | Fichier de clé du compte de service (format JSON) |

### Inscription au programme Google Wallet

1. Accédez à [Google Pay & Wallet Console](https://pay.google.com/business/console/)
2. Inscrivez votre organisation comme émetteur
3. Choisissez le type **Generic Pass**
4. Soumettez votre demande (délai d'approbation : 1–5 jours ouvrables)
5. Une fois approuvé, récupérez votre **Issuer ID**

### Configuration dans Oomus CampaignID

1. Dans l'interface Oomus, naviguez vers **Paramètres > Intégrations > Google Wallet**
2. Uploadez votre fichier de clé JSON du compte de service
3. Renseignez votre **Issuer ID**
4. Configurez le nom de la classe de pass (exemple : `vaccination_card_sn`)
5. Testez avec une émission individuelle

---

## Type de pass — Generic Pass

Oomus CampaignID génère des **passes génériques** (Generic Pass), le type le plus flexible et adapté aux cartes de santé.

### Contenu du pass

Chaque pass Google Wallet généré par Oomus CampaignID inclut :

| Champ | Valeur | Emplacement dans le pass |
|---|---|---|
| **QR code** | Jeton de vérification cryptographique | Centre du pass |
| **Nom du bénéficiaire** | Prénom + Nom | En-tête |
| **Programme** | Nom du programme de santé | Sous-titre |
| **Date d'émission** | Date de génération de la carte | Corps |
| **Date d'expiration** | Si configurée (optionnel) | Corps |
| **Logo** | Logo du programme | Coin supérieur gauche |
| **Couleur** | Couleur primaire du programme | Fond du pass |
| **Numéro de carte** | Code unique de la carte | Bas du pass |

### Ce que le pass ne contient PAS

- Données médicales ou de santé
- Identifiant MPI en clair (le QR utilise un jeton opaque)
- Adresse ou données de localisation
- Numéro de téléphone

---

## Émission individuelle

### Via l'interface

1. Depuis une campagne ou un enrollment DHIS2, localisez le bénéficiaire
2. Cliquez sur l'icône **Google Wallet** à côté du nom
3. Copiez ou partagez le lien généré avec le bénéficiaire

### Via l'API

```bash
curl -X GET "https://api.oomus.health/dhis2/google-wallet/xyz123abc" \
  -H "Authorization: Bearer <VOTRE_TOKEN>"
```

**Réponse :**

```json
{
  "enrollment_id": "xyz123abc",
  "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "jwt_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "valid_until": "2027-05-15T00:00:00Z"
}
```

Le bénéficiaire clique sur `wallet_url` pour ajouter le pass à son application Google Wallet.

---

## Émission en bulk (jusqu'à 100 cartes)

L'émission en bulk permet de générer jusqu'à 100 passes en une seule requête, idéale pour les distributions de groupe (clinic day, campagne de vaccination de zone).

```bash
curl -X POST "https://api.oomus.health/dhis2/google-wallet/bulk" \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "enrollment_ids": [
      "xyz123abc",
      "def456ghi",
      "jkl789mno"
    ],
    "programme_uid": "P3jJH5Tu5VC"
  }'
```

**Réponse :**

```json
{
  "batch_id": "batch_01HXYZ",
  "total": 3,
  "processed": 3,
  "failed": 0,
  "wallet_links": [
    {
      "enrollment_id": "xyz123abc",
      "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGci..."
    },
    {
      "enrollment_id": "def456ghi",
      "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGci..."
    },
    {
      "enrollment_id": "jkl789mno",
      "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGci..."
    }
  ]
}
```

Pour les volumes supérieurs à 100, découpez en plusieurs requêtes ou utilisez l'export CSV depuis l'interface (qui génère tous les liens en arrière-plan).

---

## Expérience bénéficiaire

### Sur Android

1. Le bénéficiaire reçoit le lien par WhatsApp ou SMS
2. Il clique sur le lien → son application Google Wallet s'ouvre
3. Il appuie sur **"Ajouter"** pour sauvegarder le pass
4. La carte est disponible offline dans son application Google Wallet

### Sur navigateur web

1. Le bénéficiaire ouvre le lien dans son navigateur
2. Il se connecte à son compte Google si nécessaire
3. Le pass est sauvegardé dans Google Wallet Web (`wallet.google.com`)

---

## Mise à jour et révocation des passes

### Mise à jour d'un pass

Si les données d'une carte changent (ex. correction du nom), le pass peut être mis à jour :

- Via l'interface : Campagne > Bénéficiaire > **"Mettre à jour le pass Wallet"**
- La mise à jour est synchronisée automatiquement dans l'application Google Wallet du bénéficiaire (requiert une connexion Internet)

### Révocation d'un pass

En cas de perte de carte ou d'invalidation :

- Via l'interface : Campagne > Bénéficiaire > **"Révoquer le pass Wallet"**
- Le pass est marqué comme invalide et n'est plus scannable
- La révocation est synchronisée dans l'application Google Wallet du bénéficiaire

---

## Disponibilité

| Plateforme | Disponibilité |
|---|---|
| Android (app Google Wallet) | Supporté |
| Google Chrome (Web) | Supporté |
| iOS (via navigateur) | Supporté via wallet.google.com |
| Application native iOS Google Wallet | Non disponible (Apple Wallet uniquement sur iOS) |

---

## Prochaines étapes

- [Distribution multicanal](../features/distribution.md) — Tous les canaux disponibles
- [Guide DHIS2](dhis2.md) — Distribution depuis DHIS2
- [Référence API DHIS2](../api-reference/dhis2.md) — Endpoints Google Wallet
