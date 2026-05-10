# Testing Strategy

## Tests a couvrir

- Authentification et renouvellement JWT.
- Autorisations utilisateur/admin.
- Creation et listing de campagnes.
- Upload et parsing de templates.
- Creation de jobs.
- Verification publique de codes.
- Calculs de facturation.
- Generation et stockage des artefacts.
- Synchronisation DHIS2 avec mapping de champs.
- Emission Google Wallet.
- Envoi WhatsApp/SMS avec mocks fournisseurs.
- Verification offline WebCrypto.
- Hash chain SHA-256.
- Detection d'anomalies IsolationForest sur codes techniques.

## Types de tests

- Tests unitaires pour fonctions de securite, schemas et calculs.
- Tests d'integration API avec base de test.
- Tests worker avec Redis/Celery en mode test.
- Tests end-to-end sur le flux campagne vers generation.

## Donnees de test

- Compte demo client.
- Compte admin.
- Template YAML minimal.
- Template JSON minimal.
- Campagne avec faible volume.
- Registre offline.
- Dataset DHIS2 factice.
- Codes avec collision simulee.

## Non-regression v3.5

- Les QR codes ne contiennent pas de donnees medicales sensibles.
- Les credentials fournisseurs ne sont jamais exposes dans les reponses API.
- Les erreurs fournisseurs sont journalisees sans fuite de secret.
- Les jobs longs ne bloquent pas l'API HTTP.
