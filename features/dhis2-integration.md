# Intégration DHIS2 Tracker

Oomus CampaignID s'intègre nativement avec **DHIS2 Tracker** pour synchroniser automatiquement les enrollments de bénéficiaires et générer des cartes directement depuis les données de votre système national de santé.

---

## Configuration de la connexion

### Prérequis

- Une instance DHIS2 (version 2.36 ou supérieure) accessible en réseau
- Un compte utilisateur DHIS2 avec droits en lecture sur les programmes Tracker cibles
- Les droits `programme_admin` sur Oomus CampaignID

### Paramètres de connexion

| Paramètre | Description | Exemple |
|---|---|---|
| **URL DHIS2** | URL complète de votre instance | `https://dhis2.sante.gov.sn` |
| **Nom d'utilisateur** | Compte DHIS2 dédié (lecture seule recommandé) | `oomus_sync_user` |
| **Mot de passe** | Mot de passe du compte DHIS2 | (stocké chiffré) |

> Oomus CampaignID n'écrit jamais dans DHIS2. Le compte utilisé doit avoir des droits en **lecture seule** (`dhis2_safe` read-only guard). Cela protège l'intégrité de vos données DHIS2.

### Test de connexion

Après avoir renseigné les paramètres, cliquez sur **"Tester la connexion"** pour valider :

```bash
curl -X POST https://api.oomus.health/dhis2/test-connection \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://dhis2.sante.gov.sn",
    "username": "oomus_sync_user",
    "password": "VotreMotDePasseDHIS2"
  }'
```

**Réponse (succès) :**
```json
{
  "status": "connected",
  "dhis2_version": "2.41.2",
  "server_date": "2026-05-15T09:00:00",
  "programmes_available": 7
}
```

---

## Listage des programmes DHIS2

Une fois connecté, récupérez la liste des programmes Tracker disponibles :

```bash
curl -X GET https://api.oomus.health/dhis2/programmes \
  -H "Authorization: Bearer <VOTRE_TOKEN>"
```

---

## Synchronisation des enrollments

### Modes de synchronisation

| Mode | Description |
|---|---|
| **Manuel** | Déclenchement à la demande depuis l'interface |
| **Automatique (cron)** | Synchronisation périodique programmée |
| **Webhook** | Déclenchement sur événement DHIS2 (si configuré) |

### Configuration du cron de synchronisation

Intervalles disponibles : **10 / 15 / 30 / 60 / 180 minutes**

```bash
curl -X POST https://api.oomus.health/dhis2/sync-schedule \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "programme_uid": "P3jJH5Tu5VC",
    "interval_minutes": 30,
    "enabled": true
  }'
```

### Processus de synchronisation

```
Déclenchement (cron / manuel)
        │
        ▼
Récupération des enrollments DHIS2
(depuis la dernière sync ou période configurable)
        │
        ▼
Normalisation des attributs
        │
        ▼
Résolution MPI (moteur probabiliste)
        │
        ▼
Garde données sensibles (IA — 7 catégories)
        │
        ▼
Mise à jour du registre Oomus
        │
        ▼
Déclenchement génération cartes (si configuré)
```

---

## Mapping des attributs DHIS2

Le mapping des attributs permet de relier les champs DHIS2 aux champs des cartes Oomus.

### Méthodes de mapping

Les attributs DHIS2 peuvent être mappés de deux manières :

**Par code d'attribut** (recommandé) :
```json
{
  "dhis2_attribute_code": "MMD_PER_NAM",
  "card_field": "first_name"
}
```

**Par nom d'affichage (displayName)** :
```json
{
  "dhis2_attribute_display_name": "First name",
  "card_field": "first_name"
}
```

### Exemple de configuration de mapping

```bash
curl -X POST https://api.oomus.health/dhis2/assign-codes \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "programme_uid": "P3jJH5Tu5VC",
    "mappings": [
      {
        "dhis2_attribute_code": "MMD_PER_NAM",
        "card_field": "first_name"
      },
      {
        "dhis2_attribute_code": "MMD_PER_LST",
        "card_field": "last_name"
      },
      {
        "dhis2_attribute_code": "MMD_PER_DOB",
        "card_field": "date_of_birth"
      },
      {
        "dhis2_attribute_code": "MMD_PER_SEX",
        "card_field": "gender"
      }
    ]
  }'
```

---

## Résolution MPI lors de la synchronisation

À chaque enrollment synchronisé, le moteur MPI est automatiquement invoqué :

1. **Extraction** des attributs d'identité (nom, prénom, DDN, sexe, commune)
2. **Scoring** probabiliste contre le registre MPI existant
3. **Liaison** automatique si score ≥ 0.95 (même MPI ID pour tous les programmes)
4. **Queue de révision** si score 0.75–0.94
5. **Création** d'un nouvel identifiant MPI si aucun match

Le résultat : chaque enrollment DHIS2 est lié à l'identité MPI souveraine correspondante.

---

## Génération de cartes depuis DHIS2

### 10 modèles DHIS2 disponibles

Oomus CampaignID propose **10 modèles** de cartes haute résolution pour les données DHIS2 :

