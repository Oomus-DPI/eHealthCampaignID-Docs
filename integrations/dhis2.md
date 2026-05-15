# Guide d'intégration DHIS2

Ce guide vous accompagne pas à pas dans la configuration complète de l'intégration DHIS2 Tracker avec Oomus CampaignID.

---

## Prérequis

Avant de commencer, assurez-vous de disposer de :

| Prérequis | Description |
|---|---|
| Instance DHIS2 | Version 2.36 ou supérieure, accessible depuis Internet |
| Compte DHIS2 dédié | Droits de lecture (`Metadata: can view` + `Tracker: export tracked entities`) |
| Droits Oomus | Rôle `programme_admin` sur votre compte Oomus CampaignID |
| Plan Oomus | Regional Ops ou supérieur pour l'intégration DHIS2 complète |

### Création d'un compte DHIS2 dédié (recommandé)

Il est vivement recommandé de créer un **compte DHIS2 dédié** pour Oomus, avec des droits en lecture seule :

1. Dans DHIS2 : Maintenance > Utilisateurs > Ajouter un utilisateur
2. Nommez-le `oomus_sync` ou similaire
3. Assignez le rôle `Metadata viewer` + `Tracker program read`
4. Notez le nom d'utilisateur et le mot de passe

Oomus CampaignID ne modifie **jamais** les données DHIS2. Le compte en lecture seule garantit l'intégrité de vos données.

---

## Versions DHIS2 supportées

| Version | Support | Notes |
|---|---|---|
| 2.36 | Supporté (baseline) | API Tracker v1 |
| 2.37 | Supporté | |
| 2.38 | Supporté | |
| 2.39 | Supporté | |
| 2.40 | Supporté | API Tracker v2 disponible |
| 2.41 | Supporté (recommandé) | Meilleures performances |
| 2.42+ | Compatibilité vérifiée régulièrement | Contactez le support si problème |

---

## Étape 1 — Configurer la connexion

### Via l'interface Oomus CampaignID

1. Depuis le Dashboard, naviguez vers **Intégrations > DHIS2**
2. Cliquez sur **"Ajouter une connexion DHIS2"**
3. Renseignez les champs :
   - **URL** : `https://dhis2.sante.gov.sn` (sans slash final)
   - **Nom d'utilisateur** : votre compte DHIS2 dédié
   - **Mot de passe** : mot de passe du compte
4. Cliquez sur **"Tester la connexion"**
5. Si le test réussit, cliquez sur **"Enregistrer"**

### Via l'API

```bash
# Configurer la connexion
curl -X POST https://api.oomus.health/dhis2/config \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://dhis2.sante.gov.sn",
    "username": "oomus_sync",
    "password": "MotDePasseDHIS2"
  }'

# Tester la connexion
curl -X POST https://api.oomus.health/dhis2/test-connection \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://dhis2.sante.gov.sn",
    "username": "oomus_sync",
    "password": "MotDePasseDHIS2"
  }'
```

---

## Étape 2 — Sélectionner un programme DHIS2

1. Dans l'interface Oomus, cliquez sur **"Sélectionner un programme DHIS2"**
2. La liste des programmes Tracker accessibles s'affiche automatiquement
3. Sélectionnez le programme à intégrer
4. Cliquez sur **"Prévisualiser les enrollments"** pour vérifier les données disponibles

---

## Étape 3 — Mapping des attributs

Le mapping des attributs est l'étape la plus importante. Elle détermine quelles données DHIS2 apparaîtront sur les cartes générées.

### Comment identifier les codes d'attributs DHIS2

Dans votre interface DHIS2 :
1. Allez dans **Maintenance > Programmes > [Votre Programme]**
2. Sélectionnez l'onglet **Attributs de l'entité suivie (TEA)**
3. Pour chaque attribut, notez le **Code** (colonne "Code")

Exemple de codes courants (varient selon votre instance) :

| Code DHIS2 | Signification |
|---|---|
| `MMD_PER_NAM` | Prénom (First name) |
| `MMD_PER_LST` | Nom de famille (Last name) |
| `MMD_PER_DOB` | Date de naissance |
| `MMD_PER_SEX` | Sexe |
| `MMD_PER_ADR` | Adresse |
| `PHONE_NUMBER` | Numéro de téléphone |

### Champs de carte Oomus disponibles pour le mapping

