# API Reference - v3.5

## Auth

| Methode | Route | Description |
| --- | --- | --- |
| POST | `/api/v1/auth/register` | Creer un compte |
| POST | `/api/v1/auth/login` | Connexion JWT |
| POST | `/api/v1/auth/refresh` | Renouveler un token |
| GET | `/api/v1/auth/me` | Profil connecte |

## Campaigns

| Methode | Route | Description |
| --- | --- | --- |
| GET | `/api/v1/campaigns` | Lister les campagnes |
| POST | `/api/v1/campaigns` | Creer une campagne |
| POST | `/api/v1/campaigns/{id}/template` | Uploader un template |
| GET | `/api/v1/campaigns/{id}/template/preview` | Voir la preview du template |

## Jobs

| Methode | Route | Description |
| --- | --- | --- |
| POST | `/api/v1/jobs/campaigns/{id}/generate` | Lancer une generation |
| GET | `/api/v1/jobs` | Lister les jobs |
| GET | `/api/v1/jobs/{id}` | Detail et URLs |
| WS | `/api/v1/jobs/ws/{id}/progress?token=<jwt>` | Progression temps reel |

## Studio

| Methode | Route | Description |
| --- | --- | --- |
| POST | `/api/v1/studio/preview` | Previsualiser une carte |
| POST | `/api/v1/studio/export` | Exporter un template JSON |
| POST | `/api/v1/studio/render/png` | Generer un apercu PNG |

## DHIS2

| Methode | Route | Description |
| --- | --- | --- |
| POST | `/api/v1/dhis2/connections` | Configurer une connexion DHIS2 |
| GET | `/api/v1/dhis2/programs` | Lister les programmes DHIS2 |
| POST | `/api/v1/dhis2/sync` | Lancer une synchronisation |
| GET | `/api/v1/dhis2/jobs` | Historique des synchronisations |

## Messaging

| Methode | Route | Description |
| --- | --- | --- |
| POST | `/api/v1/sms/send` | Envoyer un SMS |
| POST | `/api/v1/sms/bulk` | Envoyer des SMS en lot |
| POST | `/api/v1/whatsapp/send` | Envoyer un message WhatsApp |
| POST | `/api/v1/whatsapp/bulk` | Envoyer en lot via WhatsApp |

## Google Wallet

| Methode | Route | Description |
| --- | --- | --- |
| POST | `/api/v1/wallet/classes` | Creer une classe Wallet |
| POST | `/api/v1/wallet/passes` | Creer une carte Wallet |
| POST | `/api/v1/wallet/bulk` | Generer des liens Wallet en lot |

## Billing

| Methode | Route | Description |
| --- | --- | --- |
| GET | `/api/v1/billing/balance` | Solde actuel |
| GET | `/api/v1/billing/transactions` | Historique |
| GET | `/api/v1/billing/summary` | Resume mensuel |
| GET | `/api/v1/billing/pricing` | Grille tarifaire |
| PUT | `/api/v1/billing/pricing/{plan}` | Modifier un plan admin |

## Admin

| Methode | Route | Description |
| --- | --- | --- |
| GET | `/api/v1/admin/programmes` | Lister les programmes |
| POST | `/api/v1/admin/programmes` | Creer un programme |
| PUT | `/api/v1/admin/programmes/{id}` | Modifier un programme |

## Verify

| Methode | Route | Description |
| --- | --- | --- |
| GET | `/api/v1/verify/{code}` | Verifier un code voucher |
| GET | `/api/v1/verify/campaign/{id}/stats` | Stats publiques campagne |
| GET | `/api/v1/verify/campaign/{id}/offline` | Exporter un registre offline |
