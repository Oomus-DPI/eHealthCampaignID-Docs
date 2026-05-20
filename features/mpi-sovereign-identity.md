# Identité souveraine MPI

Le **Master Patient Index (MPI) souverain** est l'une des fonctionnalités les plus différenciatrices d'Oomus CampaignID. Il garantit à chaque citoyen une identité de santé numérique unique, pérenne et interopérable entre tous les programmes de santé.

---

## Qu'est-ce que le MPI souverain ?

Le MPI répond à un problème fondamental des systèmes de santé fragmentés : **le même individu peut être enregistré sous des identités différentes dans des programmes différents** — avec des orthographes de nom variées, des dates de naissance imprécises, des identifiants locaux sans lien entre eux.

### Principe fondamental

> **1 citoyen = 1 identifiant de santé numérique souverain**

Cet identifiant est :
- **Cross-programme** : partagé entre vaccination, nutrition, HIV, assurance maladie, etc.
- **À vie** : créé à la première interaction et conservé indéfiniment
- **Souverain** : hébergé sur l'infrastructure nationale (ou de l'organisation), sans dépendance à un tiers
- **Interopérable** : compatible HL7 FHIR R4, DHIS2 Tracker, OpenHIE

---

## Format de l'identifiant MPI

Un identifiant MPI Oomus suit le format :

```
SN-DKR-26-9XQ7LM2A
```

Décomposition de chaque segment :

| Segment | Exemple | Description |
|---|---|---|
| **Pays** | `SN` | Code ISO 3166-1 alpha-2 du pays |
| **Région/District** | `DKR` | Code de la région ou du district sanitaire (3 lettres) |
| **Année** | `26` | Les 2 derniers chiffres de l'année d'enregistrement |
| **Code unique** | `9XQ7LM2A` | Identifiant Base36 aléatoire cryptographiquement sécurisé (8 caractères, ~2,8 milliards de combinaisons) |

Le code unique est généré par un **CSPRNG (Cryptographically Secure Pseudo-Random Number Generator)**, garantissant l'absence de collision et la non-prédictibilité.

---

## Moteur de déduplication probabiliste

Lorsqu'un nouveau bénéficiaire est enregistré (via l'API ou via DHIS2), le moteur MPI vérifie s'il existe déjà dans le registre avant de créer un nouvel identifiant.

### Algorithme de score

Le moteur calcule un **score de similarité composite** basé sur plusieurs attributs :

| Attribut | Algorithme | Poids |
|---|---|---|
| Prénom | Jaro-Winkler + normalisation africaine | 25% |
| Nom de famille | Jaro-Winkler + normalisation africaine | 25% |
| Date de naissance | Correspondance exacte / ±1 an | 20% |
| Sexe | Correspondance exacte | 10% |
| Commune / District | Correspondance administrative | 10% |
| Identifiant externe | Correspondance DHIS2 UID / autre ID | 10% |

### Normalisation des noms africains

Le moteur embarque une couche de normalisation spécialisée pour les noms de l'Afrique de l'Ouest :
- Gestion des variantes orthographiques (Ibrahima / Ibrahim / Brahim)
- Fusion des diacritiques (é → e, ñ → n, ñ → n)
- Inversion prénom/nom fréquente dans certaines régions
- Reconnaissance des noms composés (Mamadou Lamine / Lamine Mamadou)
- Translitération Wolof/Pulaar/Mandingue

### Seuils de décision

| Score | Décision | Action |
|---|---|---|
| ≥ 0.95 | Match certain | Liaison automatique à l'identité existante |
| 0.75 – 0.94 | Match probable | Marquage pour révision manuelle (interface Doublons) |
| 0.50 – 0.74 | Match possible | Alerte dans le tableau de bord |
| < 0.50 | Pas de match | Nouvelle identité créée |

---

## Jeton QR — Préservation de la vie privée

Le QR code imprimé sur la carte ne contient **jamais** l'identifiant MPI en clair.

### Pourquoi ce choix ?

Si l'identifiant MPI était directement encodé dans le QR, n'importe qui disposant d'un lecteur QR générique pourrait lire l'identité de santé d'un bénéficiaire simplement en scannant sa carte — une violation grave de la vie privée.

### Comment cela fonctionne

À la place, le QR code contient un **jeton opaque** :
- Dérivé de l'identifiant MPI + du code de campagne + d'un secret serveur via SHA-256
- Non réversible : il est impossible de retrouver le MPI à partir du jeton seul
- Vérifiable : le serveur Oomus peut confirmer la validité d'un jeton sans exposer le MPI
- Vérifiable hors ligne : le registre `registry.json` permet la vérification sans connexion

```
Jeton QR = SHA-256(mpi_id + campaign_prefix + server_secret)
```

> Ce mécanisme garantit qu'un agent de terrain ne peut vérifier qu'une seule information : "cette carte est valide pour cette campagne". Il n'apprend rien d'autre sur le bénéficiaire.

---

## Identifiant MPI sur les cartes

