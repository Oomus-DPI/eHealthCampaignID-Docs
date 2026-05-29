# Plans & Fonctionnalités

Oomus CampaignID propose quatre plans adaptés à la taille et aux besoins de chaque programme de santé — du projet pilote au déploiement national.

> Les tarifs sont disponibles sur devis selon les accords institutionnels de votre programme. Contactez **ceo@oomus.org**.

---

## Les quatre plans

### Quotas mensuels

| Plan | Identités | SMS | WhatsApp | Vérifications | Stockage |
| --- | --- | --- | --- | --- | --- |
| **Essential** (`starter`) | 10 000 | 50 000 | 10 000 | 5 000 | 10 Go |
| **Regional Command** (`regional_ops`) | 100 000 | 250 000 | 100 000 | 50 000 | 50 Go |
| **National Infrastructure** (`national_campaign`) | 1 000 000 | 3 000 000 | 1 000 000 | 500 000 | 500 Go |
| **Sovereign Cloud** (`sovereign_enterprise`) | Illimité | Illimités | Illimités | Illimitées | Illimité |

> **Identités** = cartes générées via Card Studio + DHIS2 Tracker — quota partagé entre les deux moteurs.

Les modules listés ci-dessous sont **activés automatiquement** à la souscription du plan — aucune configuration manuelle requise.

---

### Essential (`starter`)

Idéal pour les **projets pilotes**, les programmes locaux ou les ONG à taille humaine.

**Modules inclus automatiquement :**

- Distribution SMS Omnicanale (`sms_gateway`)
- Studio de Production de Documents (`pvc_studio`)

**Fonctionnalités :**

- Accès à tous les 11 modèles de cartes (dont template Sovereign)
- Intégration DHIS2 basique
- MPI souverain
- Support communautaire

---

### Regional Command (`regional_ops`)

Conçu pour les **programmes régionaux** opérant à l'échelle d'une région ou d'un groupe de districts sanitaires.

**Modules inclus automatiquement :**

- Distribution SMS Omnicanale
- Distribution WhatsApp (`whatsapp_bsp`)
- Studio de Production de Documents
- Synchronisation Terrain Souveraine (`offline_sync`)
- Centre de Supervision Territoriale (`geo_command_center`)
- Intelligence Opérationnelle (`advanced_analytics`)

**Fonctionnalités supplémentaires :**

- RBAC multi-niveaux avec gestion d'équipe
- Intégration DHIS2 complète (sync automatique)
- Support e-mail prioritaire

---

### National Infrastructure (`national_campaign`)

Conçu pour les **déploiements nationaux** à grande échelle — directions nationales de santé, ministères.

**Modules inclus automatiquement :**

- Tous les modules Regional Command
- Détection d'Anomalies & Conformité (`ai_fraud_detection`)
- Portefeuille Citoyen Souverain (`sovereign_wallet`)
- Registre d'Identités MPI (`mpi_registry`)
- Portail National de Vérification (`dedicated_verify_platform`)
- SLA 99,9% Garanti (`advanced_sla`)

**Fonctionnalités supplémentaires :**

- SLA renforcé 99,9%
- Moteur de simulation avancé (5 modes)
- Support prioritaire

---

### Sovereign Cloud (`sovereign_enterprise`)

Le plan **Enterprise souverain**, conçu pour les déploiements multi-programmes à très grande échelle.

**Modules inclus automatiquement :**

- Tous les modules National Infrastructure
- Hébergement Cloud Souverain (`sovereign_hosting`)
- Vérification Biométrique Avancée (`biometric_signature`)
- Provisioning Dédié par Pays (`dedicated_provisioning`)
- SLA 99,99% Garanti

**Fonctionnalités supplémentaires :**

- Infrastructure dédiée par pays
- SLA Premium/Critique contractuel
- Support dédié 24/7
- Simulation complète avec provisionnement dédié

---

## Dépassement de quota

Lorsque votre programme dépasse son quota mensuel, les cartes supplémentaires sont facturées à l'unité au tarif configuré par votre administrateur. Le coût unitaire est visible depuis **Facturation > Solde**.

