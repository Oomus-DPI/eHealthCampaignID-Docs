# Modules Enterprise v5.2

Quatre modules haute valeur disponibles sur les plans **National Campaign** et **Sovereign Enterprise**. Chaque module s'appuie exclusivement sur les données réelles de votre programme — aucune valeur fictive ni démonstration simulée.

---

## Module 1 — AI Campaign Optimizer

> Intelligence artificielle appliquée à vos campagnes de santé. Prévisions, anomalies, recommandations — tout en temps réel depuis vos données historiques.

### Score de santé composite (0–100)

Un indicateur unique synthétise l'état global de votre programme :

| Composante | Poids | Source de données |
|---|---|---|
| Taux de succès des jobs | 40 % | `GenerationJob.status = 'completed'` — 30 derniers jours |
| Santé quota | 30 % | `EngineUsageRecord` / quota mensuel du plan |
| Santé solde | 30 % | `Programme.balance_fcfa` / quota mensuel du plan |

**Niveaux :**

| Score | Niveau | Signification |
|---|---|---|
| ≥ 75 | `HIGH` — Excellent | Tous les indicateurs dans les normes |
| 60–74 | `MEDIUM` — Satisfaisant | À surveiller |
| 40–59 | `LOW` — Dégradé | Action recommandée |
| < 40 | `CRITICAL` | Intervention urgente requise |

La jauge SVG animée (arc semi-circulaire) affiche le score avec une couleur dynamique. Les trois barres de composantes permettent d'identifier la source de dégradation.

### Prévisions OLS J+30

Régression linéaire par moindres carrés (OLS) calculée sur les 90 derniers jours de données réelles :

- **Cartes générées** : pente (cartes/jour), prévision 30 jours
- **Dépenses** : burn rate (FCFA/jour), projection solde
- **R²** : indicateur de qualité du modèle (0–1) — affiché sur l'interface
- **Confiance** : `min(0.95, max(0.1, R²))` — mise en contexte de la précision

> Si l'historique contient moins de 3 points de données, la section prévision affiche « Données insuffisantes » sans erreur.

### Saturation quota mensuelle

Basé sur la pente OLS et le quota restant, le moteur prédit :

- `will_saturate` : le quota sera-t-il épuisé avant la fin du mois ?
- `days_until_saturation` : nombre de jours estimé
- `saturation_pct` : pourcentage consommé à ce jour

Une alerte critique est déclenchée si la saturation est prévue dans les 7 jours.

### Détection d'anomalies Z-score

Les séries temporelles de génération et de dépenses sont analysées avec un Z-score :

- **Seuil** : 2,2 écarts-types (σ)
- **Fenêtre** : 90 derniers jours
- **Sortie** : liste d'indices anomaux avec valeur et z-score
- **Minimum** : 5 points de données requis pour l'analyse

Chaque anomalie est affichée visuellement dans la carte `AnomalyMap` avec son type (cartes ou dépenses) et sa date.

### Recommandations contextuelles

Des recommandations priorisées sont générées automatiquement à partir de vos métriques réelles :

| Priorité | Déclencheur |
|---|---|
| `critical` | Quota ≥ 90%, projection solde négative, taux succès < 70% |
| `high` | Quota ≥ 75%, solde insuffisant, taux succès < 90% |
| `medium` | > 5 échecs sync DHIS2, > 10 échecs distribution |
| `low` | Système nominal |

Les recommandations DHIS2 et distribution utilisent des données réelles issues de `GeoSyncLog` et `GeoDistributionEvent`.

---

## Module 2 — Geospatial Command Center

> Surveillance géographique en temps réel de vos campagnes. Visualisez la couverture, identifiez les zones à risque et suivez les flux de distribution par région.

### Dérivation géographique automatique

Le module extrait automatiquement la région de chaque campagne à partir de son **préfixe** (ex. `DKR-2026-VAC-01` → Dakar) grâce à `PREFIX_REGION_MAP` :

