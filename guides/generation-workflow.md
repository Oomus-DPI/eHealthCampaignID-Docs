# Generation Workflow - Secure Cards v3.5

## Sequence

1. L'utilisateur cree une campagne.
2. L'utilisateur upload un template ou configure le Card Studio.
3. L'API valide et stocke le template.
4. La campagne peut synchroniser des beneficiaires depuis DHIS2 Tracker.
5. L'utilisateur lance la generation.
6. L'API cree un `generation_job`.
7. Celery prend en charge le job.
8. Le worker genere les identifiants uniques via CSPRNG.
9. Le worker produit les QR codes securises.
10. Le worker calcule les empreintes SHA-256 et la hash chain d'audit.
11. Le worker lance la detection d'anomalies technique.
12. Les fichiers sont sauvegardes dans MinIO/S3 ou en local.
13. Le job passe a `completed`.
14. L'utilisateur telecharge les fichiers depuis l'API.
15. Les cartes peuvent etre distribuees via WhatsApp, SMS ou Google Wallet.

## Artefacts produits

- `cards.pdf`: cartes digitales.
- `qrcodes/`: QR codes securises.
- `cards.zip`: archive des cartes et QR codes.
- `register.csv`: registre des codes.
- `verification-register.json`: registre de verification offline.
- `audit-report.pdf` ou `report.txt`: rapport de generation.
- `hash-chain.json`: elements blockchain SHA-256.
- `stats.json`: statistiques de generation.

## Proprietes garanties

| Propriete | Mecanisme |
| --- | --- |
| Unicite | CSPRNG |
| Integrite | AES-256-GCM et SHA-256 |
| Auditabilite | Hash chain SHA-256 |
| Verification offline | WebCrypto + registre exporte |
| Resistance falsification | QR codes signes ou chiffres |

## Reprise sur erreur

En cas d'echec, le job passe a `failed` avec un message d'erreur. Un endpoint de retry controle peut etre ajoute pour relancer une generation sans perdre l'audit trail.
