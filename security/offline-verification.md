# Vérification hors ligne

La vérification hors ligne est l'une des capacités les plus distinctives d'Oomus CampaignID. Elle permet à un agent de terrain de vérifier l'authenticité de milliers de cartes sans aucune connexion Internet.

---

## Comment ça fonctionne

### Génération du portail

À la fin de chaque job de génération réussi, Oomus CampaignID produit automatiquement un **portail de vérification statique** :

```
verification-portal/
├── index.html          # Interface de vérification (SPA compilée)
├── registry.json       # Registre des codes (hachés SHA-256)
├── manifest.json       # Métadonnées de la campagne
└── assets/
    ├── app.js          # Logique WebCrypto
    └── styles.css
```

### Distribution du portail

Le portail peut être distribué par plusieurs moyens selon la connectivité disponible :

| Méthode | Connectivité requise | Cas d'usage |
|---|---|---|
| Téléchargement depuis l'interface Oomus | Requiert connexion au moment du téléchargement | Responsables de campagne |
| Clé USB / carte SD | Aucune | Zones sans accès Internet |
| Serveur web interne | Réseau local uniquement | Structures de santé avec intranet |
| CD-ROM / archive hors ligne | Aucune | Archivage, zones les plus reculées |

### Vérification côté client

Une fois le portail téléchargé, **toutes les vérifications s'effectuent localement dans le navigateur** :

```
Agent scanne/saisit le code
           │
           ▼
JavaScript (app.js) + WebCrypto API
           │
           ▼
Lookup SHA-256 dans registry.json local
           │
           ▼
Résultat affiché instantanément
(aucune requête réseau effectuée)
```

---

## Modes de vérification

### 1. Saisie manuelle du code

L'agent saisit le code imprimé sur la carte (ex. : `DKR-VAC-9XQ7LM2A`).

**Interface** :
- Champ de saisie avec validation du format
- Vérification immédiate après saisie (< 50 ms)
- Résultat visuel clair (vert / rouge / orange)

**Résultats possibles** :

| Statut | Affichage | Signification |
|---|---|---|
| ✅ Valide | Fond vert, "Carte authentique" | La carte est dans le registre et active |
| ❌ Révoquée | Fond rouge, "Carte révoquée" | La carte a été invalidée |
| ⚠️ Inconnue | Fond orange, "Carte non reconnue" | Le code n'est pas dans ce registre (falsification possible ou mauvais portail) |

### 2. Scanner QR code (caméra)

L'agent pointe la caméra de son smartphone sur le QR code.

**Compatibilité** :
- Android : Chrome, Firefox, Samsung Internet
- iOS : Safari, Chrome (via API WebRTC)
- Tablette : tous navigateurs modernes

**Processus** :
1. Clic sur **"Scanner un QR code"**
2. Autorisation caméra demandée (une seule fois)
3. Cadrer le QR code → vérification automatique
4. Résultat affiché en 1–2 secondes

Le scanner décode le jeton QR opaque et effectue le lookup SHA-256 local. Aucune connexion réseau n'est établie.

### 3. Vérification en masse — Import CSV

Pour les opérations impliquant de nombreuses cartes (distribution groupée, contrôle d'une liste) :

1. Clic sur **"Vérification en masse"**
2. Importer un fichier CSV contenant les codes cartes :

```csv
code
DKR-VAC-9XQ7LM2A
DKR-VAC-3KPQ8NX1
DKR-VAC-7FZM2TYB
DKR-VAC-INVALID01
```

3. Le portail traite toute la liste immédiatement (traitement local)
4. Résumé affiché : X valides / Y révoquées / Z inconnues
5. Export du rapport CSV de résultats

---

## Support multilingue

Le portail de vérification est disponible en trois langues :

| Langue | Code | Niveau de support |
|---|---|---|
| **Français** | `fr` | Interface complète + messages d'erreur |
| **English** | `en` | Interface complète + messages d'erreur |
| **Wolof** | `wo` | Interface complète + messages d'erreur |

La langue est sélectionnable par l'agent via un sélecteur en haut du portail. La langue par défaut correspond à celle de la campagne.

---

## Ce qui est vérifié

| Information | Vérifié ? | Source |
|---|---|---|
| Authenticité du code | Oui | Lookup SHA-256 dans registry.json |
| Appartenance à la campagne | Oui | Préfixe du code + registre |
| Statut (valide / révoqué) | Oui | Champ `status` dans le registre |
| Date d'émission | Oui | Champ `issued_at` dans le registre |
| Nom du bénéficiaire | Non | Non inclus dans le registre (protection vie privée) |
| Données médicales | Non | Jamais incluses |
| MPI ID | Non | Non inclus dans le registre |

---

## Ce qui n'est PAS transmis

Lors d'une vérification hors ligne :

- **Aucune donnée personnelle** n'est transmise à un serveur
- **Aucun log de scan** n'est envoyé (l'agent de terrain peut vérifier de manière totalement anonyme)
- **Aucune connexion réseau** n'est établie
- **Aucun cookie** de tracking n'est placé

---

## Garanties de sécurité

### Anti-falsification du registre

Le `registry.json` est lié par SHA-256 à la chaîne d'audit de génération stockée sur le serveur Oomus. Toute tentative de modification du registre après sa génération est détectable lors d'une vérification en ligne ultérieure.

### Pas de PII dans le registre

Le fichier `registry.json` ne contient que des hachés de jetons QR et des statuts. Il est techniquement impossible de retrouver l'identité d'un bénéficiaire à partir de ce fichier.

### Intégrité du portail

L'application JavaScript du portail est compilée et minifiée. Elle inclut des vérifications d'intégrité pour détecter toute modification du code côté client.

---

## Durée de validité du portail

Chaque portail de vérification est lié à un job de génération spécifique. Le portail contient uniquement les cartes générées dans ce job.

**Recommandation** : Pour une campagne avec plusieurs jobs de génération, téléchargez et distribuez un portail consolidé depuis l'interface de la campagne (onglet **"Portail de vérification"**).

---

## Prochaines étapes

- [Portail de vérification — Fonctionnalités](../features/verification.md)
- [Garanties cryptographiques](cryptographic-guarantees.md)
- [Protection des données](data-protection.md)
