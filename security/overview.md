# Vue d'ensemble — Sécurité

La sécurité est un principe de conception fondamental d'Oomus CampaignID, pas une couche ajoutée après coup. Cette page présente les principes directeurs et les mécanismes de sécurité de la plateforme.

---

## Principes fondamentaux

### Moindre privilège

Chaque composant, utilisateur et processus n'accède qu'aux ressources strictement nécessaires à son rôle :

- Les comptes DHIS2 utilisés par Oomus ont des droits en **lecture seule**
- Les workers de génération ne peuvent pas accéder aux données de facturation
- Les utilisateurs standard ne peuvent pas modifier les configurations système

### Secrets hors du code

Aucune information sensible (clés, mots de passe, tokens) n'est jamais stockée dans le code source ou les dépôts Git. Toutes les valeurs sensibles sont gérées par des variables d'environnement et des gestionnaires de secrets dédiés.

### Validation stricte des fichiers

Tous les fichiers uploadés (logos, templates) sont validés à plusieurs niveaux :
- Vérification du type MIME et des magic bytes
- Contrôle de la taille maximale
- Analyse du contenu (pas d'exécutable déguisé)
- Stockage dans un espace isolé du code applicatif

### Isolation des environnements

Les environnements de développement, staging et production sont strictement isolés. Les données de production ne sont jamais utilisées dans les environnements de test.

---

## Authentification

Oomus CampaignID utilise des tokens **JWT (JSON Web Token)** à courte durée de vie avec rotation automatique :

- Tokens d'accès à durée de vie limitée — renouvellement automatique via refresh token
- **Authentification à deux facteurs (2FA)** via TOTP (Google Authenticator, Authy) disponible pour tous les comptes
- **Protection brute-force** : verrouillage automatique après tentatives répétées échouées
- **Révocation de session** : déconnexion individuelle ou globale (tous les appareils)
- **Versioning de token** : un changement de mot de passe invalide immédiatement toutes les sessions actives

### Mots de passe

Tous les mots de passe utilisateur sont hachés avec une fonction de hachage adaptative conforme aux recommandations NIST :
- Jamais stockés en clair, jamais loggés
- Politique de complexité renforcée appliquée à l'inscription et au changement
- Vérification contre les bases de mots de passe compromis (via protocole k-anonymat)

---

## Autorisation — RBAC institutionnel

Oomus CampaignID implémente un contrôle d'accès basé sur les rôles (**RBAC**) au niveau institutionnel.

### Rôles standard

| Rôle | Description |
|---|---|
| **Super Admin** | Administrateur global de la plateforme |
| **Programme Admin** | Gestion complète d'un programme |
| **Utilisateur opérationnel** | Consultation, lancement de jobs, distribution |

### Rôles personnalisés

Sur les plans National Campaign et Sovereign Enterprise, des rôles personnalisés peuvent être créés avec des permissions granulaires sur les campagnes, la génération, la distribution, les utilisateurs, la facturation et les analytics.

### Périmètre d'accès

Chaque utilisateur est associé à **un programme** et ne peut pas accéder aux données d'autres programmes — isolation stricte des données par organisation.

---

## Piste d'audit immuable

Toutes les actions significatives réalisées sur la plateforme sont enregistrées dans une **piste d'audit immuable** :

- Connexions et tentatives d'authentification
- Modifications de campagnes et configurations
- Lancements de génération et distributions
- Changements de rôles et de plans
- Opérations sur les identités MPI

**Caractéristiques :** immuable, horodatée UTC, liée à l'IP source, non répudiable.

---

## Workflows d'approbation multi-niveaux

Les opérations à fort impact nécessitent une approbation explicite :

- **Simulation → Provisionnement** : workflow d'approbation admin
- **Fusion MPI** : requiert une justification écrite (auditée)
- **Rechargement de solde** : demande soumise pour validation

---

## Protection des données en transit et au repos

- **En transit** : HTTPS/TLS recommandé pour toutes les communications. Headers HSTS activés en production.
- **Au repos** : données PostgreSQL avec chiffrement recommandé au niveau du volume
- **Fichiers générés** : stockage objet S3-compatible, accès via URLs signées à durée limitée
- **Isolation réseau** : bases de données et services internes sans port externe exposé
- **Headers sécurité** : Content-Security-Policy, X-Frame-Options, X-Content-Type-Options sur toutes les réponses API

---

## Prochaines étapes

- [Garanties cryptographiques](cryptographic-guarantees.md) — Propriétés de sécurité des données
- [Vérification hors ligne](offline-verification.md) — Sécurité du portail terrain
- [Protection des données](data-protection.md) — Gestion des données personnelles
- [Conformité légale](../compliance/legal-compliance.md)
