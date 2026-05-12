# API Reference — v4.2

Base URL : `https://<your-domain>/api/v1`

Toutes les routes (sauf `/auth/login`, `/auth/register` et `/verify/*`) requièrent un header `Authorization: Bearer <access_token>`.

---

## Auth

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `POST` | `/auth/register` | Créer un compte programme |
| `POST` | `/auth/login` | Connexion — retourne `access_token` + `refresh_token` |
| `POST` | `/auth/refresh` | Renouveler l'access token via refresh token |
| `GET` | `/auth/me` | Profil du programme connecté |
| `PATCH` | `/auth/me` | Modifier le profil |
| `POST` | `/auth/change-password` | Changer le mot de passe |

---

## Campaigns

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/campaigns` | Lister les campagnes du programme |
| `POST` | `/campaigns` | Créer une campagne |
| `GET` | `/campaigns/{id}` | Détail d'une campagne |
| `PATCH` | `/campaigns/{id}` | Modifier une campagne |
| `DELETE` | `/campaigns/{id}` | Supprimer une campagne |
| `POST` | `/campaigns/{id}/template` | Uploader un template `.yml` |
| `GET` | `/campaigns/{id}/template/preview` | Prévisualiser le template |

---

## Jobs

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `POST` | `/campaigns/{id}/generate` | Lancer une génération asynchrone |
| `GET` | `/jobs` | Lister les jobs du programme |
| `GET` | `/jobs/{id}` | Détail + URLs de téléchargement |
| `GET` | `/jobs/{id}/estimate` | Estimation de coût avant lancement |
| `POST` | `/jobs/{id}/cancel` | Annuler un job en cours |
| `WS` | `/ws/jobs/{id}/progress?token=<jwt>` | Progression temps réel (WebSocket) |

---

## Card Studio

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `POST` | `/studio/preview` | Aperçu PNG temps réel (subprocess generator) |
| `POST` | `/studio/export` | Exporter un template JSON |
| `POST` | `/studio/render/png` | Générer un aperçu PNG haute résolution |

---

## DHIS2 & Distribution

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `POST` | `/dhis2/config` | Configurer la connexion DHIS2 |
| `GET` | `/dhis2/config` | Lire la configuration DHIS2 |
| `POST` | `/dhis2/test-connection` | Tester la connexion + lister les programmes |
| `POST` | `/dhis2/generate-cards` | Générer des cartes PNG depuis DHIS2 (async) |
| `POST` | `/dhis2/preview-card-png` | Aperçu PNG grille 4 cartes (données réelles) |
| `GET` | `/dhis2/jobs` | Historique des jobs DHIS2 |
| `GET` | `/dhis2/wallet/google-url` | JWT Google Wallet individuel |
| `GET` | `/dhis2/wallet/google-bulk-url` | JWT bulk (≤ 100 cartes) |
| `POST` | `/dhis2/send-card` | Envoyer une carte par WhatsApp ou SMS |
| `GET` | `/dhis2/sync-schedule` | Lire le planning de synchronisation auto |
| `POST` | `/dhis2/sync-schedule` | Configurer le planning de synchronisation auto |

---

## Messagerie

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `POST` | `/sms/send` | Envoyer un SMS (Orange API OAuth2) |
| `POST` | `/sms/bulk` | Envoyer des SMS en lot (batch 20) |
| `POST` | `/sms/test` | Tester la configuration SMS |
| `GET` | `/sms/logs` | Historique des envois SMS |

---

## Vérification publique

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/verify/{code}` | Vérifier un code voucher (sans auth) |
| `GET` | `/verify/campaign/{id}/stats` | Statistiques publiques d'une campagne |
| `GET` | `/verify/campaign/{id}/offline` | Exporter le registre offline |

---

## Simulation

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `POST` | `/simulation/quick-estimate` | Estimation rapide (sans persistence) |
| `GET` | `/simulation/dpi-pricing` | Comparatif coûts DPI 300 / 450 / 600 |
| `POST` | `/simulation` | Créer une simulation (`status: computed`) |
| `GET` | `/simulation` | Lister les simulations du programme |
| `GET` | `/simulation/{id}` | Détail d'une simulation |
| `POST` | `/simulation/{id}/submit` | Soumettre pour validation admin |
| `POST` | `/simulation/{id}/provision` | Déclencher le provisioning |
| `POST` | `/simulation/{id}/proforma` | Générer la proforma (PDF + Excel + JSON + YAML) |
| `GET` | `/simulation/{id}/proformas` | Lister les proformas d'une simulation |
| `GET` | `/simulation/{id}/proforma/{pid}/pdf` | Télécharger le PDF |
| `GET` | `/simulation/{id}/proforma/{pid}/xlsx` | Télécharger l'Excel |
| `GET` | `/simulation/{id}/config.json` | Exporter la config JSON |
| `GET` | `/simulation/{id}/config.yaml` | Exporter la config YAML |
| `GET` | `/simulation/admin/all` | **[Admin]** Lister toutes les simulations |
| `GET` | `/simulation/admin/{id}` | **[Admin]** Détail cross-programme |
| `POST` | `/simulation/admin/{id}/approve` | **[Admin]** Approuver |
| `POST` | `/simulation/admin/{id}/reject` | **[Admin]** Rejeter (motif obligatoire) |
| `POST` | `/simulation/admin/{id}/request-modification` | **[Admin]** Retourner à l'institution |
| `GET` | `/simulation/admin/{id}/proforma/{pid}/pdf` | **[Admin]** PDF cross-programme |
| `GET` | `/simulation/admin/{id}/config.json` | **[Admin]** Config JSON cross-programme |