Pour les 10 modèles de cartes DHIS2 (vital / emerald / pulse / mothercare / shield / nomad / aero / horizon / aurora / sovereign), l'identifiant MPI souverain s'affiche dans le **pied de carte**, remplaçant le code-barre traditionnel. Le modèle `sovereign` affiche le MPI-ID de manière proéminente en gros caractères gras dans la zone identité (design boarding-pass 1011×375 px @ 300 DPI), accompagné d'une icône empreinte digitale (8 crêtes). La fonction `generate_dhis2_card_png()` accepte un paramètre optionnel `sovereign_config: Optional[SovereignCardConfig]` pour personnaliser les couleurs accent/fond, l'échelle de fonte et le nombre maximum d'attributs affichés.

**Avantages par rapport au code-barre :**
- Lisible par l'agent de santé (format humain)
- Utilisable pour recherche manuelle dans le registre
- Plus robuste à la dégradation physique de la carte
- Porteur d'information sémantique (pays, région, année)

---

## Flux de vérification KYC

Le flux KYC (Know Your Beneficiary) permet de vérifier et d'enrichir l'identité d'un bénéficiaire :

```
1. L'agent de santé scanne la carte (QR ou saisie manuelle du MPI)
2. Requête au portail de vérification (en ligne ou hors ligne)
3. Validation du jeton QR cryptographique
4. Affichage du statut : valide / invalide / révoqué
5. [Si en ligne] Affichage du nombre de programmes associés (sans PII)
```

---

## Flux DHIS2 → Résolution MPI

Lors d'une synchronisation DHIS2, chaque enrollment est traité selon le flux suivant :

```
DHIS2 Enrollment
       │
       ▼
Extraction attributs (prénom, nom, DDN, sexe, etc.)
       │
       ▼
Normalisation des noms (algorithme africain)
       │
       ▼
Calcul score similarité (moteur probabiliste)
       │
       ├─ Score ≥ 0.95 → Liaison au MPI existant
       │
       ├─ Score 0.75–0.94 → Queue révision manuelle
       │
       └─ Score < 0.75 → Création nouvel identifiant MPI
                │
                ▼
         Génération code Base36 (CSPRNG)
         Format: {PAYS}-{REGION}-{ANNÉE}-{8 chars}
```

---

## Interopérabilité HL7 FHIR R4

Le MPI Oomus est compatible avec les standards d'interopérabilité internationaux :

- **Ressource Patient FHIR R4** : chaque entrée MPI est exposable comme ressource Patient
- **Requêtes PIXm** (Patient Identity Cross-Reference for Mobile) : résolution d'identité cross-programme via requêtes FHIR
- **Endpoint FHIR** : `https://api.oomus.health/fhir/r4/`

Voir [Intégration HL7 FHIR R4](../integrations/fhir-r4.md) pour la documentation complète.

---

## Endpoints API publics MPI

| Méthode | Endpoint | Description |
|---|---|---|
| `POST` | `/mpi/register` | Enregistrer un nouveau bénéficiaire |
| `GET` | `/mpi/search` | Rechercher dans le registre MPI |
| `GET` | `/mpi/{mpi_id}` | Obtenir les détails d'une identité MPI |
| `PATCH` | `/mpi/{mpi_id}` | Mettre à jour une identité MPI |
| `POST` | `/mpi/match` | Vérifier une correspondance (score) |
| `POST` | `/mpi/{mpi_id}/link-dhis2` | Lier un UID DHIS2 à un MPI |
| `GET` | `/mpi/resolve-dhis2/{dhis2_uid}` | Résoudre un UID DHIS2 vers un MPI |
| `POST` | `/mpi/merge` | Fusionner deux identités en doublon |
| `GET` | `/mpi/{mpi_id}/card` | Obtenir la carte associée |
| `GET` | `/mpi/verify/{token}` | Vérifier un jeton QR **(public, sans auth)** |
| `GET` | `/mpi/qr-config` | Configuration QR active |

Voir [Référence API — MPI](../api-reference/mpi.md) pour les détails complets avec exemples.

---

## Interface utilisateur MPI

L'interface d'Oomus CampaignID inclut trois pages dédiées au MPI :

### Dashboard MPI
Vue d'ensemble des statistiques du registre : nombre total d'identités, taux de déduplication, répartition géographique, évolution temporelle.

### Registre MPI
Tableau paginé et filtrable de toutes les identités du registre. Recherche par nom, MPI ID, district, programme associé. Accès aux détails de chaque identité.

### Révision des doublons
Interface dédiée à la révision manuelle des matchs probables (score 0.75–0.94). L'agent peut :
- Consulter les deux identités côte à côte
- Voir le score de similarité et les champs qui ont contribué
- Valider la fusion (merge) ou rejeter la correspondance
- Ajouter une note de décision (auditée)

---

## Prochaines étapes

- [Référence API — MPI](../api-reference/mpi.md)
- [Intégration DHIS2](dhis2-integration.md) — Résolution MPI lors de la synchronisation
- [HL7 FHIR R4](../integrations/fhir-r4.md)
- [Protection des données](../security/data-protection.md)
