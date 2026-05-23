# Dashboard & Analytics

Le Dashboard d'Oomus CampaignID est le centre de pilotage opérationnel de votre programme de santé. Il offre une vue en temps réel de toutes les activités, métriques et alertes de votre infrastructure.

---

## Centre de pilotage opérationnel

### Actualisation automatique

Le tableau de bord se rafraîchit automatiquement **toutes les 30 secondes** pour afficher les données les plus récentes. L'indicateur de fraîcheur des données est visible en haut de la page.

---

## Cartes KPI principales

Six indicateurs clés sont affichés en permanence en haut du tableau de bord :

| KPI | Description | Unité |
|---|---|---|
| **Solde** | Crédit disponible sur votre compte | FCFA |
| **Cartes générées** | Total des cartes produites (tous temps) | Nombre |
| **Jobs actifs** | Jobs de génération en cours d'exécution | Nombre |
| **Taux de succès** | Pourcentage de jobs terminés avec succès | % |
| **Campagnes** | Nombre total de campagnes (actives + archivées) | Nombre |
| **Quota restant** | Bénéficiaires restants dans le quota mensuel | Nombre |

---

## Graphique d'activité — Jan–Déc (année en cours)

Le graphique principal affiche l'activité mensuelle de janvier à décembre de l'année en cours. Il comporte **quatre séries individuellement activables/désactivables** via les boutons de sélection :

| Série | Type | Axe | Couleur |
| --- | --- | --- | --- |
| **Cartes** | Aire (Area) | Gauche (volume) | Bleu |
| **Studio** | Barres empilées | Gauche (volume) | Violet |
| **DHIS2** | Barres empilées | Gauche (volume) | Teal |
| **Succès %** | Ligne pointillée | Droite (0–100 %) | Vert |

Cliquez sur chaque pill-bouton pour afficher ou masquer la série correspondante. Toutes les séries sont actives par défaut.

Les mois futurs affichent zéro et le graphique reste visible même si aucune activité n'a encore été enregistrée.

Le graphique permet d'identifier les pics d'activité (campagnes saisonnières, pointes de distribution) et de suivre la fiabilité opérationnelle (taux de succès mensuel).

---

## Barres de consommation de quota

Les jauges de quota affichent en temps réel :

| Ressource | Affichage |
|---|---|
| **Bénéficiaires** | X utilisés sur Y alloués (%) |
| **SMS** | X envoyés sur Y inclus (%) |
| **WhatsApp** | X envoyés sur Y inclus (%) |
| **Stockage** | X Go utilisés sur Y Go (%) |

Une alerte visuelle apparaît lorsque la consommation dépasse 80% du quota.

---

## Score de santé de l'infrastructure

Un indicateur synthétique (0–100) agrège la santé de tous les composants de la plateforme :

| Composant surveillé | Indicateurs |
|---|---|
| API Backend | Latence, taux d'erreur, disponibilité |
| Workers Celery | Files de tâches, jobs en attente, workers actifs |
| Base de données | Connexions, temps de requête |
| Cache Redis | Utilisation mémoire, hit rate |
| Stockage MinIO/S3 | Disponibilité, espace |
| Intégration DHIS2 | Dernière sync réussie, latence |

---

## Panneau d'alertes

Les alertes sont classées par priorité :

| Priorité | Exemples |
|---|---|
| **Critique** | Infrastructure indisponible, job bloqué, quota épuisé |
| **Haute** | Quota > 90% consommé, échec sync DHIS2, job en erreur |
| **Moyenne** | Quota > 80%, latence élevée, certificat expirant |
| **Info** | Job terminé, sync réussie, nouveau bénéficiaire MPI |

---

## Flux d'activité en direct

Le panneau "Activité récente" affiche en temps réel les derniers événements :

- Cartes générées (avec nom de la campagne et nombre)
- Jobs démarrés / terminés
- Synchronisations DHIS2 (enrollments récupérés)
- Cartes distribuées (WhatsApp / SMS / Google Wallet)
- Connexions utilisateur
- Modifications de configuration

---

## Section gouvernance

La section Gouvernance du dashboard affiche :

- **Approbations en attente** : simulations et demandes nécessitant une action admin
- **Audit log récent** : dernières actions enregistrées (acteur, action, ressource, IP)
- **Accès utilisateurs actifs** : liste des utilisateurs connectés et leurs rôles
- **Demandes de rechargement** : demandes de quota en attente de traitement

---

## Analytics avancées (module premium)

Le module Analytics avancées est disponible sur les plans National Campaign et Sovereign Enterprise.

### Onglet Résumé

- Récapitulatif global : bénéficiaires uniques, cartes totales, taux de distribution
- Comparaison période précédente
- Top 3 des campagnes les plus actives

### Onglet Chronologie

Visualisation temporelle granulaire (jour / semaine / mois) de :
- Générations de cartes
- Distributions par canal
- Synchronisations DHIS2

### Onglet Dépenses dans le temps

Évolution mensuelle du budget consommé, avec décomposition par :
- Génération de cartes (par DPI)
- Distribution SMS
- Distribution WhatsApp
- Stockage
- Modules premium

### Onglet Types de campagne

Répartition des activités par type de programme :
- Vaccination, MILD, Nutrition, HIV, Assurance, etc.
- Volume de cartes par type
- Taux de distribution par type

### Onglet Top campagnes

Classement des campagnes les plus actives par :
- Nombre de bénéficiaires
- Volume de cartes générées
- Taux de distribution
- Succès de vérification

---

## Export des données analytiques

Les données du dashboard peuvent être exportées :

| Format | Contenu |
|---|---|
| **CSV** | Données brutes filtrées (date, type, volume) |
| **Excel** | Rapport formaté multi-onglets |
| **JSON** | Données structurées pour intégration BI |

---

## Prochaines étapes

- [Gestion des campagnes](campaigns.md) — Voir le détail d'une campagne
- [Moteur de simulation](simulation-engine.md) — Planifier un déploiement
- [Plans & Tarification](../getting-started/plans-and-pricing.md) — Module Analytics avancées