| Modèle | Palette | Description |
|---|---|---|
| `vital` | Bleu médical / cyan | Carte de santé essentielle, champs minimaux |
| `emerald` | Vert forêt / émeraude | Carte intermédiaire avec champs programme |
| `pulse` | Dark navy / cyan néon | Carte complète avec QR, MPI et données étendues |
| `mothercare` | Rose / teal | Suivi prénatal et vaccination mère-enfant |
| `shield` | Slate / or | Protection sociale et droits sociaux |
| `nomad` | Noir / orange | Élevage et populations pastorales, grand QR |
| `aero` | Navy / ambre | Vaccination renforcée et campagnes terrain |
| `horizon` | Indigo profond / bleu clair | Carte premium gradient horizontal |
| `aurora` | Indigo / violet / rose | Programmes spécialisés, encoches perforées |
| **`sovereign`** | **Navy sombre / or** | **DHIS2 Digital Pass — design boarding-pass identique au prévisualiseur interactif** |

#### Modèle `sovereign` — DHIS2 Digital Pass

Le modèle `sovereign` reproduit exactement le composant `Dhis2BoardingCard` du prévisualiseur interactif. Il a été entièrement redessiné en v5.9 :

- **Format** : 1011×375 px @ 300 DPI (ratio 480:178, boarding-pass horizontal)
- **En-tête** : jusqu'à 3 logos + "DHIS2 DIGITAL PASS" (tout en majuscules) + "Identity Pass · eHealth Platform" — badge "Sovereign Id" supprimé
- **Identifiant MPI** : label "MPI-ID" + icône empreinte digitale (8 crêtes) + valeur en gros caractères gras blancs
- **Grille attributs** : 2 colonnes × 2 lignes (jusqu'à 4 attributs) sans chevauchement
- **Zone QR** : fond noir profond, cadre à 4 crochets dorés, "SCAN TO VERIFY", identifiant TEI tronqué
- **Séparateur** : tirets dorés avec losanges ornementaux haut/bas
- **Pied** : nom du programme (majuscules) à gauche + date d'enrollment à droite
- **Configurable** via `SovereignCardConfig` : champs `accent_hex`, `bg_hex`, `bg_deep_hex`, `font_scale`, `max_attrs`
- Fonds navy/or avec ellipses topographiques subtiles

### Prévisualisation en temps réel — Onglet Card Design

Avant de lancer la génération, l'onglet **Card Design** affiche un aperçu de la carte en **temps réel** :

- L'aperçu se rafraîchit automatiquement 450 ms après chaque modification : template, couleur, logos, attributs sélectionnés, sous-titre, taille de titre
- Fonctionne **hors connexion DHIS2** : des données fictives représentatives sont utilisées si aucun enrollment n'est disponible
- Rendu à **300 DPI** via le même moteur serveur que la génération en masse
- Indicateur de statut : cyan animé (chargement) → vert (prêt) → gris (aucun aperçu)
- Cliquez **"Rafraîchir"** pour forcer une mise à jour manuelle

### Lancer la génération depuis DHIS2

```bash
curl -X POST https://api.oomus.health/dhis2/generate-cards \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "programme_uid": "P3jJH5Tu5VC",
    "template": "pulse",
    "dpi": 300,
    "include_mpi_id": true,
    "enrollment_filter": {
      "enrollment_date_from": "2026-01-01",
      "org_unit": "DiszpKrYNg8"
    }
  }'
```

### Aperçu PNG avec données réelles

Avant de lancer une génération en masse, prévisualisez une carte avec les données réelles d'un enrollment :

```bash
curl -X GET "https://api.oomus.health/dhis2/preview-card-png?enrollment_id=xyz123&template=pulse" \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  --output apercu_carte.png
```

---

## Garde des données sensibles (IA)

Avant tout export vers les pipelines ML/analytics, le moteur de garde analyse automatiquement les attributs DHIS2 et **bloque les données sensibles** selon 7 catégories :

| Catégorie | Exemples d'attributs bloqués |
|---|---|
| **HIV/SIDA** | Statut sérologique HIV, charge virale, CD4 |
| **IST** | Syphilis, gonorrhée, résultats IST |
| **Tuberculose** | Statut TB, traitement DOT, BK positif |
| **Santé mentale** | Diagnostics psychiatriques, traitements psy |
| **Sérologique** | Résultats de tests sérologiques sensibles |
| **Biométrique** | Empreintes digitales, iris, ADN |
| **Financier** | Revenus, statut économique, paiements |

Ces attributs sont automatiquement exclus des exports analytiques, des logs de débogage et des traitements IA — ils restent présents dans le registre chiffré.

---

## Distribution depuis DHIS2

Après génération, les cartes peuvent être distribuées directement depuis l'onglet **Distribution** de la vue DHIS2 :

- **WhatsApp** : envoi de l'image de carte au numéro de téléphone DHIS2
- **SMS** : envoi d'un lien de téléchargement au numéro enregistré
- **Google Wallet** : émission d'un pass Google Wallet individuel ou en bulk

---

## Versions DHIS2 supportées

| Version DHIS2 | Support |
|---|---|
| 2.36 | Supporté (baseline) |
| 2.37 | Supporté |
| 2.38 | Supporté |
| 2.39 | Supporté |
| 2.40 | Supporté |
| 2.41 | Supporté (recommandé) |
| 2.42+ | Compatibilité testée régulièrement |

> Pour les versions antérieures à 2.36, contactez l'équipe Oomus pour une évaluation de compatibilité.

---

## Prochaines étapes

- [Guide d'intégration DHIS2](../integrations/dhis2.md) — Guide pas-à-pas complet
- [Référence API — DHIS2](../api-reference/dhis2.md)
- [Identité MPI souveraine](mpi-sovereign-identity.md) — Résolution MPI
- [Distribution multicanal](distribution.md) — Envoyer les cartes
