# Messagerie — WhatsApp & SMS

Oomus CampaignID intègre deux canaux de messagerie complémentaires pour la distribution des cartes de santé numériques : **WhatsApp** (Meta Graph API v25.0) et **SMS** (Orange SMS API OAuth2).

---

## WhatsApp

### Technologie

Oomus CampaignID utilise la **Meta Graph API version 25.0** (WhatsApp Business Platform) pour l'envoi de messages WhatsApp. Il s'agit de l'API officielle de Meta pour les communications Business-to-Consumer à grande échelle.

### Prérequis

Pour configurer l'intégration WhatsApp, vous avez besoin de :

| Prérequis | Description | Où l'obtenir |
|---|---|---|
| Meta Business Account | Compte Business Manager vérifié | [business.facebook.com](https://business.facebook.com) |
| Application Meta | Application dans votre Business Manager | Meta Developer Portal |
| Numéro WhatsApp Business | Numéro dédié à WhatsApp Business (non personnel) | Fournisseur de numéros ou opérateur local |
| Token d'accès permanent | Meta System User Access Token | Meta Developer > System Users |
| Phone Number ID | Identifiant du numéro WhatsApp dans Meta | Meta WhatsApp Manager |

> Votre numéro WhatsApp Business ne peut pas être utilisé sur l'application WhatsApp personnelle en même temps.

### Vérification du compte Meta Business

Meta exige une vérification du compte Business avant d'autoriser l'envoi en volume :

1. Accédez à **Meta Business Manager > Paramètres > Vérification de l'entreprise**
2. Soumettez les documents de votre organisation (acte de constitution, documents officiels)
3. Délai de vérification : 1–5 jours ouvrables
4. Une fois vérifié, votre limite d'envoi passe de 250 à 1 000 puis 100 000+ conversations/jour

### Configuration dans Oomus CampaignID

1. Dans l'interface Oomus, naviguez vers **Paramètres > Intégrations > WhatsApp**
2. Renseignez :
   - **Token d'accès Meta** : votre System User Access Token
   - **Phone Number ID** : l'identifiant de votre numéro WhatsApp
   - **Numéro d'expéditeur** : votre numéro WhatsApp Business (format E.164)
3. Cliquez sur **"Tester l'envoi"** pour envoyer un message de test

### Format des messages WhatsApp

Chaque message WhatsApp envoyé par Oomus comprend :

- **Image** : PNG de la carte du bénéficiaire (haute résolution)
- **Texte** : message personnalisé avec variables dynamiques

Variables dynamiques disponibles dans le message :

| Variable | Valeur |
|---|---|
| `{first_name}` | Prénom du bénéficiaire |
| `{last_name}` | Nom de famille |
| `{card_code}` | Code unique de la carte |
| `{programme_name}` | Nom du programme |
| `{issue_date}` | Date d'émission |
| `{verification_url}` | URL du portail de vérification |

**Exemple de message :**
```
Bonjour {first_name} {last_name},

Voici votre carte {programme_name}. Conservez ce message précieusement.

Code carte : {card_code}
Vérification : {verification_url}
```

### Conformité Meta et templates de messages

Pour les messages initiés par votre programme (hors-session de 24h), Meta exige l'utilisation de **Message Templates** pré-approuvés.

- Catégorie recommandée : **Utility** (notifications transactionnelles)
- Langue(s) requise(s) : Français, Anglais (selon vos marchés)
- Délai d'approbation : 24–72 heures

L'équipe Oomus fournit des templates de messages pré-rédigés pour les cas d'usage santé courants (vaccination, nutrition, assurance maladie).

---

## SMS

### Technologie

Oomus CampaignID utilise l'**Orange SMS API** avec authentification **OAuth2** pour l'envoi de SMS en Afrique de l'Ouest.

### Couverture géographique Orange

| Pays | Code | Opérateur |
|---|---|---|
| Sénégal | SN | Orange Sénégal |
| Côte d'Ivoire | CI | Orange CI |
| Mali | ML | Orange Mali |
| Burkina Faso | BF | Orange BF |
| Guinée | GN | Orange Guinée |
| Cameroun | CM | Orange Cameroun |
| Madagascar | MG | Orange Madagascar |
| Tunisie | TN | Orange Tunisie |
| Égypte | EG | Orange Egypt |

> Pour les pays hors couverture Orange, contactez l'équipe Oomus pour discuter d'une intégration avec un opérateur local partenaire.

### Prérequis SMS

| Prérequis | Description |
|---|---|
| Compte développeur Orange | Compte sur [developer.orange.com](https://developer.orange.com) |
| Application créée | Application avec accès à "SMS" dans le portail développeur Orange |
| Client ID + Client Secret | Credentials OAuth2 de votre application |
| Sender Name | Nom d'expéditeur alphanumériques (max 11 caractères, ex: `SANTE-SN`) |

### Configuration dans Oomus CampaignID

1. Dans l'interface Oomus, naviguez vers **Paramètres > Intégrations > SMS**
2. Renseignez :
   - **Client ID** : votre identifiant client Orange API
   - **Client Secret** : votre secret client Orange API
   - **Sender Name** : nom d'expéditeur (ex: `SANTE-SN`, `VACC-CI`)
   - **Pays par défaut** : code pays pour les numéros sans indicatif
3. Cliquez sur **"Tester l'envoi"** pour envoyer un SMS de test

### Format des messages SMS

Les SMS sont limités à **160 caractères** (SMS standard). Au-delà, le message est segmenté en SMS concaténés (facturation par segment).

**Exemple de SMS de distribution :**
```
[PNV-SN] Bonjour Aminata, votre carte vaccination
est prête. Téléchargez-la : https://v.oomus.health/c/DKR-VAC-9XQ7LM2A
```

---

## Envoi depuis l'interface DHIS2

Le moyen le plus courant d'envoyer des cartes est depuis l'onglet **Distribution** de la vue DHIS2 :

### Envoi individuel

1. Sélectionnez un enrollment dans la vue DHIS2
2. Cliquez sur l'onglet **"Distribution"**
3. Le numéro de téléphone est automatiquement extrait de l'attribut DHIS2 mappé
4. Choisissez le canal : WhatsApp ou SMS
5. Éditez le message si nécessaire
6. Cliquez sur **"Envoyer"**

### Envoi en masse

1. Depuis la liste des enrollments DHIS2, cochez plusieurs bénéficiaires
2. Cliquez sur **"Distribution groupée"**
3. Choisissez le canal
4. Confirmez l'envoi

---

## Logging et suivi

Chaque message envoyé est journalisé avec :

| Information | Description |
|---|---|
| Timestamp | Date et heure d'envoi (UTC) |
| Canal | WhatsApp ou SMS |
| Statut | `pending` / `sent` / `delivered` / `read` / `failed` |
| Identifiant message | ID externe (Meta Message ID ou Orange Message ID) |
| Numéro destinataire | Masqué partiellement (ex: `+22177***4567`) |
| ID campagne / enrollment | Référence interne (sans PII) |

Les logs sont consultables dans :
- **Dashboard > Distribution > Logs**
- **Campagne > Onglet Distribution > Historique**

---

## Bonnes pratiques

### WhatsApp
- Obtenez le **consentement explicite** des bénéficiaires avant d'envoyer des messages WhatsApp (GDPR / réglementation locale)
- Utilisez les templates approuvés par Meta pour les envois hors session 24h
- Envoyez les cartes dans les 24h suivant la génération pour une expérience optimale

### SMS
- Vérifiez que les numéros de téléphone dans DHIS2 sont au format **E.164** (`+221771234567`)
- Évitez les envois entre 21h et 7h (heure locale) pour respecter les réglementations locales
- Pour les campagnes massives (>10 000 SMS), planifiez les envois en dehors des heures de pointe

---

## Prochaines étapes

- [Distribution multicanal](../features/distribution.md) — Vue d'ensemble de la distribution
- [Guide DHIS2](dhis2.md) — Distribution depuis DHIS2
- [Google Wallet](google-wallet.md) — Canal Google Wallet