| Champ carte Oomus | Description |
|---|---|
| `first_name` | Prénom du bénéficiaire |
| `last_name` | Nom de famille |
| `date_of_birth` | Date de naissance |
| `gender` | Sexe (M/F/U) |
| `beneficiary_id` | Identifiant local du bénéficiaire |
| `phone_number` | Numéro de téléphone |
| `health_center` | Nom du centre de santé |
| `org_unit` | Unité organisationnelle |
| `enrollment_date` | Date d'enrollment |
| `zone` | Zone géographique |

### Configuration du mapping

```bash
curl -X POST https://api.oomus.health/dhis2/assign-codes \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "programme_uid": "P3jJH5Tu5VC",
    "mappings": [
      {"dhis2_attribute_code": "MMD_PER_NAM", "card_field": "first_name"},
      {"dhis2_attribute_code": "MMD_PER_LST", "card_field": "last_name"},
      {"dhis2_attribute_code": "MMD_PER_DOB", "card_field": "date_of_birth"},
      {"dhis2_attribute_code": "MMD_PER_SEX", "card_field": "gender"},
      {"dhis2_attribute_display_name": "Phone Number", "card_field": "phone_number"}
    ]
  }'
```

---

## Étape 4 — Résolution MPI

Lors de chaque synchronisation, le moteur MPI est invoqué automatiquement. Voici comment comprendre les résultats :

### Indicateurs de synchronisation

Après chaque synchronisation, les métriques suivantes sont disponibles :

| Indicateur | Description |
|---|---|
| `mpi_resolved` | Enrollments liés à un MPI existant |
| `mpi_created` | Nouveaux identifiants MPI créés |
| `mpi_pending_review` | Doublons probables en attente de révision |
| `sensitive_blocked` | Attributs bloqués par la garde IA |

### Traiter les doublons en attente

Rendez-vous dans **Dashboard MPI > Révision des doublons** pour traiter les identités en attente (score 0.75–0.94).

---

## Étape 5 — Configurer la synchronisation automatique

### Choisir l'intervalle

| Intervalle | Cas d'usage recommandé |
|---|---|
| 10 min | Clinique à fort trafic, données très fraîches requises |
| 15 min | Clinique active, campagne de vaccination en cours |
| 30 min | Usage standard, programmes réguliers |
| 60 min | Programme à faible volume, économie de ressources |
| 180 min | Synchronisation légère, programmes saisonniers |

```bash
curl -X POST https://api.oomus.health/dhis2/sync-schedule \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "programme_uid": "P3jJH5Tu5VC",
    "interval_minutes": 30,
    "enabled": true,
    "auto_generate_cards": false
  }'
```

---

## Étape 6 — Générer les cartes

### Aperçu avant génération en masse

Avant de lancer une génération sur des milliers de bénéficiaires, prévisualisez la carte d'un enrollment réel :

1. Dans l'interface DHIS2 d'Oomus, sélectionnez un enrollment
2. Cliquez sur **"Aperçu carte PNG"**
3. Vérifiez que tous les champs sont correctement mappés et affichés

### Lancer la génération

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

---

## Étape 7 — Distribuer les cartes

Depuis l'onglet **Distribution** de la vue DHIS2 dans Oomus :

1. Sélectionnez les bénéficiaires (individuellement ou en groupe)
2. Choisissez le canal : WhatsApp, SMS ou Google Wallet
3. Personnalisez le message (si applicable)
4. Cliquez sur **"Envoyer"**

Les logs de distribution sont disponibles dans l'onglet **Distribution > Logs**.

---

## Dépannage courant

| Problème | Cause possible | Solution |
|---|---|---|
| Test de connexion échoue (401) | Identifiants incorrects | Vérifiez le nom d'utilisateur et mot de passe DHIS2 |
| Test de connexion échoue (503) | DHIS2 inaccessible | Vérifiez que l'URL est accessible depuis Internet |
| Attributs non mappés | Codes d'attributs incorrects | Vérifiez les codes dans DHIS2 Maintenance > Programmes |
| Synchronisation bloquée | Version DHIS2 < 2.36 | Mettez à jour DHIS2 ou contactez le support |
| Doublons excessifs | Données DHIS2 incohérentes | Nettoyez les données à la source, puis réinitialisez le cache MPI |

---

## Prochaines étapes

- [Référence API DHIS2](../api-reference/dhis2.md)
- [Identité MPI souveraine](../features/mpi-sovereign-identity.md)
- [Distribution multicanal](../features/distribution.md)
