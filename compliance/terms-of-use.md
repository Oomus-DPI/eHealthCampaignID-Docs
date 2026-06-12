# Conditions Générales d'Utilisation — Oomus CampaignID

**Infrastructure Numérique Publique de Santé · Édition Continentale Africaine**

> Version 1.0 · Date d'entrée en vigueur : 1er juin 2026  
> Éditeur : Oomus — Dakar, Sénégal · ceo@oomus.org  
> Applicable à tous les programmes déployés en Afrique et dans les pays à revenus faibles et intermédiaires

---

## Préambule

Les présentes Conditions Générales d'Utilisation (ci-après « CGU ») régissent l'accès et l'utilisation de la plateforme **Oomus CampaignID**, infrastructure numérique publique de santé (DPI — Digital Public Infrastructure) conçue pour la gestion souveraine de l'identité sanitaire, la génération et la distribution de cartes de santé numériques, la vérification cryptographique et la gouvernance institutionnelle des campagnes de santé à grande échelle.

Oomus CampaignID est développée et opérée par **Oomus**, société de droit sénégalais, dont le siège social est établi à Dakar, Sénégal.

En accédant à la plateforme, en créant un compte ou en utilisant tout ou partie des services décrits aux présentes, **l'Organisation cliente** reconnaît avoir lu, compris et accepté sans réserve les présentes CGU, ainsi que la Politique de Confidentialité et, le cas échéant, l'Accord de Niveau de Service (SLA) et l'Accord de Traitement des Données (DPA) applicables à son plan.

---

## Article 1 — Définitions

Aux fins des présentes CGU, les termes ci-dessous ont la signification suivante :

**« Plateforme »** désigne l'ensemble des services logiciels, API, portails web et applications mobiles constituant Oomus CampaignID, accessibles à l'adresse `app.campaignid.oomus.io` et ses sous-domaines.

**« Organisation cliente »** ou **« Client »** désigne toute entité institutionnelle — ministère de la Santé, agence gouvernementale, organisation internationale, ONG, programme de santé publique ou partenaire technique — ayant souscrit un plan d'utilisation de la Plateforme.

**« Administrateur »** désigne l'utilisateur désigné par l'Organisation cliente pour gérer le compte programme, les accès, la configuration et la facturation.

**« Utilisateur autorisé »** désigne toute personne physique — agent de santé, gestionnaire de programme, opérateur terrain — disposant d'un accès à la Plateforme accordé par l'Organisation cliente.

**« Bénéficiaire »** désigne toute personne physique dont les données de santé sont traitées via la Plateforme dans le cadre d'une campagne de santé publique.

**« Données bénéficiaires »** désigne l'ensemble des données à caractère personnel des bénéficiaires traitées via la Plateforme, incluant les données d'identification, de santé, de géolocalisation et de contact.

**« MPI »** (Master Patient Index) désigne le registre souverain d'identité numérique de santé, attribuant à chaque bénéficiaire un identifiant unique, inter-programmes et pérenne.

**« Identifiant MPI »** désigne l'identifiant alphanumérique unique au format `[PAYS]-[RÉGION]-[ANNÉE]-[CODE8]` attribué à chaque bénéficiaire dans le registre MPI.

**« Carte de santé numérique »** désigne le document numérique sécurisé généré par la Plateforme, intégrant un identifiant MPI, un code QR cryptographique et les attributs de santé du bénéficiaire.

**« QR Engine »** désigne le module cryptographique de génération et de vérification des codes QR, opérant par signatures HMAC-SHA256 et chiffrement AES-256-GCM.

**« Card Studio »** désigne l'éditeur visuel de modèles de cartes de santé intégré à la Plateforme.

**« Portail de vérification »** désigne l'application web hors ligne permettant aux agents terrain de vérifier l'authenticité des cartes sans connexion Internet.

**« Sovereign Wallet »** désigne le module d'émission de passes digitaux compatibles Google Wallet et Apple Wallet.

**« Services de distribution »** désigne les modules d'envoi omnicanal de cartes via WhatsApp (Meta Graph API), SMS (Orange API, Africa's Talking) et Google Wallet.

**« DHIS2 »** désigne le système d'information sanitaire de district (District Health Information Software version 2), système de référence en santé publique africaine.

**« Données d'utilisation »** désigne les métadonnées techniques générées par l'utilisation de la Plateforme (logs d'accès, métriques de performance, statistiques d'utilisation anonymisées).

**« Piste d'audit »** désigne le journal immuable et cryptographiquement lié de toutes les actions significatives réalisées sur la Plateforme.

**« SLA »** (Service Level Agreement) désigne l'accord de niveau de service précisant les engagements de disponibilité, de performance et de support applicables au plan souscrit.

