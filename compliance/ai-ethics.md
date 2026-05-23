# IA & Éthique

Oomus CampaignID intègre des composants d'intelligence artificielle pour améliorer la qualité du service. Cette page décrit de manière transparente l'usage de l'IA, ses limites et les garanties éthiques en place.

---

## Usages de l'IA dans Oomus CampaignID

Oomus CampaignID utilise l'intelligence artificielle dans **deux contextes précis et circonscrits** :

### 1. Détection d'anomalies — IsolationForest

**Composant** : Module premium "AI Fraud Detection"

**Algorithme** : IsolationForest (algorithme d'apprentissage non supervisé pour la détection d'anomalies)

**Ce qu'il analyse** :
- Patterns de scan de cartes (fréquences, volumes, horaires)
- Comportements de distribution atypiques
- Activités de compte inhabituelles (volumes de génération, requêtes API)

**Ce qu'il n'analyse PAS** :
- Données personnelles des bénéficiaires (nom, DDN, santé)
- Contenu des cartes générées
- Données médicales ou cliniques

**Résultat** : Signalement d'anomalies comportementales pour investigation par un humain. Aucune décision automatique n'est prise.

---

### 2. Déduplication MPI — Moteur probabiliste

**Composant** : MPI Sovereign Identity Engine (inclus dans tous les plans)

**Algorithme** : Calcul de score de similarité composite (Jaro-Winkler + normalisation africaine + correspondance sémantique)

**Ce qu'il analyse** :
- Prénom, nom (similarité phonétique et orthographique)
- Date de naissance (correspondance exacte ou proche)
- Sexe, district géographique
- Identifiants externes (DHIS2 UID)

**Ce qu'il n'est PAS** :
- Un système biométrique (pas de reconnaissance faciale, empreintes digitales, iris)
- Un système de profilage (il ne "profite" pas du bénéficiaire)
- Un système d'identification prédictive (il ne prédit pas les comportements)

**Résultat** :
- Score ≥ 0.95 → liaison automatique (haute certitude)
- Score 0.75–0.94 → **révision humaine obligatoire** avant toute fusion
- Score < 0.75 → création d'une nouvelle identité

---

## Garanties éthiques

### Aucun profilage des bénéficiaires

Oomus CampaignID ne construit pas de profils comportementaux, socio-démographiques ou médicaux des bénéficiaires à des fins commerciales ou analytiques. Les données collectées servent uniquement à l'émission et la vérification des cartes.

### Aucune décision automatique affectant les bénéficiaires

Conformément aux principes RGPD (article 22) et aux meilleures pratiques éthiques, **aucune décision significative affectant directement un bénéficiaire n'est prise automatiquement** par un algorithme d'Oomus CampaignID :

- La fusion de deux identités MPI nécessite toujours une validation humaine (si score < 0.95)
- La révocation d'une carte est toujours une action humaine intentionnelle
- L'accès aux soins ne dépend jamais d'une décision algorithmique d'Oomus

### Surveillance humaine systématique

Toutes les sorties des systèmes IA d'Oomus sont des **recommandations** destinées à être examinées par un opérateur humain :

| Sortie IA | Niveau de surveillance humaine |
|---|---|
| Anomalie de scan détectée | Alerte → investigation humaine requise |
| Match MPI probable (score 0.75–0.94) | Queue de révision → décision humaine obligatoire |
| Match MPI certain (score ≥ 0.95) | Liaison automatique autorisée (seuil calibré) |

### Garde des données sensibles

Avant tout traitement analytique ou entraînement de modèle, un système de **garde automatique** exclut les attributs appartenant à 7 catégories sensibles :

| Catégorie | Pourquoi exclue |
|---|---|
| HIV/SIDA | Données hautement stigmatisantes |
| IST | Données hautement stigmatisantes |
| Tuberculose | Données de santé sensibles |
| Santé mentale | Données de santé sensibles, risque de discrimination |
| Sérologique | Données biologiques sensibles |
| Biométrique | Données irremplaçables, risque de sécurité permanent |
| Financier | Données socio-économiques, risque de discrimination |

---

## Biais algorithmiques — Mesures prises

Le moteur de déduplication MPI a été spécialement conçu pour réduire les biais algorithmiques liés aux noms d'Afrique de l'Ouest :

### Problème identifié

Les algorithmes de similarité de texte standards (Levenshtein simple, soundex anglais) performent très mal sur les noms africains :
- Variantes orthographiques multiples : Ibrahim / Ibrahima / Brahim / Braima
- Inversion prénom/nom culturellement variable
- Noms composés fréquents
- Translitérations du Wolof, Pulaar, Mandingue

### Solution implémentée

Le moteur MPI intègre une **couche de normalisation spécialisée** :
- Dictionnaire de variantes de noms d'Afrique de l'Ouest
- Détection et traitement des noms composés
- Robustesse aux diacritiques et apostrophes
- Tests de régression sur des datasets anonymisés représentatifs

---

## Transparence algorithmique

Oomus CampaignID s'engage à documenter de manière transparente les algorithmes utilisés et leurs limites :

- Les algorithmes utilisés sont des méthodes établies et auditables (Jaro-Winkler, IsolationForest)
- Les seuils de décision sont documentés et configurables par les administrateurs
- Les résultats des algorithmes sont explicables (les champs ayant contribué au score sont indiqués)

---

## Feuille de route éthique

Oomus CampaignID s'engage à maintenir et améliorer ses garanties éthiques :

- **Audits réguliers** des biais algorithmiques du moteur de déduplication
- **Consultation** des communautés d'utilisateurs finaux (agents de santé, bénéficiaires) sur les usages IA
- **Publication** des métriques de performance des algorithmes (précision, rappel, faux positifs)
- **Veille réglementaire** sur l'IA Act européen et les équivalents africains

---

## Contact

Pour toute question sur l'usage de l'IA dans Oomus CampaignID :

- **E-mail** : ceo@oomus.org
- **Documentation** : cette page est mise à jour à chaque évolution significative des composants IA

---

## Prochaines étapes

- [Conformité légale](legal-compliance.md)
- [Protection des données](../security/data-protection.md)
- [Identité MPI souveraine](../features/mpi-sovereign-identity.md)