| Préfixe | Région |
|---|---|
| `DKR`, `SN` | Dakar |
| `THIS`, `TH` | Thiès |
| `ZIG` | Ziguinchor |
| `DIO` | Diourbel |
| `SL` | Saint-Louis |
| `MAT` | Matam |
| `LOU` | Louga |
| `KED` | Kédougou |
| `GN` | Conakry (Guinée) |
| `ML` | Bamako (Mali) |

Aucune configuration supplémentaire n'est requise — les données géographiques émergent naturellement de vos préfixes de campagnes.

### KPI géo

Quatre indicateurs clés en temps réel :

| KPI | Description |
|---|---|
| **Régions actives** | Nombre de régions distinctes identifiées dans les préfixes |
| **Cartes générées** | Volume total `GenerationJob.total_lots` (tous temps) |
| **Campagnes** | Nombre de campagnes du programme |
| **Zones à risque** | Régions avec `risk_score > 30` ou `coverage < 40%` |

### Heatmap régionale

La heatmap affiche la distribution d'une métrique sur toutes les régions :

| Métrique | Description |
|---|---|
| `cards_generated` | Volume de cartes par région |
| `coverage_pct` | Pourcentage de couverture estimé |
| `risk_score` | Score de risque composite (0–100) |
| `distribution_success_rate` | Taux de livraison des distributions |
| `beneficiaries_count` | Nombre de bénéficiaires enregistrés |

L'intensité normalisée (0–1) permet la comparaison inter-régions indépendamment des volumes absolus.

### Modèle de risque régional

Chaque région reçoit un score de risque composite :

| Dimension | Poids max | Seuils |
|---|---|---|
| **Couverture** | 40 pts | < 20% → 40 pts · < 40% → 25 pts · < 60% → 10 pts |
| **Succès distribution** | 35 pts | < 60% → 35 pts · < 75% → 20 pts · < 85% → 8 pts |
| **Volume cartes** | 25 pts | < 100 → 25 pts · < 500 → 15 pts · < 1 000 → 5 pts |

Régions sans données → score élevé (≥ 60 pts) — elles sont signalées comme risque `HIGH` plutôt que masquées.

### Flux d'événements live

Le panel **Flux d'événements live** affiche les derniers `GeoDistributionEvent` :

- **Canal** : WhatsApp (vert), SMS (ambre), QR (violet)
- **Type** : `delivered` (vert), `sent` (ambre), `failed` (rouge)
- **Auto-refresh** : toutes les 30 secondes
- **Fallback offline** : démonstration locale si API inaccessible (visuellement distinct)

---

## Module 3 — Health Trust Score

> Un score de confiance numérique pour chaque identité MPI. Évalue la fiabilité, la complétude et l'intégrité des données de santé d'un bénéficiaire.

### Calcul du score (0–100)

Le Trust Score est une valeur composite basée sur :

| Signal | Impact |
|---|---|
| Identité vérifiée KYC | +30 points |
| NIN renseigné | +15 points |
| Téléphone renseigné | +10 points |
| Date de naissance renseignée | +10 points |
| Lié à un programme actif | +10 points par programme (max 3) |
| Doublon probable signalé | -20 points |
| Données démographiques incohérentes | -10 points |

**Classification :**

| Score | Niveau | Utilisation recommandée |
|---|---|---|
| ≥ 80 | Haute confiance | Autoriser distribution, émission Wallet |
| 60–79 | Confiance modérée | Vérification complémentaire conseillée |
| 40–59 | Confiance faible | Validation agent de terrain requise |
| < 40 | Non fiable | Bloquer les services sensibles |

### Signaux de risque

Les flags de risque sont automatiquement attachés à une identité :

- **Doublon probable** : un autre MPI avec score ≥ 75 sur les mêmes données démographiques
- **Incohérence démographique** : âge improbable, format téléphone invalide, genre vs prénom
- **Absence de NIN** : pour les programmes nécessitant une identité légale

