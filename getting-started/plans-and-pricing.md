# Plans & Fonctionnalités

Oomus CampaignID propose quatre plans adaptés à la taille et aux besoins de chaque programme de santé — du projet pilote au déploiement national.

> Les tarifs sont disponibles sur devis selon les accords institutionnels de votre programme. Contactez **ceo@oomus.org** pour obtenir une proposition personnalisée.

---

## Les quatre plans

### Starter

Idéal pour les **projets pilotes**, les programmes locaux ou les ONG à taille humaine. Conçu pour explorer toutes les fonctionnalités de la plateforme sur un périmètre géographique réduit (une commune, un district).

- Accès à tous les modèles de cartes
- Distribution WhatsApp et SMS inclus
- MPI souverain
- Intégration DHIS2 basique
- Support communautaire

---

### Regional Ops

Conçu pour les **programmes régionaux** opérant à l'échelle d'une région ou d'un groupe de districts sanitaires. Adapté aux campagnes saisonnières (vaccination de routine, distribution MILD).

- RBAC multi-niveaux avec gestion d'équipe
- Intégration DHIS2 complète (sync automatique)
- Support e-mail prioritaire

---

### National Campaign

Conçu pour les **déploiements nationaux** à grande échelle — directions nationales de santé, ministères, programmes intégrés multi-districts.

- Analytics avancées incluses
- Intégration DHIS2 complète avec sync automatique
- SLA renforcé
- Moteur de simulation avancé

---

### Sovereign Enterprise

Le plan **Enterprise souverain**, conçu pour les déploiements nationaux multi-programmes à très grande échelle. Option d'hébergement souverain disponible.

- Hébergement souverain optionnel (infrastructure dédiée en pays)
- SLA Premium/Critique contractuel
- Support dédié 24/7
- Accès à tous les modules premium
- Simulation et provisionnement avancés

---

## Dépassement de quota

Lorsque votre programme dépasse son quota mensuel de bénéficiaires, les cartes supplémentaires sont facturées à l'unité, au tarif configuré par votre administrateur de plateforme. Le coût unitaire est visible depuis votre dashboard (section **Facturation > Solde**).

Vous pouvez à tout moment :
- Consulter votre consommation en temps réel dans le Dashboard
- Soumettre une demande de rechargement de quota
- Changer de plan via l'API ou l'interface

---

## Modules premium

Les fonctionnalités suivantes sont disponibles en supplément, selon votre plan :

| Module | Description |
|---|---|
| **SMS Gateway** | Accès étendu aux passerelles SMS supplémentaires |
| **WhatsApp BSP** | Canal WhatsApp Business Solution Provider dédié |
| **Advanced Analytics** | Tableaux de bord analytiques avancés, exports |
| **AI Fraud Detection** | Détection d'anomalies temps réel par IA (IsolationForest) |
| **Offline Sync** | Synchronisation hors ligne avancée avec résolution de conflits |
| **Biometric Signature** | Signature biométrique des émissions de cartes |
| **Sovereign Hosting** | Hébergement dédié en infrastructure nationale ou régionale |
| **Physical PVC Cards** | Impression de cartes PVC physiques (partenaires certifiés) |
| **Dedicated Verification Portal** | Portail de vérification personnalisé, à votre domaine |
| **Advanced Simulation** | Moteur de simulation avancé, proforma contractuel |
| **Dedicated Provisioning** | Provisionnement d'infrastructure dédié avec SLA contractuel |
| **Advanced SLA** | Garanties SLA renforcées avec pénalités contractuelles |

---

## Matrice des fonctionnalités par plan

| Fonctionnalité | Starter | Regional Ops | National Campaign | Sovereign Enterprise |
|---|:---:|:---:|:---:|:---:|
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
| Analytics avancées | Non | Non | Oui | Oui |
| Moteur de simulation | Non | Basique | Avancé | Complet |
| SLA | Standard | Standard | Renforcé | Premium / Critique |
| Support | Communautaire | E-mail | Prioritaire | Dédié 24/7 |
| Hébergement souverain | Non | Non | Optionnel | Oui |
| Modules premium | Limité | Sélection | Sélection élargie | Tous |

---

## Changer de plan

```bash
curl -X POST https://api.oomus.health/billing/change-plan \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "plan_alias": "national_campaign"
  }'
```

Les alias de plan disponibles sont : `starter`, `regional_ops`, `national_campaign`, `sovereign_enterprise`.

---

## Questions fréquentes

**Est-ce que les SMS et WhatsApp non utilisés sont reportés ?**  
Non. Les quotas SMS/WhatsApp sont mensuels et non reportables au mois suivant.

**Puis-je changer de plan en cours de mois ?**  
Oui. Le changement de plan est effectif immédiatement. Un prorata peut s'appliquer selon les termes de votre contrat.

**Les prix peuvent-ils varier selon le pays ?**  
Oui. Les tarifs sont adaptés par les administrateurs de la plateforme pour refléter les accords institutionnels locaux.

**Existe-t-il une version d'essai gratuite ?**  
Contactez l'équipe Oomus pour un accès démo ou une période d'évaluation encadrée.

---

> Pour obtenir un devis personnalisé ou discuter d'un déploiement souverain, contactez-nous : **ceo@oomus.org**
