# Conformité légale

Cette page présente le cadre de conformité légale d'Oomus CampaignID et les mesures en place pour respecter les réglementations applicables en matière de données personnelles de santé.

---

## Cadres réglementaires applicables

### Principes RGPD

Oomus CampaignID applique les **principes du Règlement Général sur la Protection des Données (RGPD)** de l'Union Européenne comme référentiel de bonne pratique, même pour les déploiements hors UE. Ces principes représentent le standard le plus exigeant en matière de protection des données personnelles.

### Réglementations nationales — Afrique de l'Ouest

Les programmes déployés dans les pays de la CEDEAO doivent respecter les législations nationales applicables. Les principaux cadres de référence incluent :

| Pays | Loi / Autorité | Référence |
|---|---|---|
| **Sénégal** | Loi n°2008-12 sur la protection des données personnelles | Commission de Protection des Données Personnelles (CDP) |
| **Côte d'Ivoire** | Loi n°2013-450 relative à la protection des données personnelles | Autorité Nationale de Régulation des TIC (ARTCI) |
| **Mali** | Loi n°2013-015 portant protection des données personnelles | Commission Nationale de l'Informatique et des Libertés (CNIL Mali) |
| **Burkina Faso** | Loi n°010-2004/AN | |
| **Guinée** | Code numérique de la République de Guinée | Autorité de Régulation des Postes et Télécommunications |

> Il appartient à l'organisation déployant Oomus CampaignID de s'assurer de la conformité avec la législation nationale applicable. L'équipe Oomus peut accompagner cette démarche sur demande.

### Convention de l'Union Africaine sur la Cybersécurité et la Protection des Données (Convention de Malabo)

La **Convention de Malabo** (2014) établit un cadre continental africain pour la protection des données personnelles, incluant les données de santé. Oomus CampaignID est conçu pour être compatible avec les exigences de cette convention.

---

## Périmètre de conformité

### Minimisation des données

Oomus CampaignID collecte **uniquement les données nécessaires** à la génération, distribution et vérification des cartes de santé :

- Nom, prénom, date de naissance, sexe (pour la déduplication MPI)
- Numéro de téléphone (si distribution SMS/WhatsApp)
- Données de programme (type de carte, campagne, date d'émission)

Les dossiers médicaux complets, données biométriques brutes et données financières personnelles ne sont pas collectés.

### Limitation de la finalité

Les données collectées servent exclusivement à :
1. Générer les cartes de santé numériques
2. Distribuer les cartes aux bénéficiaires
3. Permettre la vérification d'authenticité des cartes
4. Construire et maintenir le registre d'identité MPI souverain

Elles ne sont pas utilisées à des fins commerciales, de profilage ou de ciblage publicitaire.

### Contrôle d'accès

L'accès aux données bénéficiaires est contrôlé par le système RBAC institutionnel :
- Seuls les utilisateurs autorisés du programme peuvent accéder aux données
- L'accès est limité au périmètre géographique et fonctionnel défini
- Toute action est tracée dans la piste d'audit immuable

### Sécurité des données

Les données sont protégées par :
- Chiffrement AES-256-GCM pour les données sensibles au repos
- HTTPS/TLS pour toutes les communications
- Hachage bcrypt pour les mots de passe
- Jetons QR opaques non réversibles

---

## Droits des personnes concernées

### Droit d'accès

Un bénéficiaire peut demander au programme de santé responsable l'accès aux données le concernant. Le responsable programme peut utiliser l'interface MPI pour exporter les données d'un bénéficiaire.

### Droit de rectification

Les données d'un bénéficiaire peuvent être corrigées via l'interface MPI (section "Registre MPI") ou via l'API `PATCH /mpi/{mpi_id}`. Les corrections sont enregistrées dans la piste d'audit.

### Droit à l'oubli

La suppression d'une identité MPI peut être demandée via le support Oomus. La désactivation de l'identité est alors effectuée, les cartes associées sont révoquées.

### Droit d'opposition

Un bénéficiaire peut s'opposer à la génération d'une carte ou demander sa révocation via le programme de santé responsable.

---

## Responsabilités — Modèle de responsabilité partagée

### Oomus CampaignID (sous-traitant)

En tant que **sous-traitant des données**, Oomus CampaignID s'engage à :
- Traiter les données uniquement selon les instructions du programme client
- Mettre en place les mesures techniques et organisationnelles appropriées
- Notifier le programme client dans les 72 heures en cas d'incident de sécurité
- Permettre et faciliter les audits de conformité
- Ne pas sous-traiter le traitement des données sans accord du programme client

### Organisation cliente (responsable de traitement)

En tant que **responsable du traitement**, le programme de santé client est responsable de :
- Obtenir le consentement légal des bénéficiaires selon la législation locale applicable
- Définir les finalités et la base légale du traitement
- Informer les bénéficiaires de leurs droits
- Respecter les obligations déclaratives auprès des autorités locales de protection des données
- Définir les durées de rétention des données conformément à la réglementation locale

---

## Notification de violation de données

En cas de violation de sécurité affectant des données personnelles :

1. **Détection** : Oomus CampaignID notifie le programme client dans les **72 heures** suivant la prise de connaissance
2. **Contenu de la notification** : nature de la violation, données concernées, mesures prises, point de contact
3. **Notification aux autorités** : il appartient au programme client de notifier l'autorité de protection des données compétente si requis
4. **Information des bénéficiaires** : si la violation présente un risque élevé, le programme client informe les bénéficiaires concernés

---

## Divulgation responsable

Si vous découvrez une vulnérabilité de sécurité dans Oomus CampaignID :

1. **Ne pas exploiter** la vulnérabilité
2. **Contacter** l'équipe sécurité : `security@oomus.health`
3. **Décrire** la vulnérabilité avec suffisamment de détails pour reproduction
4. **Accorder** un délai raisonnable de correction (90 jours)

Nous nous engageons à :
- Accuser réception dans les **48 heures**
- Qualifier la vulnérabilité dans les **7 jours**
- Corriger les vulnérabilités critiques dans les **30 jours**
- Vous informer de la correction et (si vous le souhaitez) vous mentionner dans nos notes de remerciement

---

## Accord de traitement des données (DPA)

Pour les programmes nécessitant un **Data Processing Agreement (DPA)** formalisé — notamment pour les déploiements impliquant des ressortissants de l'UE ou des partenaires exigeant une conformité RGPD — contactez l'équipe Oomus à **legal@oomus.health**.

---

## Prochaines étapes

- [IA & Éthique](ai-ethics.md)
- [Protection des données](../security/data-protection.md)
- [Vue d'ensemble sécurité](../security/overview.md)
