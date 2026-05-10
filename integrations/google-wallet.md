# Google Wallet Integration

## Objectif

Permettre l'emission de cartes digitales compatibles Google Wallet pour faciliter la distribution et l'usage terrain.

## Fonctionnalites

- Creation de classes Wallet.
- Creation de cartes digitales.
- Generation de JWT signes.
- Distribution individuelle.
- Distribution bulk.
- QR code integre.
- Ajout rapide au Wallet.

## Capacites

- Jusqu'a 100 cartes par JWT selon les contraintes d'usage.
- Integration avec les campagnes et registres de verification.
- Tracabilite des liens generes.

## Securite

- Les cles de signature doivent etre conservees hors depot Git.
- Les payloads Wallet doivent rester minimises.
- Les liens de distribution doivent etre journalises par campagne.