Vous pouvez à tout moment :

- Consulter votre consommation en temps réel dans le Dashboard
- Soumettre une demande de rechargement de quota
- Changer de plan via l'API ou l'interface

---

## Modules premium additionnels

Les modules suivants sont disponibles en add-on selon votre plan :

| Module | Clé | Description |
| --- | --- | --- |
| **SMS Gateway** | `sms_gateway` | Passerelles SMS (Orange, Africa's Talking) |
| **WhatsApp BSP** | `whatsapp_bsp` | Canal WhatsApp Meta Graph API v25.0 |
| **Advanced Analytics** | `advanced_analytics` | Intelligence opérationnelle, exports |
| **AI Fraud Detection** | `ai_fraud_detection` | Détection d'anomalies par IA (Z-score) |
| **Offline Sync** | `offline_sync` | Synchronisation terrain hors ligne |
| **Biometric Signature** | `biometric_signature` | Signature biométrique des émissions |
| **Sovereign Hosting** | `sovereign_hosting` | Infrastructure dédiée en pays |
| **Physical PVC Cards** | `pvc_studio` | Cartes PVC physiques (Standard / Offset Industriel) |
| **Dedicated Verify Portal** | `dedicated_verify_platform` | Portail de vérification à votre domaine |
| **Advanced Simulation** | `advanced_simulation` | Moteur de simulation avancé, proforma |
| **Dedicated Provisioning** | `dedicated_provisioning` | Provisionnement infrastructure dédié |
| **Advanced SLA** | `advanced_sla` | SLA renforcé avec pénalités contractuelles |

---

## Matrice des fonctionnalités

| Fonctionnalité | Essential | Regional Command | National Infra | Sovereign Cloud |
| --- | :---: | :---: | :---: | :---: |
| Génération de cartes | Oui | Oui | Oui | Oui |
| Card Studio (11 modèles) | Oui | Oui | Oui | Oui |
| Distribution WhatsApp | Oui | Oui | Oui | Oui |
| Distribution SMS | Oui | Oui | Oui | Oui |
| Google Wallet | Oui | Oui | Oui | Oui |
| Vérification hors ligne | Oui | Oui | Oui | Oui |
| Intégration DHIS2 | Basique | Complète | Complète | Complète |
| MPI souverain | Oui | Oui | Oui | Oui |
| RBAC (niveaux) | 2 | 3 | 4 | 4+ personnalisé |
| DPI disponibles | 300 | 300 / 450 | 300 / 450 / 600 | 300 / 450 / 600 |
| Analytics avancées | Non | Oui | Oui | Oui |
| Geo Command Center | Non | Oui | Oui | Oui |
| Moteur de simulation | Non | Basique | 5 modes | Complet |
| SLA | Standard | Standard | 99,9% | 99,99% |
| Support | Communautaire | E-mail | Prioritaire | Dédié 24/7 |
| Hébergement souverain | Non | Non | Optionnel | Oui |

---

## Changer de plan

```bash
curl -X POST https://api.oomus.health/billing/change-plan \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{ "plan_alias": "national_campaign" }'
```

Les alias disponibles : `starter`, `regional_ops`, `national_campaign`, `sovereign_enterprise`.

Le changement de plan débite le prix mensuel complet et active automatiquement tous les modules inclus dans le nouveau plan.

---

## Questions fréquentes

**Est-ce que les SMS et WhatsApp non utilisés sont reportés ?**
Non. Les quotas SMS/WhatsApp sont mensuels et non reportables au mois suivant.

**Puis-je changer de plan en cours de mois ?**
Oui. Le changement est effectif immédiatement. Un prorata peut s'appliquer selon votre contrat.

**Les prix peuvent-ils varier selon le pays ?**
Oui. Les tarifs sont adaptés par les administrateurs pour refléter les accords institutionnels locaux.

**Existe-t-il une version d'essai gratuite ?**
Contactez l'équipe Oomus pour un accès démo ou une période d'évaluation encadrée.

---

> Pour obtenir un devis personnalisé ou discuter d'un déploiement souverain, contactez-nous : **ceo@oomus.org**
