# Messaging Integration

## WhatsApp Cloud API

Fonctionnalites:

- Envoi de messages.
- Envoi de templates.
- Envoi bulk.
- Tests integres.
- Logs temps reel.

## Orange SMS API

Fonctionnalites:

- OAuth2 Client Credentials.
- Envoi SMS unitaire.
- Envoi bulk.
- Cache token.
- Chiffrement des credentials.

## Contraintes

- Respecter les opt-ins et politiques fournisseurs.
- Suivre les couts par campagne.
- Prevoir retries et files d'attente.
- Ne jamais exposer les tokens dans le frontend.

## Observabilite

Chaque envoi doit conserver:

- Campagne.
- Canal.
- Statut.
- Horodatage.
- Fournisseur.
- Message d'erreur technique si applicable.
