# Cryptographic Security

## Technologies

- AES-256-GCM.
- SHA-256.
- Base36.
- JWT RS256 en cible production.
- WebCrypto pour verification offline.

## Proprietes garanties

| Propriete | Mecanisme |
| --- | --- |
| Unicite | Generation CSPRNG |
| Integrite | AES-256-GCM |
| Verification | SHA-256 |
| Auditabilite | Blockchain hash chain |
| Verification offline | WebCrypto |

## Hash chain

Chaque lot de generation peut produire une chaine d'empreintes SHA-256 reliant les codes, les artefacts et les rapports. Cette chaine sert de preuve technique d'integrite et d'auditabilite.

## Recommandations production

- Rotation des cles.
- Secret manager.
- Separation des cles par environnement.
- Tests de collision.
- Export d'audit par campagne.
