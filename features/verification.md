# Portail de vérification

Le portail de vérification d'Oomus CampaignID permet à tout agent de santé de vérifier l'authenticité d'une carte, **sans connexion Internet**, dans n'importe quel contexte de terrain.

---

## Principe fondamental : Offline-first

La conception du portail est radicalement différente des systèmes de vérification traditionnels qui nécessitent un appel serveur à chaque scan. Oomus CampaignID utilise une approche **offline-first** :

1. Le portail est un **artefact statique** (HTML + JavaScript + JSON) généré à la fin de chaque job de génération
2. Il est téléchargé **une seule fois** par l'agent de santé (ou distribué sur clé USB)
3. Toutes les vérifications s'effectuent **localement dans le navigateur**, sans aucune donnée transmise au serveur
4. Le portail fonctionne sur n'importe quel appareil doté d'un navigateur (smartphone, tablette, ordinateur)

---

## Ce qui est vérifié (et ce qui ne l'est pas)

### Ce que le portail vérifie

| Information | Méthode |
|---|---|
| Authenticité du code carte | Lookup SHA-256 dans le registre local |
| Appartenance à la campagne | Vérification du préfixe et du registre |
| Statut de la carte | `valid` / `revoked` / `not_found` |
| Date d'émission | Contenu du registre |

### Ce que le portail NE transmet pas

- Aucune donnée personnelle du bénéficiaire n'est transmise au serveur
- Aucun log de vérification n'est envoyé (confidentialité de l'agent)
- Aucune connexion réseau n'est établie lors de la vérification

---

## Modes de vérification

### 1. Saisie manuelle du code

L'agent entre manuellement le code imprimé sur la carte (format : `DKR-VAC-9XQ7LM2A`). Le portail effectue un lookup immédiat dans le registre local.

### 2. Scanner QR code

L'agent pointe la caméra de son smartphone sur le QR code de la carte. Le portail utilise l'API WebRTC du navigateur pour décoder le QR et effectue la vérification cryptographique.

**Flux de vérification QR :**
```
Scan QR code
     │
     ▼
Décodage du jeton opaque
     │
     ▼
Lookup dans registry.json (SHA-256)
     │
     ├─ Trouvé + valide → ✓ Carte authentique
     │
     ├─ Trouvé + révoquée → ✗ Carte révoquée
     │
     └─ Non trouvé → ✗ Carte inconnue / falsifiée
```

### 3. Vérification en masse — CSV

Pour les opérations de terrain impliquant de nombreuses cartes à vérifier (recensement, distribution), l'agent peut importer un fichier CSV contenant les codes cartes. Le portail traite toute la liste en quelques secondes et génère un rapport de vérification.

Format CSV accepté :
```csv
code
DKR-VAC-9XQ7LM2A
DKR-VAC-3KPQ8NX1
DKR-VAC-7FZM2TYB
```

---

## Structure du portail (artefacts)

À la fin de chaque job de génération réussi, le portail contient :

```
verification-portal/
├── index.html          # Interface de vérification (React compilé)
├── registry.json       # Registre des codes (hashé SHA-256, non PII)
├── manifest.json       # Métadonnées de la campagne
└── assets/
    ├── app.js          # Logique WebCrypto de vérification
    └── styles.css      # Interface utilisateur
```

### Structure de registry.json

```json
{
  "campaign": "DKR-VAC",
  "generated_at": "2026-05-15T09:10:48Z",
  "total_cards": 1000,
  "entries": {
    "a3f8c2e1d4b7...": {
      "status": "valid",
      "issued_at": "2026-05-15"
    },
    "9b2e4a7f1c8d...": {
      "status": "valid",
      "issued_at": "2026-05-15"
    }
  }
}
```

> Les clés du registre sont des **hachés SHA-256** des jetons QR — elles ne contiennent aucune information personnelle sur le bénéficiaire.

---

## Vérification cryptographique (WebCrypto)

La vérification s'effectue via l'**API WebCrypto** du navigateur — disponible sur tout navigateur moderne sans plugin ni application :

```javascript
// Extrait illustratif — exécuté dans le navigateur
const tokenHash = await crypto.subtle.digest(
  "SHA-256",
  new TextEncoder().encode(scannedToken)
);
const hashHex = toHexString(tokenHash);
const result = registry.entries[hashHex];
// result.status === "valid" → carte authentique
```

**Avantages de WebCrypto :**
- Natif dans tous les navigateurs modernes (Chrome, Firefox, Safari, Edge)
- Aucune donnée ne quitte l'appareil
- Résistant aux manipulations (exécuté dans le contexte navigateur sécurisé)

---

## Support multilingue

Le portail de vérification est disponible en trois langues :

| Langue | Code | Disponibilité |
|---|---|---|
| Français | `fr` | Toutes les campagnes |
| English | `en` | Toutes les campagnes |
| Wolof | `wo` | Campagnes en Afrique de l'Ouest |

La langue est sélectionnable dans l'interface du portail. La langue par défaut correspond à celle configurée lors de la création de la campagne.

---

## Endpoint MPI public (sans PII)

Pour les vérifications en ligne, un endpoint public permet de vérifier un jeton QR sans exposer de données personnelles :

```bash
# Vérification publique — aucun token d'auth requis
curl -X GET https://api.oomus.health/mpi/verify/TOKEN_QR_ICI
```

**Réponse :**
```json
{
  "status": "valid",
  "programme_count": 2,
  "verified_at": "2026-05-15T10:30:00Z"
}
```

La réponse confirme uniquement :
- Validité du jeton (`valid` / `invalid` / `revoked`)
- Nombre de programmes associés à ce bénéficiaire (sans nommer les programmes)
- Timestamp de vérification

**Aucune donnée personnelle (nom, date de naissance, MPI ID) n'est jamais renvoyée par cet endpoint.**

---

## Modèle de sécurité

### Chaîne cryptographique

Chaque portail de vérification est lié à la chaîne d'audit de la génération :

```
Génération carte → Hash SHA-256 du code → Entrée registry.json
                 → Hash SHA-256 du registry → Stocké dans audit_log
```

### Evidence anti-falsification

- Le `registry.json` est signé à sa génération — toute modification post-génération invalide la vérification
- La chaîne de hachage SHA-256 lie le registre à la chaîne d'audit immuable du serveur
- Toute tentative de modification du registre est détectable

### Distribution sécurisée du portail

Il est recommandé de distribuer les portails de vérification via :
- Téléchargement direct depuis l'interface Oomus (lien signé, durée limitée)
- Serveur web interne HTTPS de l'organisation
- Clé USB pour les zones sans accès Internet

---

## Prochaines étapes

- [Garanties cryptographiques](../security/cryptographic-guarantees.md) — Détails techniques de la sécurité
- [Vérification hors ligne](../security/offline-verification.md) — Guide de déploiement terrain
- [Identité MPI souveraine](mpi-sovereign-identity.md) — MPI et jetons QR
