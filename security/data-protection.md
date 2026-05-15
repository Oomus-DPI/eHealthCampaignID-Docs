# Protection des données

Oomus CampaignID est conçu selon le principe de **minimisation des données** : seules les informations strictement nécessaires au fonctionnement du service sont collectées et traitées.

---

## Catégories de données traitées

### Données du programme (organisation)

| Donnée | Description | Finalité |
|---|---|---|
| Nom de l'organisation | Nom du programme ou ministère | Identification du compte |
| Adresse e-mail | Contact du responsable programme | Authentification, notifications |
| Plan tarifaire | Plan souscrit, quotas | Facturation, accès aux fonctionnalités |
| Configuration DHIS2 | URL et identifiants (chiffrés) | Synchronisation des enrollments |
| Logs d'audit | Actions effectuées (acteur, action, IP) | Sécurité, conformité |

### Identifiants bénéficiaires

| Donnée | Description | Ce qui est stocké |
|---|---|---|
| Identifiant MPI | Code souverain unique | Oui — c'est la donnée principale |
| Prénom / Nom | Nom du bénéficiaire | Oui — affiché sur la carte |
| Date de naissance | DDN du bénéficiaire | Oui — utilisée pour la déduplication MPI |
| Sexe | Genre | Oui — utilisé pour la déduplication MPI |
| Numéro de téléphone | Contact du bénéficiaire | Oui, si fourni (pour distribution SMS/WhatsApp) |
| District / Région | Zone géographique | Oui — composant du MPI ID |
| UID DHIS2 | Identifiant externe | Oui, si lié via DHIS2 |

### Codes cartes

| Donnée | Description | Stockage |
|---|---|---|
| Code carte unique | Identifiant Base36 de la carte | Haché SHA-256 dans le registre offline |
| Jeton QR | Jeton opaque SHA-256 | Stocké haché uniquement |
| Statut de la carte | `valid` / `revoked` | Oui |
| Date d'émission | Date de génération | Oui |

---

## Ce qui n'est jamais stocké

Oomus CampaignID ne stocke **jamais** :

- Des dossiers médicaux complets (diagnostics, traitements, prescriptions)
- Des données biométriques brutes (empreintes digitales, iris, ADN)
- Des numéros de carte d'identité nationale (sauf si fournis volontairement comme `external_id`)
- Des données financières personnelles (numéros de compte bancaire)
- Des données de localisation en temps réel

---

## Principes RGPD et minimisation

Bien qu'Oomus CampaignID ne soit pas soumis directement au RGPD européen (si déployé dans des pays hors UE), nous appliquons les principes RGPD comme bonne pratique universelle :

| Principe RGPD | Application dans Oomus CampaignID |
|---|---|
| **Minimisation** | Seules les données nécessaires à l'émission et la vérification de cartes sont collectées |
| **Limitation de la finalité** | Les données bénéficiaires ne sont utilisées que pour la génération et distribution de cartes |
| **Exactitude** | Interface de correction des données via le registre MPI |
| **Limitation de la conservation** | Durée de rétention configurable par programme |
| **Intégrité et confidentialité** | Chiffrement AES-256-GCM, HTTPS, RBAC, audit trail |
| **Responsabilité** | Piste d'audit complète, workflows d'approbation |

---

## Garde des données sensibles — 7 catégories

Avant tout export vers les pipelines analytiques ou ML, Oomus CampaignID applique une **garde automatique des données sensibles**. Les attributs DHIS2 appartenant aux 7 catégories suivantes sont **automatiquement exclus** de ces traitements :

| Catégorie | Exemples d'attributs concernés |
|---|---|
| **HIV/SIDA** | Statut sérologique HIV, charge virale, CD4, ARV |
| **IST** | Syphilis, gonorrhée, chlamydia, résultats IST |
| **Tuberculose** | Statut TB, BAAR positif, traitement DOT, MDR-TB |
| **Santé mentale** | Diagnostics psychiatriques, médicaments psychotropes |
| **Sérologique** | Résultats de tests sérologiques sensibles (au-delà HIV/IST) |
| **Biométrique** | Empreintes digitales, iris, scanners faciaux, ADN |
| **Financier** | Revenus déclarés, statut socio-économique, paiements de santé |

Ces attributs restent **présents et chiffrés dans la base de données** pour les besoins légitimes du programme. Ils sont uniquement exclus des :
- Exports CSV analytiques
- Pipelines de machine learning
- Logs de débogage
- API analytics
- Tableaux de bord partagés

### QR code et vie privée

Le QR code imprimé sur la carte contient un **jeton opaque dérivé par SHA-256** — et non l'identifiant MPI ou le nom du bénéficiaire. Cette conception garantit que :
- Scanner la carte n'expose aucune information personnelle
- Un agent malveillant collectant des données QR ne peut rien en faire
- La vérification révèle uniquement "valide / invalide", pas l'identité

---

## Rétention des données

| Donnée | Durée par défaut | Configurable |
|---|---|---|
| Identités MPI | Durée du programme + 5 ans | Oui (par programme) |
| Codes cartes | Durée de la campagne + 2 ans | Oui |
| Logs d'audit | 5 ans | Non (conformité) |
| Logs de distribution | 1 an | Oui |
| Artifacts PDF/ZIP | 6 mois | Oui |
| Tokens de session | Durée du token (30 min / 30 jours) | Non |

---

## Droits des bénéficiaires

Bien que les bénéficiaires n'interagissent pas directement avec l'API Oomus, leurs droits peuvent être exercés via le programme de santé responsable :

| Droit | Mécanisme |
|---|---|
| **Accès** | Le responsable programme peut exporter les données d'un bénéficiaire sur demande |
| **Rectification** | Le registre MPI peut être mis à jour via `PATCH /mpi/{mpi_id}` |
| **Suppression** | Désactivation de l'identité MPI possible (contact support) |
| **Opposition** | Révocation de la carte possible depuis l'interface campagne |

---

## Option hébergement souverain

Pour les programmes nécessitant que les données restent sur le territoire national :

- **Hébergement souverain** : déploiement dédié sur l'infrastructure nationale ou régionale du programme
- **Isolation complète** : les données ne quittent jamais le pays
- **Conformité réglementaire locale** : adapté aux réglementations nationales de protection des données de santé

L'option hébergement souverain est disponible avec le plan Sovereign Enterprise.

---

## Prochaines étapes

- [Conformité légale](../compliance/legal-compliance.md)
- [Vue d'ensemble sécurité](overview.md)
- [Garanties cryptographiques](cryptographic-guarantees.md)