**« Force majeure »** désigne tout événement imprévisible, irrésistible et extérieur rendant impossible l'exécution des obligations contractuelles, incluant les catastrophes naturelles, les conflits armés, les coupures d'infrastructure nationale ou les pandémies.

---

## Article 2 — Description des services

### 2.1 Infrastructure d'identité numérique souveraine (MPI)

La Plateforme fournit un registre MPI souverain permettant :

- L'enregistrement de chaque bénéficiaire avec un identifiant numérique de santé unique, pérenne et inter-programmes
- La déduplication probabiliste des identités via un moteur algorithmique basé sur la correspondance phonétique, la similarité des attributs et les scores de confiance pondérés
- La résolution automatique des doublons avec validation humaine obligatoire avant fusion
- L'interopérabilité HL7 FHIR R4 pour l'échange de données avec les systèmes d'information sanitaires existants
- La gestion du cycle de vie des identités : création, mise à jour, fusion, révocation

### 2.2 Card Studio — Éditeur de cartes de santé

La Plateforme met à disposition un éditeur visuel permettant :

- La création et la personnalisation de modèles de cartes de santé (11 modèles prédéfinis dont le template Sovereign)
- L'intégration des logos institutionnels, couleurs primaires et identités visuelles des programmes
- La configuration des champs dynamiques recto/verso (attributs bénéficiaires, données de santé)
- La génération d'aperçus haute résolution (300, 450 ou 600 DPI) avant lancement en production
- L'export des configurations de modèles aux formats YAML et JSON pour versionnage
- La génération de codes QR cryptographiques opaques non réversibles

### 2.3 Génération de cartes à grande échelle

La Plateforme assure la génération asynchrone de cartes via :

- Des workers de génération parallèles supportant jusqu'à 2 millions de cartes par campagne
- Un suivi de progression en temps réel via WebSocket
- La génération d'artefacts : cartes PDF individuelles, registre d'authenticité (SHA-256), rapport d'audit
- Le remboursement automatique du quota en cas d'échec de génération
- La gestion par lots avec téléchargement ZIP sécurisé des sorties

### 2.4 Intégration DHIS2

La Plateforme propose une intégration native avec DHIS2 Tracker incluant :

