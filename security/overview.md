# Vue d'ensemble — Sécurité

La sécurité est un principe de conception fondamental d'Oomus CampaignID, pas une couche ajoutée après coup. Cette page présente les principes directeurs et les mécanismes de sécurité de la plateforme.

---

## Principes fondamentaux

### Moindre privilège

Chaque composant, utilisateur et processus n'accède qu'aux ressources strictement nécessaires à son rôle :

- Les comptes DHIS2 utilisés par Oomus ont des droits en **lecture seule**
- Les workers de génération ne peuvent pas accéder aux données de facturation
- Les utilisateurs `programme_user` ne peuvent pas modifier les configurations

### Secrets hors du code

Aucune information sensible (clés, mots de passe, tokens) n'est jamais stockée dans le code source ou les dépôts Git. Toutes les valeurs sensibles sont gérées par des variables d'environnement et des gestionnaires de secrets dédiés.

### Validation stricte des fichiers

Tous les fichiers uploadés (logos, templates) sont validés à plusieurs niveaux :
- Vérification du type MIME (whitelist stricte)
- Contrôle de la taille maximale
- Analyse du contenu (pas d'exécutable déguisé)
- Stockage dans un espace isolé du code applicatif

### Isolation des environnements

Les environnements de développement, staging et production sont strictement isolés. Les données de production ne sont jamais utilisées dans les environnements de test.

---

## Authentification

### JWT HS256

Oomus CampaignID utilise des tokens **JWT (JSON Web Token)** avec algorithme **HS256** :

- **Access token** : durée de vie de **30 minutes** — minimise le risque en cas de fuite
- **Refresh token** : durée de vie de **30 jours** — stocké sécurisement côté client
- Rotation automatique des tokens à chaque rafraîchissement

### Mots de passe

Tous les mots de passe utilisateur sont hachés avec **bcrypt** :
- Facteur de coût adapté aux recommandations NIST
- Jamais stockés en clair, jamais loggés
- Politique de complexité appliquée à la création et au changement

### Sessions concurrentes

Oomus CampaignID ne limite pas le nombre de sessions concurrentes par défaut, mais les tokens peuvent être révoqués par l'administrateur du programme si une compromission est suspectée.

---

## Autorisation — RBAC institutionnel

Oomus CampaignID implémente un contrôle d'accès basé sur les rôles (**RBAC**) au niveau institutionnel.

### Rôles standard

| Rôle | Description | Droits |
|---|---|---|
| `super_admin` | Administrateur global de la plateforme | Accès complet à tous les programmes et la configuration système |
| `programme_admin` | Administrateur d'un programme | Gestion complète de son programme (campagnes, utilisateurs, configuration) |
| `programme_user` | Utilisateur opérationnel | Consultation, lancement de jobs, distribution — pas de configuration |

### Rôles personnalisés

Sur les plans National Campaign et Sovereign Enterprise, des rôles personnalisés peuvent être créés avec des permissions granulaires sur :
- La gestion des campagnes (lecture / création / modification / suppression)
- La génération de cartes (lancement, annulation)
- La distribution (lecture, exécution)
- La gestion des utilisateurs (lecture, invitation, désactivation)
- La facturation (lecture, demande de rechargement)
- La configuration DHIS2 (lecture, modification)
- Les analytics (lecture)

### Périmètre d'accès

Chaque utilisateur est associé à **un programme** et ne peut pas accéder aux données d'autres programmes (isolation stricte des données par organisation).

---

## Piste d'audit immuable

Toutes les actions significatives réalisées sur la plateforme sont enregistrées dans une **piste d'audit immuable**.

### Ce qui est enregistré

| Événement | Données enregistrées |
|---|---|
| Connexion utilisateur | Acteur, IP source, timestamp, succès/échec |
| Création / modification de campagne | Acteur, action, données modifiées |
| Lancement de job de génération | Acteur, campagne, volume, DPI |
| Distribution de carte | Acteur, canal, timestamp (sans numéro de téléphone complet) |
| Modification de configuration | Acteur, paramètre modifié, ancienne/nouvelle valeur |
| Changement de rôle utilisateur | Acteur, cible, rôle attribué |
| Changement de plan | Acteur, plan source, plan cible |
| Fusion MPI (merge) | Acteur, MPI source, MPI cible, raison |

### Caractéristiques de la piste d'audit

- **Immuable** : les entrées d'audit ne peuvent pas être modifiées ni supprimées (même par un super_admin)
- **Horodatée** : timestamp UTC précis à la seconde
- **IP source** : adresse IP de l'acteur enregistrée
- **Non répudiable** : chaque entrée lie une action à un acteur identifié

---

## Workflows d'approbation multi-niveaux

Les opérations à fort impact nécessitent une approbation explicite :

- **Changement de plan** : notification à l'administrateur
- **Simulation → Provisionnement** : workflow d'approbation admin (submitted → approved → provisioning → provisioned)
- **Fusion MPI** : requiert une justification écrite (auditée)
- **Rechargement de solde** : demande soumise pour validation

---

## Protection des données en transit et au repos

- **En transit** : HTTPS/TLS 1.3 recommandé pour toutes les communications (HTTP uniquement en développement local)
- **Au repos** : données stockées dans PostgreSQL avec chiffrement recommandé au niveau du volume
- **Fichiers générés** : stockés dans MinIO/S3-compatible, accès via URLs signées à durée limitée
- **Secrets de configuration** : gérés par variables d'environnement, jamais en base de données

---

## Prochaines étapes

- [Garanties cryptographiques](cryptographic-guarantees.md) — Détails des algorithmes
- [Vérification hors ligne](offline-verification.md) — Sécurité du portail
- [Protection des données](data-protection.md) — Gestion des données personnelles
- [Conformité légale](../compliance/legal-compliance.md)
