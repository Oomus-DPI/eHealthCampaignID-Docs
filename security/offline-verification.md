# Offline Verification

## Objectif

Permettre la verification publique d'une carte meme en situation de connectivite intermittente.

## Fonctionnalites

- Verification manuelle.
- Scan QR.
- Verification bulk.
- Export CSV.
- Statistiques temps reel lorsque la connectivite revient.
- Fonctionnement hors ligne.

## Technologies

- WebCrypto.
- SHA-256.
- BarcodeDetector API.
- HTML statique.
- Registre de verification exportable.

## Principe

Le portail offline embarque les elements minimaux necessaires pour verifier l'authenticite d'un code sans interroger l'API en continu.

## Confidentialite

- Aucune donnee medicale sensible dans le QR code.
- Payload minimal.
- Empreintes et signatures privilegiees.
- Donnees nominatives evitees quand elles ne sont pas strictement necessaires.
