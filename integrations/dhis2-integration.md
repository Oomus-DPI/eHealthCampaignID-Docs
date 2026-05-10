# DHIS2 Tracker Integration

## Objectif

Transformer les donnees operationnelles deja presentes dans DHIS2 Tracker en identites digitales exploitables sur le terrain, sans imposer de refonte des systemes nationaux existants.

## Valeur operationnelle

- Eviter les doubles saisies.
- Limiter les erreurs humaines.
- Ameliorer la distribution des services.
- Renforcer la tracabilite des beneficiaires.
- Accelerer les campagnes.
- Reduire les couts operationnels.

## Fonctionnalites

- Connexion a DHIS2.
- Synchronisation automatique.
- Assignation de codes.
- Generation de cartes.
- Previsualisation.
- Distribution.
- Analytics.

## Synchronisation automatique

La synchronisation s'appuie sur:

- Celery Beat.
- Taches asynchrones.
- Cron configurable.
- Monitoring des synchronisations.
- Historique des erreurs.

## Donnees et mapping

Chaque campagne doit definir:

- Instance DHIS2 cible.
- Programme Tracker.
- Organisation unit.
- Champs beneficiaires a importer.
- Champs a afficher sur la carte.
- Champs exclus pour raison de confidentialite.

## Regles de securite

- Les credentials DHIS2 doivent etre chiffres.
- Les donnees medicales sensibles doivent etre exclues des QR codes.
- Chaque synchronisation doit etre auditable.
- Les erreurs doivent etre journalisees sans exposer de donnees sensibles.
