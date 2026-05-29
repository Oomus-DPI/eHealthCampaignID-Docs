# Garanties cryptographiques

Oomus CampaignID intègre des garanties cryptographiques à chaque étape du cycle de vie des cartes — de la génération à la vérification hors ligne. Cette page décrit ces garanties d'un point de vue fonctionnel.

---

## Unicité — Identifiants Base36

Chaque carte générée par Oomus CampaignID reçoit un **identifiant unique** produit par un générateur de nombres aléatoires cryptographiquement sécurisé (**CSPRNG**).

### Propriétés

- **Format** : Base36 (chiffres 0–9 + lettres A–Z)
- **Longueur** : 8 caractères
- **Espace de combinaisons** : environ **2,8 milliards** de valeurs possibles
- **Collision** : pratiquement impossible dans l'espace d'une campagne

### Garanties

| Propriété | Garantie |
|---|---|
| **Non-prédictibilité** | Impossible de deviner le prochain identifiant généré |
| **Unicité** | Chaque identifiant est unique dans le contexte d'une campagne |
| **Lisibilité humaine** | Format alphanumérique court, facilement saisie par un agent de terrain |

L'identifiant MPI suit la même logique Base36 pour son segment unique (`9XQ7LM2A`).

---

## Intégrité — Chiffrement AES-256-GCM

Les données de vérification sensibles sont protégées par un **chiffrement authentifié AES-256-GCM** (Advanced Encryption Standard, mode Galois/Counter Mode).

### Pourquoi AES-256-GCM ?

L'AES-256-GCM offre deux propriétés simultanées :
- **Confidentialité** : les données chiffrées sont illisibles sans la clé
- **Authenticité** : tout chiffre altéré est détecté immédiatement (tag d'authentification GCM)

Cela signifie qu'une tentative de falsification des données chiffrées est détectable, pas seulement illisible.

---

## Auditabilité — Chaîne de hachage SHA-256

Oomus CampaignID maintient une **chaîne de hachage SHA-256** qui lie chaque entrée d'audit à la précédente, formant une chaîne immuable.

### Comment fonctionne la chaîne

```
Bloc 0 (Genesis)
   │ SHA-256(données_0 + hash_genesis)
   ▼
Bloc 1 : hash_1 = SHA-256(données_1 + hash_0)
   │ SHA-256(données_1 + hash_0)
   ▼
Bloc 2 : hash_2 = SHA-256(données_2 + hash_1)
   │
   ▼
   ...
```

### Garanties

| Propriété | Garantie |
|---|---|
| **Immutabilité** | Modifier une entrée passée invalide tous les blocs suivants |
| **Détection de falsification** | Toute manipulation de la chaîne est détectable |
| **Non-répudiation** | Chaque action est irréfutablement liée à son acteur et son horodatage |

La chaîne d'audit est également utilisée pour lier le `registry.json` du portail de vérification à l'audit de génération.

---

## Vérification hors ligne — WebCrypto API

La vérification des cartes en mode hors ligne utilise l'**API WebCrypto** native des navigateurs modernes (standard W3C).

### Avantages de WebCrypto

| Propriété | Bénéfice |
|---|---|
| **Natif navigateur** | Aucun plugin, aucune application à installer |
| **Exécution locale** | Aucune donnée ne quitte l'appareil lors de la vérification |
| **Standard W3C** | Disponible sur Chrome, Firefox, Safari, Edge (tous appareils) |
| **Performances** | Vérification instantanée (< 50 ms) |

### Processus de vérification

```
Code carte ou jeton QR scanné
          │
          ▼
Dérivation SHA-256 dans le navigateur (WebCrypto)
          │
          ▼
Lookup dans registry.json local
          │
          ├─ Hash trouvé, statut = "valid" → Carte authentique
          ├─ Hash trouvé, statut = "revoked" → Carte révoquée
          └─ Hash non trouvé → Carte inconnue / potentiellement falsifiée
```

---

## Jeton QR opaque — Non-réversibilité

Le QR code imprimé sur chaque carte contient un **jeton opaque non-réversible**, dérivé par hachage SHA-256.

### Propriétés du jeton

| Propriété | Description |
|---|---|
| **Opacité** | Le jeton ne contient aucune information déchiffrable sans la clé serveur |
| **Non-réversibilité** | Impossible de retrouver le MPI ID ou le nom du bénéficiaire à partir du jeton |
| **Unicité** | Un jeton par carte — deux cartes du même bénéficiaire ont des jetons différents |
| **Vérifiabilité** | Le serveur peut confirmer la validité sans révéler le MPI |

### Pourquoi c'est important

Un acteur malveillant qui scannerait des QR codes de masse dans une clinique :
- **Avec un QR classique** : pourrait récolter des identités de santé, lier des noms à des programmes (HIV, etc.)
- **Avec le jeton opaque Oomus** : ne récolte que des chaînes de caractères aléatoires sans valeur exploitable

---

## Signatures de factures — HMAC-SHA256

Chaque facture émise par la plateforme porte une **signature HMAC-SHA256** (keyed hash) stockée dans `Invoice.signature_hash`.

### Algorithme et format

| Élément | Valeur |
| --- | --- |
| **Algorithme** | HMAC-SHA256 (Python `hmac` standard library) |
| **Clé** | `SECRET_KEY` de l'environnement (≥ 64 chars en production) |
| **Payload signé** | `invoice_id:invoice_number:total_fcfa:billing_period` |
| **Résistance** | Impossible de forger une signature sans connaître `SECRET_KEY` |

### Vérification

Toute facture dont la signature ne correspond pas au recalcul HMAC est considérée comme **altérée**. Le endpoint `GET /billing/v2/invoices/{id}/verify` permet de vérifier une facture à tout moment.

> **Note importante** : les factures générées avant la v5.14.1 utilisaient un hachage SHA-256 sans clé (non-HMAC). Ces factures peuvent être identifiées par leur format de `signature_hash` (longueur identique mais non vérifiable via HMAC). Il est recommandé de les régénérer ou de les annoter comme `legacy_signature` dans vos audits.

---

## Recommandations pour la production

Bien qu'Oomus CampaignID intègre des garanties cryptographiques robustes, les déploiements en production bénéficient des mesures complémentaires suivantes :

| Mesure | Description |
|---|---|
| **SSL/TLS** | Certificat TLS valide sur toutes les URLs publiques (`https://`) |
| **WAF** | Web Application Firewall pour protéger l'API contre les attaques courantes |
| **Rotation des clés** | Politique de rotation régulière des clés de signature JWT |
| **Pare-feu réseau** | Restriction des accès à l'infrastructure aux plages IP autorisées |
| **Monitoring** | Surveillance des tentatives d'accès anormales |

---

## Prochaines étapes

- [Vérification hors ligne](offline-verification.md) — Portail de vérification terrain
- [Vue d'ensemble sécurité](overview.md)
- [Protection des données](data-protection.md)