- La synchronisation bidirectionnelle des enrollements depuis les instances DHIS2
- Le mapping configurable des attributs DHIS2 vers les champs MPI et les champs de carte
- La génération automatique de cartes depuis les enrollements synchronisés
- Le mode de synchronisation planifiée (intervalles configurables : 10, 15, 30, 60 ou 180 minutes)
- La protection en lecture seule des données DHIS2 (aucune écriture dans l'instance source)
- La garde des données sensibles : 7 catégories de données sanitaires sensibles (HIV/SIDA, addictions, santé mentale, IVG, génétique, biométrie, orientation sexuelle) détectées et marquées automatiquement

### 2.5 Distribution multicanale

La Plateforme offre des services de distribution des cartes numériques via :

**WhatsApp (Meta Graph API)**
- Envoi de cartes de santé par message WhatsApp avec image PNG et message personnalisé
- Envoi individuel ou en masse depuis le module de distribution DHIS2
- Gestion des accusés de réception et logs de livraison
- Fallback automatique vers SMS en cas d'échec WhatsApp

**SMS**
- Envoi de SMS avec lien de téléchargement de la carte ou code de référence
- Intégration Orange API (Afrique de l'Ouest) et Africa's Talking (couverture panafricaine)
- Support multi-opérateurs avec failover automatique

**Google Wallet**
- Émission de passes digitaux individuels et en masse (jusqu'à 100 passes simultanément)
- Passes signés cryptographiquement et synchronisables hors ligne
- Révocation auditée avec piste d'audit immuable

### 2.6 Portail de vérification hors ligne

La Plateforme génère un portail de vérification statique déployable localement permettant :

- La vérification de l'authenticité des cartes sans connexion Internet
- La vérification par saisie manuelle du code ou scan QR
- La vérification en masse via import CSV
- L'utilisation de l'API WebCrypto native des navigateurs sans installation de logiciel
- Le support multilingue (français, anglais, wolof)
- La détection des cartes falsifiées, révoquées ou inconnues

### 2.7 Simulation financière et provisionnement

La Plateforme propose un moteur de simulation permettant :

- L'estimation des coûts d'infrastructure, de communication et de gouvernance avant déploiement
- La génération de proformas institutionnels aux formats PDF et Excel
- Le workflow d'approbation admin à niveaux configurables (soumission → approbation → provisionnement)
- Le provisionnement automatisé d'environnements dédiés sur approbation

### 2.8 Facturation et gouvernance financière

La Plateforme assure une traçabilité financière complète via :

- Un registre comptable centralisé : chaque débit génère automatiquement une Transaction, une Facture signée et un AuditLog immuable
- Des factures formelles numérotées selon le schéma `INV-{TYPE}-{YYYYMM}-{6CHARS}`
- La vérification de l'intégrité des factures via signature cryptographique
- La gestion de quotas mensuels avec calcul automatique des dépassements
- Le tableau de bord de gouvernance financière avec suivi par moteur (Campaign Delivery / Studio Print)

### 2.9 Cartes PVC physiques

La Plateforme propose un service de production de cartes physiques incluant :

- Deux niveaux de qualité : Standard PVC et Offset Industriel
- Suivi de commande en temps réel avec timeline d'état (soumis → en production → expédié → livré)
- Date de livraison estimée calculée automatiquement selon le type d'impression
- Historique des statuts JSON avec horodatage à chaque étape

### 2.10 Intelligence artificielle et analytics

La Plateforme intègre des composants d'intelligence artificielle pour :

- La détection d'anomalies comportementales sur les accès et les scans QR (modèle IsolationForest)
- La déduplication probabiliste des identités bénéficiaires
- Les prévisions de consommation de quota et de génération
- Les recommandations contextuelles de gestion de campagne
- Les analyses géospatiales de couverture et de risque territorial

### 2.11 RBAC et gouvernance institutionnelle

La Plateforme fournit un système de contrôle d'accès institutionnel comprenant :

- 10 niveaux de rôles hiérarchiques (de `sub_account` à `super_admin_global`)
- 18 permissions granulaires configurables par programme
- Des workflows d'approbation multi-niveaux jusqu'à 10 niveaux configurables
- Une piste d'audit immuable de toutes les actions significatives
- L'isolation stricte des données par programme (multi-tenancy)

---

## Article 3 — Conditions d'accès et d'inscription

### 3.1 Éligibilité

L'accès à Oomus CampaignID est réservé aux organisations suivantes :

- Ministères de la Santé et agences gouvernementales de santé publique
- Organisations internationales et agences onusiennes (OMS, UNICEF, UNFPA, PNUD, etc.)
- Organisations non gouvernementales et associations opérant dans le domaine de la santé
- Partenaires techniques et financiers de programmes de santé publique
- Universités et institutions de recherche en santé publique
- Toute autre entité dont l'activité est directement liée à la santé publique, sous réserve de validation par Oomus

L'accès à des fins purement commerciales, marketing ou non liées à la santé publique est expressément exclu.

### 3.2 Création du compte

L'Organisation cliente doit désigner un Administrateur responsable qui :

- Fournit des informations exactes, complètes et à jour lors de l'inscription
- Garantit que l'Organisation cliente dispose des autorisations légales nécessaires pour traiter les données bénéficiaires
- Accepte les présentes CGU au nom de l'Organisation cliente
- Est seul responsable de la sécurité des identifiants d'accès

L'inscription est soumise à validation par Oomus. Oomus se réserve le droit de refuser ou de révoquer tout accès sans justification.

### 3.3 Obligation de conformité préalable

Avant de traiter des données bénéficiaires via la Plateforme, l'Organisation cliente s'engage à :

- Avoir obtenu le consentement éclairé des bénéficiaires ou disposer d'une base légale équivalente selon la réglementation nationale applicable
- Avoir réalisé une Analyse d'Impact relative à la Protection des Données (AIPD) si requise par la réglementation applicable
- Avoir notifié son autorité nationale de protection des données si requis
- Avoir mis en place une politique interne de protection des données

---

## Article 4 — Obligations de l'Organisation cliente

### 4.1 Utilisation conforme

L'Organisation cliente s'engage à utiliser la Plateforme exclusivement dans le cadre de programmes de santé publique licites et à :

- Ne traiter que les données strictement nécessaires aux finalités déclarées (principe de minimisation)
- Informer les bénéficiaires du traitement de leurs données de santé
- Respecter les droits des bénéficiaires (accès, rectification, suppression, opposition)
- Ne pas détourner les données bénéficiaires à des fins autres que celles ayant justifié leur collecte
- Mettre à jour et supprimer les données conformément aux politiques de rétention applicables

### 4.2 Sécurité des accès

L'Organisation cliente est seule responsable de :

- La sécurisation des identifiants de connexion de tous ses Utilisateurs autorisés
- L'activation de l'authentification à deux facteurs (2FA TOTP) pour les comptes administrateurs et les accès aux données sensibles
- La révocation immédiate des accès en cas de départ d'un Utilisateur autorisé ou de compromission suspectée
- La non-divulgation des tokens d'API à des tiers non autorisés
- La gestion des droits d'accès conformément au principe du moindre privilège

### 4.3 Intégrations tierces

L'Organisation cliente utilisant les intégrations DHIS2, WhatsApp, SMS ou Google Wallet s'engage à :

- Disposer des droits d'accès appropriés aux systèmes tiers intégrés
- Respecter les conditions d'utilisation des fournisseurs tiers (Meta/WhatsApp, Orange, Africa's Talking, Google)
- Ne pas configurer des intégrations permettant l'accès non autorisé à des données tierces
- Informer Oomus de toute modification des droits d'accès aux systèmes intégrés

### 4.4 Données de santé sensibles

Le traitement de données relevant de catégories particulières (état de santé, données génétiques, biométrie, HIV/SIDA, santé mentale, addictions, orientation sexuelle) via la Plateforme est soumis aux obligations renforcées suivantes :

- Obtention d'un consentement explicite du bénéficiaire ou d'une base légale spécifique
- Mise en place de mesures de sécurité techniques renforcées
- Tenue d'un registre des activités de traitement
- Désignation d'un Délégué à la Protection des Données (DPD) si requis par la réglementation nationale
- Notification immédiate à Oomus de toute violation de données impliquant ces catégories

### 4.5 Interdictions

Il est expressément interdit à l'Organisation cliente de :

- Utiliser la Plateforme pour surveiller, traquer ou profiler des bénéficiaires à des fins autres que la santé publique
- Vendre, céder, louer ou partager des données bénéficiaires extraites de la Plateforme
- Décompiler, désassembler ou tenter d'extraire le code source de la Plateforme
- Contourner les mécanismes de sécurité, de contrôle d'accès ou de facturation
- Utiliser des robots, scripts ou systèmes automatisés pour un accès non autorisé à l'API
- Stocker des clés API ou tokens dans des dépôts de code publics
- Permettre l'accès à la Plateforme à des entités non autorisées par les présentes CGU

---

## Article 5 — Traitement des données personnelles

### 5.1 Responsabilités partagées

Le traitement des données bénéficiaires s'inscrit dans le cadre d'une responsabilité partagée :

**L'Organisation cliente agit en qualité de Responsable de traitement** — elle détermine les finalités et les moyens du traitement des données bénéficiaires, détient la relation directe avec les bénéficiaires et assume la responsabilité principale du respect des droits des personnes concernées.

**Oomus agit en qualité de Sous-traitant** — elle traite les données bénéficiaires pour le compte exclusif de l'Organisation cliente, selon ses instructions documentées et dans les limites définies au présent article et à l'Accord de Traitement des Données (DPA).

### 5.2 Cadre légal applicable

Le traitement des données bénéficiaires est soumis aux réglementations suivantes, selon les pays d'opération :

**Cadre régional africain :**
- Convention de l'Union Africaine sur la Cybersécurité et la Protection des Données Personnelles (Convention de Malabo, 2014)
- Acte Additionnel de la CEDEAO sur la Protection des Données Personnelles (A/SA.1/01/10, 2010)
- Directives de l'UEMOA relatives à la protection des données personnelles

**Réglementations nationales (liste non exhaustive) :**
- Sénégal : Loi n° 2008-12 relative à la protection des données à caractère personnel — Commission de Protection des Données Personnelles (CDP)
- Côte d'Ivoire : Loi n° 2013-450 relative à la protection des données à caractère personnel — ARTCI
- Mali : Loi n° 2013-015 relative à la protection des données à caractère personnel — APDP
- Burkina Faso : Loi n° 010-2004 AN — CIL (Commission de l'Informatique et des Libertés)
- Niger : Ordonnance n° 2011-22 — ANPRD
- Bénin : Loi n° 2009-09 — AIP
- Cameroun : Loi n° 2010/012 relative à la cybersécurité et cybercriminalité
- Kenya : Data Protection Act, 2019 — ODPC
- Ghana : Data Protection Act, 2012 (Act 843) — DPC
- Nigéria : Nigeria Data Protection Regulation (NDPR) 2019 — NITDA

**Réglementations internationales applicables :**
- Règlement (UE) 2016/679 (RGPD) pour les organisations clientes établies dans l'Union Européenne ou traitant des données de résidents européens
- Recommandations de l'OMS en matière de confidentialité des données de santé

En cas de conflit entre ces réglementations, la loi la plus protectrice des droits des bénéficiaires prévaut.

### 5.3 Données traitées par Oomus

Oomus traite, pour le compte de l'Organisation cliente, les catégories de données suivantes :

| Catégorie | Données | Base légale |
|---|---|---|
| Identité bénéficiaire | Nom, prénom, date de naissance, sexe, nationalité | Exécution de la mission de santé publique |
| Contact | Numéro de téléphone (pour distribution), adresse | Consentement ou mission de santé publique |
| Santé | Attributs de programme (vaccination, nutrition, etc.) | Mission de santé publique, consentement explicite |
| Identification | Identifiant MPI, code carte, numéro NIN (optionnel) | Mission de santé publique |
| Géolocalisation | Région, district sanitaire, unité organisationnelle | Mission de santé publique |
| Données sensibles | HIV, santé mentale, addictions (si activé) | Consentement explicite renforcé obligatoire |

### 5.4 Finalités du traitement

Les données bénéficiaires sont traitées exclusivement aux fins suivantes :

1. Émission et gestion des cartes de santé numériques
2. Déduplication et résolution des identités MPI
3. Distribution des cartes via les canaux autorisés par l'Organisation cliente
4. Vérification de l'authenticité des cartes par les agents terrain
5. Synchronisation avec les systèmes DHIS2 désignés par l'Organisation cliente
6. Génération de statistiques de campagne anonymisées ou pseudonymisées
7. Détection de fraudes et d'anomalies pour la sécurité du programme

Tout traitement à des fins autres que celles listées ci-dessus est formellement interdit.

### 5.5 Droits des bénéficiaires

L'Organisation cliente est responsable de la mise en œuvre effective des droits des bénéficiaires. Oomus fournit les outils techniques permettant :

- **Droit d'accès** : consultation des données MPI associées à un identifiant
- **Droit de rectification** : mise à jour des attributs du profil bénéficiaire
- **Droit à l'effacement** : anonymisation ou suppression des données bénéficiaires sur demande de l'Organisation cliente
- **Droit à la portabilité** : export des données bénéficiaires aux formats CSV ou JSON
- **Droit d'opposition** : blocage du traitement pour un bénéficiaire spécifique

### 5.6 Transferts internationaux de données

Les données bénéficiaires sont hébergées dans les datacenters désignés lors du provisionnement du compte. Pour les plans Sovereign Enterprise, un hébergement souverain sur infrastructure nationale ou régionale est disponible.

Tout transfert de données hors du pays d'opération de l'Organisation cliente est soumis à l'accord préalable de celle-ci et aux mécanismes de transfert appropriés (décision d'adéquation, clauses contractuelles types, ou dispositions équivalentes selon la réglementation nationale applicable).

### 5.7 Durée de conservation

Les données bénéficiaires sont conservées aussi longtemps que le compte programme est actif. En cas de résiliation, les données sont conservées pendant 90 jours puis supprimées définitivement, sauf demande d'export préalable de l'Organisation cliente ou obligation légale de conservation plus longue.

---

## Article 6 — Intelligence artificielle — Garanties éthiques

### 6.1 Principes de l'IA responsable

Oomus s'engage à déployer des systèmes d'intelligence artificielle conformément aux principes suivants :

- **Non-discrimination** : les algorithmes de déduplication sont régulièrement audités pour détecter et corriger les biais, en particulier sur les noms et prénoms africains
- **Absence de profilage** : aucune donnée bénéficiaire n'est utilisée pour créer des profils comportementaux à des fins autres que la détection de fraudes dans le contexte du programme
- **Décision humaine** : aucune décision significative affectant les droits ou les intérêts d'un bénéficiaire n'est prise de façon entièrement automatique sans supervision humaine
- **Transparence** : l'Organisation cliente est informée des composants IA actifs dans son programme
- **Garde des données sensibles** : 7 catégories de données sanitaires sensibles bénéficient d'une protection algorithmique renforcée

### 6.2 Détection d'anomalies

Le module de détection d'anomalies (IsolationForest) analyse les patterns d'accès et les comportements de scan QR pour détecter des fraudes potentielles. Ses résultats ont valeur d'alerte et ne constituent pas une décision définitive — toute suspicion de fraude nécessite une investigation humaine avant action corrective.

### 6.3 Déduplication MPI

Le moteur de déduplication probabiliste génère des scores de confiance permettant de rapprocher des identités potentiellement dupliquées. Une fusion d'identités ne peut être exécutée qu'après validation explicite par un utilisateur disposant des droits appropriés.

---

## Article 7 — Propriété intellectuelle

### 7.1 Droits d'Oomus

La Plateforme Oomus CampaignID, son code source, son architecture, ses algorithmes, ses modèles de cartes, ses interfaces, sa documentation et ses marques déposées sont la propriété exclusive d'Oomus et sont protégés par les droits de propriété intellectuelle applicables, notamment la Loi n° 2008-09 portant Code de la Propriété Intellectuelle du Sénégal et les conventions internationales en vigueur.

La souscription à un plan n'emporte aucun transfert de propriété intellectuelle sur la Plateforme ou ses composants.

### 7.2 Licence d'utilisation

Oomus concède à l'Organisation cliente une licence d'utilisation non exclusive, non cessible, non sous-licenciable, limitée à la durée d'abonnement et aux seuls territoires d'opération déclarés, pour accéder et utiliser la Plateforme aux seules fins prévues aux présentes CGU.

### 7.3 Données bénéficiaires — Propriété de l'Organisation cliente

Les données bénéficiaires collectées et traitées via la Plateforme restent la propriété exclusive de l'Organisation cliente. Oomus ne revendique aucun droit sur ces données et s'engage à ne les utiliser qu'aux fins définies à l'article 5.4.

### 7.4 Données d'utilisation agrégées

Oomus peut utiliser des données d'utilisation agrégées et anonymisées (métriques de performance, statistiques de génération, indicateurs de santé de la plateforme) à des fins d'amélioration de la Plateforme, sous réserve qu'aucune donnée permettant d'identifier l'Organisation cliente ou ses bénéficiaires ne soit exploitée.

### 7.5 Retours et suggestions

Toute suggestion, amélioration ou retour soumis par l'Organisation cliente concernant la Plateforme est cédé à Oomus à titre gratuit, sans contrepartie ni droit de revendication ultérieur.

---

## Article 8 — Confidentialité

### 8.1 Obligations réciproques

Chaque partie s'engage à traiter comme strictement confidentielles toutes les informations de l'autre partie qualifiées comme telles ou dont la nature confidentielle résulte du contexte, incluant sans limitation : les données bénéficiaires, les configurations de programme, les clés API, les tarifs négociés, les informations financières et les secrets techniques.

### 8.2 Exceptions

Les obligations de confidentialité ne s'appliquent pas aux informations :

- Qui sont ou deviennent publiques sans faute de la partie réceptrice
- Qui étaient déjà connues de la partie réceptrice avant leur divulgation
- Qui sont développées indépendamment sans utilisation des informations confidentielles
- Dont la divulgation est requise par une obligation légale ou réglementaire, après notification préalable de l'autre partie dans les meilleurs délais

### 8.3 Durée

Les obligations de confidentialité s'appliquent pendant la durée d'utilisation de la Plateforme et pendant une période de **5 ans** après la résiliation du contrat.

---

## Article 9 — Sécurité

### 9.1 Obligations d'Oomus

Oomus s'engage à mettre en œuvre et maintenir les mesures de sécurité techniques et organisationnelles suivantes :

- Chiffrement AES-256-GCM des données en transit et au repos pour les données sensibles
- Authentification à deux facteurs (2FA TOTP) disponible pour tous les comptes
- Protection brute-force sur tous les endpoints d'authentification
- Révocation de tokens JWT individuels et globaux (JTI blacklist)
- Scanning automatique des vulnérabilités à chaque déploiement (SAST, dependency audit)
- Détection de secrets exposés dans le code source (Gitleaks)
- Journalisation immuable et complète de toutes les actions sensibles
- Isolation réseau des services de base de données et de cache
- TLS 1.3 sur toutes les communications en production

### 9.2 Obligations de l'Organisation cliente

L'Organisation cliente s'engage à :

- Activer la 2FA pour tous les comptes administrateurs
- Utiliser des mots de passe conformes à la politique de complexité de la Plateforme
- Ne pas stocker les tokens d'API dans des emplacements non sécurisés
- Signaler toute vulnérabilité découverte à security@oomus.org dans les 48 heures
- Ne pas réaliser de tests de pénétration sur la Plateforme sans accord écrit préalable d'Oomus

### 9.3 Notification de violation de données

En cas de violation de données détectée par Oomus susceptible d'affecter les données bénéficiaires de l'Organisation cliente, Oomus s'engage à :

- Notifier l'Organisation cliente dans un délai de **72 heures** après prise de connaissance
- Fournir une description de la nature de la violation, des catégories de données concernées et du nombre estimé de bénéficiaires affectés
- Communiquer les mesures prises ou envisagées pour remédier à la violation

L'Organisation cliente reste responsable des notifications aux autorités de protection des données et aux bénéficiaires concernés, conformément à la réglementation nationale applicable.

---

## Article 10 — Facturation et paiement

### 10.1 Plans et tarification

L'accès à la Plateforme est soumis à la souscription d'un plan parmi les offres disponibles :

- **Essential** : conçu pour les programmes pilotes et de petite échelle
- **Regional Command** : conçu pour les programmes régionaux multi-districts
- **National Infrastructure** : conçu pour les programmes nationaux à fort volume
- **Sovereign Cloud** : conçu pour les infrastructures nationales souveraines et les déploiements multi-pays

Les tarifs sont communiqués sur devis institutionnel. Oomus se réserve le droit de modifier ses tarifs avec un préavis de 60 jours.

### 10.2 Quotas et dépassements

Chaque plan inclut des quotas mensuels d'identités, de communications et de vérifications. Les dépassements sont facturés au tarif unitaire défini dans le plan souscrit. L'Organisation cliente est informée par notification lorsqu'elle atteint 80 % de son quota mensuel.

### 10.3 Factures

Chaque débit génère automatiquement une facture formelle comportant un numéro unique, une signature cryptographique et un journal d'audit associé. Les factures sont téléchargeables depuis l'espace de facturation et vérifiables via l'API.

### 10.4 Paiements

Les conditions de paiement, les devises acceptées et les modalités de rechargement de compte sont définies dans l'accord commercial conclu entre Oomus et l'Organisation cliente. Les paiements sont définitifs et non remboursables sauf cas prévu à l'article 2.3 (remboursement automatique en cas d'échec de génération).

---

## Article 11 — Disponibilité et niveaux de service

### 11.1 Engagements de disponibilité

Les engagements de disponibilité varient selon le plan souscrit :

| Plan | Disponibilité cible | Fenêtre de maintenance |
|---|---|---|
| Essential | 99,5 % | Hebdomadaire (week-end) |
| Regional Command | 99,7 % | Bi-mensuelle (week-end) |
| National Infrastructure | 99,9 % | Mensuelle (nuit locale) |
| Sovereign Cloud | 99,99 % | Sur planification contractuelle |

### 11.2 Exclusions

Les engagements de disponibilité ne couvrent pas les indisponibilités dues à :

- La maintenance planifiée avec notification préalable
- Les défaillances des services tiers (DHIS2, WhatsApp, Orange SMS, Google)
- Les attaques DDoS d'ampleur exceptionnelle
- Les cas de force majeure définis à l'article 1
- L'utilisation non conforme de la Plateforme par l'Organisation cliente

### 11.3 Portail de vérification hors ligne

Le portail de vérification hors ligne, une fois déployé localement, fonctionne de manière totalement autonome sans dépendance à la disponibilité de la Plateforme centrale.

---

## Article 12 — Responsabilité

### 12.1 Limitation de responsabilité d'Oomus

Dans les limites autorisées par la loi applicable, la responsabilité totale d'Oomus au titre des présentes CGU, pour quelque cause que ce soit, est limitée au montant effectivement payé par l'Organisation cliente au cours des 12 mois précédant le fait générateur du dommage.

Oomus ne saurait être tenu responsable des dommages indirects, consécutifs, spéciaux ou punitifs, incluant sans limitation : la perte de données, la perte de revenus, l'atteinte à la réputation ou l'interruption d'activité, même si Oomus a été informé de la possibilité de tels dommages.

### 12.2 Exclusions de responsabilité d'Oomus

Oomus ne saurait être tenu responsable :

- Des actes ou omissions de l'Organisation cliente ou de ses Utilisateurs autorisés
- De l'utilisation non conforme de la Plateforme par l'Organisation cliente
- Des défaillances des services tiers intégrés (DHIS2, Meta/WhatsApp, opérateurs téléphoniques, Google)
- Des violations de données résultant d'une compromission des identifiants de l'Organisation cliente
- Des décisions prises par l'Organisation cliente sur la base des données ou analyses fournies par la Plateforme

### 12.3 Responsabilité de l'Organisation cliente

L'Organisation cliente est seule responsable :

- Du contenu des données bénéficiaires traitées via la Plateforme
- De la légalité du traitement des données au regard de la réglementation nationale applicable
- Des communications envoyées aux bénéficiaires via les services de distribution
- De la conformité du programme de santé aux réglementations sanitaires nationales
- Des conséquences pour les bénéficiaires résultant d'erreurs dans les données traitées

---

## Article 13 — Résiliation

### 13.1 Résiliation à l'initiative de l'Organisation cliente

L'Organisation cliente peut résilier son abonnement à tout moment avec un préavis de 30 jours par écrit adressé à ceo@oomus.org. Les sommes déjà versées ne sont pas remboursées, sauf disposition contraire dans l'accord commercial.

### 13.2 Résiliation à l'initiative d'Oomus

Oomus peut résilier l'accès de l'Organisation cliente dans les cas suivants :

- Non-respect des présentes CGU, notamment en cas de violation de données, d'utilisation non conforme ou de non-paiement
- Utilisation de la Plateforme à des fins illicites ou portant atteinte aux droits des bénéficiaires
- Force majeure rendant l'exécution du service impossible sur une durée supérieure à 30 jours
- Cessation d'activité d'Oomus

En cas de résiliation pour faute grave, Oomus se réserve le droit de suspendre l'accès sans préavis.

### 13.3 Effets de la résiliation

À la date d'effet de la résiliation :

- L'Organisation cliente cesse immédiatement d'utiliser la Plateforme
- Oomus conserve les données bénéficiaires pendant 90 jours pour permettre leur export
- Au terme de ce délai, les données sont supprimées définitivement
- La piste d'audit immuable est conservée conformément aux obligations légales applicables
- Les obligations de confidentialité et les articles 5, 7 et 8 demeurent en vigueur

---

## Article 14 — Modifications des CGU

Oomus se réserve le droit de modifier les présentes CGU à tout moment. Les modifications sont notifiées à l'Organisation cliente par email avec un préavis de **30 jours** avant leur entrée en vigueur.

En cas de refus des modifications, l'Organisation cliente peut résilier son abonnement sans pénalité dans les 30 jours suivant la notification, avec remboursement prorata temporis des sommes prépayées correspondant à la période non consommée.

La poursuite de l'utilisation de la Plateforme après l'entrée en vigueur des modifications vaut acceptation de celles-ci.

---

## Article 15 — Droit applicable et règlement des litiges

### 15.1 Droit applicable

Les présentes CGU sont régies par le **droit sénégalais**, sans préjudice des dispositions impératives de protection applicables dans le pays d'établissement de l'Organisation cliente.

Pour les Organisations clientes établies dans des pays membres de l'UEMOA, les actes additionnels de l'UEMOA en matière de données personnelles et de cybersécurité s'appliquent de manière complémentaire.

### 15.2 Règlement amiable

En cas de différend relatif à l'interprétation, l'exécution ou la résiliation des présentes CGU, les parties s'engagent à rechercher une solution amiable dans un délai de **60 jours** à compter de la notification du différend par la partie lésée.

### 15.3 Médiation

À défaut de règlement amiable dans le délai prévu, les parties peuvent soumettre le différend à la médiation de la **Chambre Arbitrale de Dakar** ou de tout autre organisme de médiation agréé accepté par les deux parties.

### 15.4 Juridiction compétente

À défaut de règlement amiable ou de médiation aboutie, tout litige est soumis à la compétence exclusive des tribunaux du **Tribunal de Grande Instance Hors Classe de Dakar**, République du Sénégal.

Pour les Organisations clientes relevant d'un statut diplomatique ou d'une immunité de juridiction, des modalités de règlement des litiges spécifiques sont définies dans l'accord bilatéral conclu avec Oomus.

---

## Article 16 — Dispositions générales

### 16.1 Intégralité

Les présentes CGU, complétées le cas échéant par l'accord commercial, le SLA et le DPA, constituent l'intégralité de l'accord entre les parties concernant l'utilisation de la Plateforme et remplacent tout accord antérieur, oral ou écrit.

### 16.2 Divisibilité

Si une disposition des présentes CGU est déclarée nulle ou inapplicable, elle est réputée non écrite sans affecter la validité des autres dispositions.

### 16.3 Non-renonciation

Le fait pour Oomus de ne pas se prévaloir d'un manquement de l'Organisation cliente ne constitue pas une renonciation à se prévaloir de manquements ultérieurs.

### 16.4 Cession

L'Organisation cliente ne peut céder tout ou partie de ses droits et obligations au titre des présentes CGU sans l'accord écrit préalable d'Oomus. Oomus peut céder les présentes CGU dans le cadre d'une fusion, acquisition ou cession de la totalité ou d'une partie substantielle de ses actifs, sous réserve d'en informer l'Organisation cliente.

### 16.5 Force majeure

Aucune partie n'est responsable des manquements à ses obligations contractuelles résultant d'un cas de force majeure. La partie empêchée notifie l'autre dans les 48 heures et met tout en œuvre pour limiter les effets et reprendre l'exécution dans les meilleurs délais.

---

## Article 17 — Contact et signalement

| Objet | Contact |
|---|---|
| Questions générales | ceo@oomus.org |
| Support technique | Via l'espace client de la Plateforme |
| Protection des données | ceo@oomus.org — objet : « DPO — [sujet] » |
| Signalement de vulnérabilité | ceo@oomus.org — objet : « Security Disclosure » |
| Abus / utilisation non conforme | ceo@oomus.org — objet : « Report Abuse » |
| Demandes légales / judiciaires | ceo@oomus.org — objet : « Legal Request » |

**Oomus**  
Dakar, Sénégal  
+221 77 803 91 91  
ceo@oomus.org  

---

*Dernière mise à jour : 1er juin 2026 — Version 1.0*  
*Ces CGU remplacent toutes les versions précédentes.*

> **Oomus CampaignID** — *Une identité. Un citoyen. Un système de santé souverain.*
