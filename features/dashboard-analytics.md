# Dashboard & Analytics IA

Le Dashboard d'Oomus CampaignID est le centre de pilotage opérationnel de votre programme de santé. Il offre une vue en temps réel de toutes les activités, métriques, alertes et **prédictions d'intelligence artificielle** de votre infrastructure.

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

## Graphique d'activité — 12 mois

Le graphique principal affiche l'évolution sur les 12 derniers mois de trois séries :

- **Jobs de génération** : nombre de jobs lancés par mois
- **Cartes générées** : volume mensuel de cartes produites
- **Synchronisations DHIS2** : nombre de synchronisations DHIS2 effectuées par mois

Le graphique permet d'identifier les pics d'activité (campagnes saisonnières, pointes de distribution) et de planifier les capacités futures.

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

---

## Analytics IA & Prédictions (v5.0)

Le module Analytics Avancées intègre un moteur d'intelligence artificielle entièrement client-side, sans dépendance backend supplémentaire.

### Prédiction par régression linéaire

Le système calcule une projection des 30 prochains jours de génération de cartes à partir de l'historique réel :

- Algorithme : régression linéaire O(n) par moindres carrés (least squares)
- Visualisation : `ForecastChart` — données réelles en cyan solide, prédictions en indigo pointillé
- Séparateur `ReferenceLine` entre historique et projection
- Badge **R²** affiché sur le graphique pour indiquer la qualité du modèle

### Détection d'anomalies Z-score

Les points anormaux dans les séries temporelles de génération sont automatiquement détectés :

- Seuil : **2,2 écarts-types** (σ) — équilibre sensibilité / faux positifs
- Affichage : outliers mis en évidence en rouge sur le graphique
- Déclencheur d'alerte dans le panneau IA si plus de 2 anomalies sont détectées sur la période

### Score de santé IA (0–100)

Un indicateur composite synthétise la santé de votre programme en un seul chiffre :

| Composante | Poids | Calcul |
| ---------- | ----- | ------ |
| Taux de succès des jobs | 40% | `success_rate / 100 * 40` |
| Pente de tendance (génération) | 30% | Normalisée entre 0 et 30 |
| Score de balance | 30% | Logarithmique sur le solde disponible |

Jauge SVG semicirculaire (arc stroke-dasharray) avec couleur dynamique :

- Vert (`#10B981`) : score ≥ 70
- Ambre (`#F59E0B`) : score 40–69
- Rouge (`#EF4444`) : score < 40

### Projection des dépenses

- **Burn rate** : moyenne mobile sur 7 jours glissants
- **Projection 30 jours** : extrapolation linéaire du solde
- Carte d'alerte si la projection est négative (solde insuffisant)
- Graphique `AreaChart` burn rate + projection

### Panneau de recommandations IA

Des recommandations contextuelles sont générées automatiquement à partir des métriques réelles :

| Niveau | Icône | Exemple de recommandation |
| ------ | ----- | ------------------------- |
| Warning | `AlertTriangle` | "Projection solde négatif dans 18 jours — envisager un rechargement" |
| Info | `Info` | "Tendance forte détectée — envisager une montée en plan" |
| Success | `CheckCircle2` | "Score de santé excellent — infrastructure optimale" |

Les recommandations s'adaptent dynamiquement : elles disparaissent dès que la condition sous-jacente est résolue.

---

## Prochaines étapes

- [Gestion des campagnes](campaigns.md) — Voir le détail d'une campagne
- [Moteur de simulation](simulation-engine.md) — Planifier un déploiement
- [Plans & Tarification](../getting-started/plans-and-pricing.md) — Module Analytics avancées
