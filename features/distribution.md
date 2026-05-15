# Distribution multicanal

Oomus CampaignID permet de distribuer les cartes de santé numériques via trois canaux complémentaires : **WhatsApp**, **SMS** et **Google Wallet**. La distribution peut être déclenchée depuis l'interface ou via l'API, pour chaque bénéficiaire individuellement ou en masse.

---

## Comparatif des canaux

| Canal | Technologie | Couverture | Format | Offline possible |
|---|---|---|---|---|
| **WhatsApp** | Meta Graph API v25.0 | Afrique + mondial | Image PNG + texte | Non (requiert connexion) |
| **SMS** | Orange SMS API OAuth2 | Afrique de l'Ouest | Lien court + texte | Non |
| **Google Wallet** | Google Wallet API | Android + Web | Pass générique QR | Oui (pass téléchargé) |

---

## WhatsApp

### Technologie

Oomus CampaignID utilise la **Meta Graph API v25.0** pour l'envoi de messages WhatsApp Business. Chaque envoi comprend :
- L'image PNG de la carte générée
- Un message personnalisable (intégrant le prénom du bénéficiaire, le nom du programme et le numéro de carte)
- Un lien vers le portail de vérification (optionnel)

### Prérequis

Pour utiliser la distribution WhatsApp, votre programme doit disposer de :
- Un **compte Meta Business** vérifié
- Un **numéro de téléphone WhatsApp Business** enregistré auprès de Meta
- Une clé API (Meta Graph Token) configurée dans les paramètres d'intégration Oomus

### Fonctionnement

1. Oomus CampaignID récupère l'image PNG de la carte du bénéficiaire
2. Le message est personnalisé (nom, programme, code carte)
3. La requête est envoyée à Meta Graph API v25.0
4. Le statut de livraison est enregistré (envoyé / délivré / lu / échec)
5. Les logs sont disponibles dans l'onglet Distribution

### Exemple d'envoi individuel via API

```bash
curl -X POST https://api.oomus.health/dhis2/send-card \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "enrollment_id": "xyz123",
    "channel": "whatsapp",
    "phone_number": "+221771234567",
    "message": "Bonjour {first_name}, voici votre carte de vaccination. Conservez ce message."
  }'
```

### Logging des envois

Chaque message WhatsApp envoyé est journalisé avec :
- Timestamp d'envoi
- Numéro de téléphone (masqué partiellement dans les logs)
- Statut de livraison (envoyé / délivré / lu / échec)
- Identifiant message Meta
- Identifiant du bénéficiaire (sans PII)

---

## SMS

### Technologie

Oomus CampaignID utilise l'**Orange SMS API OAuth2** pour l'envoi de SMS en Afrique de l'Ouest.

### Couverture géographique

Le service SMS est optimisé pour les pays d'Afrique de l'Ouest couverts par Orange :

| Pays | Code | Couverture |
|---|---|---|
| Sénégal | SN | Orange Sénégal |
| Côte d'Ivoire | CI | Orange CI |
| Mali | ML | Orange Mali |
| Burkina Faso | BF | Orange BF |
| Guinée | GN | Orange Guinée |
| Cameroun | CM | Orange Cameroun |

> Pour les pays hors couverture Orange, contactez l'équipe Oomus pour discuter d'une intégration avec un opérateur local.

### Contenu du SMS

Le SMS contient :
- Un message court personnalisé (60–160 caractères)
- Un lien court vers le portail de vérification en ligne ou le téléchargement de la carte

Exemple de SMS :
```
Bonjour Aminata, votre carte de vaccination EPI est disponible :
https://v.oomus.health/c/DKR-VAC-9XQ7LM2A
```

### Exemple d'envoi SMS en masse (depuis DHIS2)

L'envoi en masse est déclenché depuis l'onglet **Distribution** de la vue DHIS2, en sélectionnant les bénéficiaires et le canal SMS. Les numéros de téléphone sont automatiquement extraits des attributs DHIS2 mappés.

---

## Google Wallet

### Qu'est-ce qu'un pass Google Wallet ?

Un **Google Wallet Generic Pass** est une carte numérique stockée dans l'application Google Wallet sur Android ou accessible via navigateur web. Il peut être utilisé sans connexion Internet une fois téléchargé.

### Contenu du pass

Chaque pass généré par Oomus CampaignID inclut :

| Champ | Valeur |
|---|---|
| **QR code** | Jeton de vérification cryptographique |
| **Nom du bénéficiaire** | Prénom + Nom (depuis les données campagne) |
| **Programme** | Nom du programme de santé |
| **Date d'émission** | Date de génération de la carte |
| **Date d'expiration** | Si configurée (optionnel) |
| **Logo** | Logo du programme (optionnel) |
| **Couleur** | Couleur du programme |

### Émission individuelle

```bash
curl -X GET "https://api.oomus.health/dhis2/google-wallet/{enrollment_id}" \
  -H "Authorization: Bearer <VOTRE_TOKEN>"
```

**Réponse :**
```json
{
  "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGci...",
  "jwt_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "enrollment_id": "xyz123",
  "valid_until": "2027-05-15T00:00:00Z"
}
```

Le bénéficiaire clique sur `wallet_url` pour ajouter le pass à son Google Wallet.

### Émission en bulk (jusqu'à 100 cartes)

```bash
curl -X POST "https://api.oomus.health/dhis2/google-wallet/bulk" \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "enrollment_ids": ["xyz123", "abc456", "def789"],
    "programme_uid": "P3jJH5Tu5VC"
  }'
```

**Réponse :**
```json
{
  "batch_id": "batch_01HXYZ",
  "total": 3,
  "wallet_links": [
    {
      "enrollment_id": "xyz123",
      "wallet_url": "https://pay.google.com/gp/v/save/eyJhbGci..."
    },
    ...
  ]
}
```

> L'émission en bulk est limitée à **100 passes par requête**. Pour les volumes plus importants, effectuez plusieurs requêtes ou utilisez l'export CSV depuis l'interface.

---

## Logs de distribution

Chaque envoi (quel que soit le canal) est journalisé dans le tableau de bord :

- **Accès** : Dashboard > Distribution > Logs
- **Filtres** : par canal, date, statut, campagne, bénéficiaire (par ID, sans afficher de PII)
- **Statuts** : `sent`, `delivered`, `failed`, `pending`
- **Export** : CSV disponible pour audit

---

## Prochaines étapes

- [Guide WhatsApp & SMS](../integrations/messaging.md) — Configuration des intégrations
- [Guide Google Wallet](../integrations/google-wallet.md) — Prérequis et configuration
- [Intégration DHIS2](dhis2-integration.md) — Distribuer depuis DHIS2
