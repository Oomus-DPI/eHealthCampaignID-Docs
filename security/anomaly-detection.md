# AI Anomaly Detection

## Objectif

Renforcer la qualite et la fiabilite technique des campagnes par detection d'anomalies sur les codes generes.

La plateforme n'utilise pas l'IA pour produire des decisions medicales automatisees.

## Moteur

Le moteur principal repose sur IsolationForest de scikit-learn.

## Cas detectes

- Collisions potentielles.
- Irregularites de generation.
- Comportements atypiques.
- Risques de falsification.
- Anomalies de qualite.

## Donnees utilisees

- Ordinals ASCII des codes.
- Metadonnees techniques des codes.

## Donnees exclues

- Donnees personnelles.
- Donnees medicales.
- Donnees cliniques.
- Donnees biometriques.

## Garanties

- Pas de profilage.
- Pas de scoring medical.
- Transparence des scores.
- Decision humaine conservee.