---

## Facturation

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/billing/balance` | Solde + plan actuel |
| `GET` | `/billing/transactions` | Historique des transactions |
| `GET` | `/billing/summary` | Résumé mensuel (débits, crédits, count) |
| `GET` | `/billing/advanced-pricing` | Grille tarifaire deux moteurs |
| `GET` | `/billing/dpi-multipliers` | Multiplicateurs DPI |
| `GET` | `/billing/engine-usage` | Consommation par moteur (période courante) |
| `POST` | `/billing/change-plan` | Changer de plan d'abonnement |
| `GET` | `/billing/premium-modules` | Catalogue des modules premium |
| `GET` | `/billing/premium-modules/active` | Modules actifs du programme |
| `POST` | `/billing/premium-modules/activate` | Activer un module premium |
| `DELETE` | `/billing/premium-modules/{key}` | Désactiver un module |
| `GET` | `/billing/verify-platforms` | Plateformes de vérification dédiées |
| `POST` | `/billing/verify-platforms` | Créer une plateforme dédiée |
| `POST` | `/billing/recharge-requests` | Soumettre une demande de rechargement |
| `GET` | `/billing/recharge-requests` | Lister les demandes de rechargement |

### Plans Quota (v4.2)

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/billing/quota-plans` | Lister les 4 plans quota (public) |
| `PUT` | `/billing/quota-plans/{plan}` | **[Admin]** Configurer quotas / overage rates / infra_factor |

---

## Analytics Avancés *(module `advanced_analytics` requis)*

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/analytics/summary?days=30` | Résumé : campagnes, cartes, taux succès, dépenses |
| `GET` | `/analytics/campaigns-timeline?days=30` | Timeline journalière jobs + cartes |
| `GET` | `/analytics/spend-timeline?days=30` | Timeline journalière des dépenses FCFA |
| `GET` | `/analytics/campaign-types` | Répartition par type de campagne |
| `GET` | `/analytics/top-campaigns?limit=5` | Top campagnes par volume de cartes |

> Retourne `HTTP 403` si le module n'est pas activé pour ce programme.

---

## Détection Fraude IA *(module `ai_fraud_detection` requis)*

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/fraud/summary` | Résumé des anomalies détectées |
| `GET` | `/fraud/alerts` | Liste des alertes actives |
| `GET` | `/fraud/by-type` | Répartition des anomalies par type |
| `POST` | `/fraud/report` | Signaler une anomalie manuellement |
| `DELETE` | `/fraud/alerts/{id}` | Dismisser une alerte |

---

## Cartes PVC Physiques *(module `physical_pvc_card` requis)*

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/pvc` | Lister les commandes PVC |
| `POST` | `/pvc` | Passer une commande PVC |
| `GET` | `/pvc/{id}` | Détail d'une commande |

---

## Sécurité RBAC

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/security/permissions` | Catalogue des permissions disponibles |
| `GET` | `/security/roles` | Rôles disponibles |
| `GET` | `/security/me` | Profil RBAC du programme connecté |
| `PATCH` | `/security/programmes/{id}/role` | Assigner un rôle et des permissions |
| `GET` | `/security/audit-logs` | Journal d'audit (admin) |
| `GET` | `/security/approvals` | Workflows d'approbation en cours |
| `POST` | `/security/approvals` | Créer un workflow d'approbation |

---

## Administration *(is_admin requis)*

| Méthode | Route | Description |
| ------- | ----- | ----------- |
| `GET` | `/admin/programmes` | Lister tous les programmes |
| `POST` | `/admin/programmes` | Créer un programme |
| `PATCH` | `/admin/programmes/{id}` | Modifier un programme (plan, solde, statut) |
| `DELETE` | `/admin/programmes/{id}` | Supprimer un programme |
| `GET` | `/admin/stats` | Statistiques globales plateforme |
| `GET` | `/admin/recharge-requests` | Toutes les demandes de rechargement |
| `PATCH` | `/admin/recharge-requests/{id}` | Valider ou rejeter un rechargement |
| `GET` | `/admin/pricing` | Grille tarifaire admin |
| `PUT` | `/admin/pricing/{plan}` | Modifier une grille tarifaire |
| `POST` | `/admin/messaging/whatsapp` | Configurer WhatsApp (admin global) |
| `POST` | `/admin/messaging/sms` | Configurer SMS (admin global) |

---

## Codes d'erreur courants

| Code | Signification |
| ---- | ------------- |
| `400` | Données invalides ou contrainte métier non respectée |
| `401` | Token absent, expiré ou invalide |
| `403` | Accès refusé — droits insuffisants ou module non activé |
| `404` | Ressource introuvable |
| `409` | Conflit (ex. : plan déjà actif, ressource dupliquée) |
| `422` | Erreur de validation Pydantic |
| `500` | Erreur serveur interne |