### Recalcul admin

L'endpoint `POST /api/v1/mpi/trust/admin/recompute` permet de recalculer les scores en batch pour tous les MPI d'un ou plusieurs programmes après une mise à jour de données.

---

## Module 4 — Sovereign Wallet

> Portefeuille de passes numériques souverains, fonctionnel hors ligne. Chaque bénéficiaire dispose d'un pass digital signé cryptographiquement, synchronisable sur ses appareils.

### Passes digitaux

Un **pass** est la représentation numérique de l'éligibilité d'un bénéficiaire à un programme de santé :

| Champ | Description |
|---|---|
| `pass_id` | Identifiant unique (hex 16 chars) |
| `mpi_id` | Identité souveraine liée |
| `holder_name` | Nom du bénéficiaire |
| `pass_type` | `health` / `vaccination` / `distribution` |
| `status` | `active` / `revoked` / `expired` |
| `qr_payload` | Charge utile QR — données essentielles encodées |
| `qr_signature` | Signature HMAC-SHA256 — vérification hors ligne possible |
| `expires_at` | Date d'expiration ISO 8601 |

La création d'un pass est **idempotente** sur `mpi_id` : si un pass actif existe déjà, il est retourné sans dupliquer.

### Bundle offline

Le bundle offline permet de stocker un pass sur un appareil sans connexion :

```json
{
  "algo": "xor-sha256",
  "encrypted": "<base64 data>",
  "hash": "<SHA-256 integrity>",
  "pass_id": "...",
  "expires_at": "..."
}
```

Le déchiffrement se fait localement avec la clé dérivée de l'appareil — aucun appel réseau requis.

### Synchronisation appareils

1. Enregistrez un appareil : `POST /wallet/device/register` avec un `device_token` unique
2. Récupérez les passes à synchroniser : `GET /wallet/sync?device_token={token}`
3. L'historique de synchronisation est consultable : `GET /wallet/sync/history`

### Indicateur offline (frontend)

Si l'API Wallet est temporairement inaccessible, l'interface :

1. Charge les passes depuis le cache `localStorage` (mis à jour à chaque fetch réussi)
2. Affiche un **banner ambre** : _"Données en cache — API inaccessible"_ avec horodatage
3. Propose un bouton **Réessayer** pour déclencher un nouveau fetch

Aucune donnée périmée n'est affichée sans indication visible à l'opérateur.

### Sécurité cryptographique

| Mécanisme | Algorithme | Usage |
|---|---|---|
| Signature QR | HMAC-SHA256 | Vérification authenticité hors ligne |
| Bundle offline | Chiffrement symétrique | Chiffrement données sur appareil |
| Transport | HTTPS · TLS 1.3 | Communication API |
| Révocation | Auditée + horodatée | Journalisée dans `WalletRevocation` |

---

## Accès aux modules

Les modules enterprise sont disponibles sur les plans suivants :

| Plan | AI Optimizer | Geo Command | Trust Score | Sovereign Wallet |
|---|---|---|---|---|
| `starter` | — | — | — | — |
| `regional_ops` | — | — | — | — |
| `national_campaign` | ✓ | ✓ | ✓ | ✓ |
| `sovereign_enterprise` | ✓ | ✓ | ✓ | ✓ |

Pour activer un module : contactez votre gestionnaire de compte Oomus ou accédez à **Paramètres → Modules Premium**.

---

## Prochaines étapes

- [Référence API Enterprise](../api-reference/enterprise-modules.md) — Documentation technique des endpoints
- [Plans & Fonctionnalités](../getting-started/plans-and-pricing.md) — Comparer les plans
- [Intégration DHIS2](../features/dhis2-integration.md) — Synchroniser vos données terrain
- [Identité souveraine MPI](../features/mpi-sovereign-identity.md) — Registre national de santé
