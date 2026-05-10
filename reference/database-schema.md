# Database Schema

## Tables principales

| Table | Description |
| --- | --- |
| `users` | Comptes, role, plan, solde |
| `programmes` | Programmes clients ou institutionnels |
| `campaigns` | Campagnes de generation |
| `generation_jobs` | Jobs Celery et progression |
| `vouchers` | Codes uniques generes |
| `transactions` | Credits, debits et ajustements |
| `pricing_plans` | Tarification par plan |
| `dhis2_connections` | Connexions DHIS2 chiffrees |
| `dhis2_sync_jobs` | Jobs de synchronisation |
| `message_deliveries` | Traces WhatsApp/SMS |
| `wallet_passes` | Cartes Google Wallet |
| `audit_hashes` | Empreintes SHA-256 et hash chain |

## Relations

- Un utilisateur possede plusieurs campagnes.
- Une campagne appartient optionnellement a un programme.
- Une campagne possede plusieurs jobs et vouchers.
- Une transaction appartient a un utilisateur et peut etre liee a un job.
- Une connexion DHIS2 appartient a un tenant ou programme.
- Une distribution message appartient a une campagne et a un voucher.
- Un wallet pass appartient a un voucher.
- Une empreinte d'audit appartient a une generation.

## Etats

Campaign:

- `draft`
- `ready`
- `generating`
- `completed`
- `failed`

Job:

- `pending`
- `running`
- `completed`
- `failed`
