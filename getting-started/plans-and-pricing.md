# Plans & Tarification

Oomus CampaignID propose quatre plans adaptés à la taille et aux besoins de chaque programme de santé — du projet pilote au déploiement national.

> Les tarifs indiqués sont en **Francs CFA (FCFA)**. Les prix sont configurables par les administrateurs de la plateforme selon les accords institutionnels.

---

## Tableau comparatif des plans

| Plan | Abonnement / mois | Quota cartes/mois | Bénéficiaires | SMS inclus | WhatsApp | Stockage |
| --- | --- | --- | --- | --- | --- | --- |
| **Starter** | 100 000 FCFA | 3 000 cartes | 10 000 | 25 000 | 5 000 | 20 Go |
| **Regional Ops** | 250 000 FCFA | 50 000 cartes | 100 000 | 300 000 | 50 000 | 100 Go |
| **National Campaign** | 450 000 FCFA | 100 000 cartes | 1 000 000 | 3 000 000 | 500 000 | 1 000 Go |
| **Sovereign Enterprise** | 750 000 FCFA | Volume personnalisé | 10 000 000 | 30 000 000 | 5 000 000 | 10 000 Go |

---

## Tarification à la génération

Le quota mensuel de cartes est **inclus dans l'abonnement**. Toute carte générée au-delà du quota est facturée à l'unité, selon le moteur utilisé et le plan.

### Studio (Card Studio + Campagnes)

| Plan | Cartes incluses / mois | Prix / carte au-delà du quota | Surcharge 600 DPI |
| --- | --- | --- | --- |
| **Starter** | 3 000 | 8 FCFA | +2 FCFA |
| **Regional Ops** | 50 000 | 10 FCFA | +3 FCFA |
| **National Campaign** | 100 000 | 12 FCFA | +3 FCFA |
| **Sovereign Enterprise** | Volume négocié | 15 FCFA | +4 FCFA |

### DHIS2 Tracker

Les cartes générées depuis DHIS2 Tracker utilisent le même quota mensuel que les cartes Studio. Au-delà du quota :

| Plan | Prix / carte DHIS2 au-delà du quota | Surcharge DPI ≥ 450 |
| --- | --- | --- |
| **Starter** | 9 FCFA | +2 FCFA |
| **Regional Ops** | 10 FCFA | +3 FCFA |
| **National Campaign** | 15 FCFA | +3 FCFA |
| **Sovereign Enterprise** | 20 FCFA | +4 FCFA |

> **Important :** Le quota mensuel est partagé entre toutes les générations du mois (Studio + DHIS2). Il se remet à zéro le 1er de chaque mois.

---

## Logique de facturation à la génération

```text
1. Vérifier les cartes générées ce mois (Studio + DHIS2 confondus)
2. Si cartes_utilisées < quota_mensuel :
   → Cartes dans le quota = 0 FCFA
   → Cartes au-delà du quota = prix/carte du plan
   (un même job peut être partiellement gratuit / partiellement facturé)
3. Si cartes_utilisées ≥ quota_mensuel :
   → Toutes les cartes du job = prix/carte du plan
4. La génération est bloquée si : cartes_utilisées > quota × 3
```

**Exemple — plan Starter :**

| Situation | Job de 5 000 cartes | Coût |
| --- | --- | --- |
| 0 carte utilisée ce mois | 3 000 gratuites + 2 000 facturées | 2 000 × 8 = **16 000 FCFA** |
| 2 000 cartes déjà utilisées | 1 000 gratuites + 4 000 facturées | 4 000 × 8 = **32 000 FCFA** |
| 3 500 cartes déjà utilisées | 0 gratuites + 5 000 facturées | 5 000 × 8 = **40 000 FCFA** |

---

## Description des plans

### Starter — 100 000 FCFA/mois

Idéal pour les **projets pilotes**, les programmes locaux ou les ONG à taille humaine. Conçu pour explorer toutes les fonctionnalités de la plateforme sur un périmètre géographique réduit (une commune, un district).

- Jusqu'à 10 000 bénéficiaires
- 3 000 cartes/mois incluses (Studio + DHIS2)
- 25 000 SMS et 5 000 messages WhatsApp inclus
- 20 Go de stockage sécurisé
- Accès à tous les modèles de cartes
- Support communautaire

---

### Regional Ops — 250 000 FCFA/mois

Conçu pour les **programmes régionaux** opérant à l'échelle d'une région ou d'un groupe de districts sanitaires. Adapté aux campagnes saisonnières (vaccination de routine, distribution MILD).

- Jusqu'à 100 000 bénéficiaires
- 50 000 cartes/mois incluses (Studio + DHIS2)
- 300 000 SMS et 50 000 messages WhatsApp inclus
- 100 Go de stockage sécurisé
- RBAC avec gestion d'équipe (rôles multiples)
- Support e-mail prioritaire

---

### National Campaign — 450 000 FCFA/mois

Conçu pour les **déploiements nationaux** à grande échelle — directions nationales de santé, ministères, programmes intégrés multi-districts.

- Jusqu'à 1 000 000 de bénéficiaires
- 100 000 cartes/mois incluses (Studio + DHIS2)
- 3 000 000 SMS et 500 000 messages WhatsApp inclus
- 1 000 Go de stockage sécurisé
- Analytics avancées incluses
- Intégration DHIS2 complète (sync automatique)
- SLA renforcé

---

### Sovereign Enterprise — 750 000 FCFA/mois

Le plan **Enterprise souverain**, conçu pour les déploiements nationaux multi-programmes à très grande échelle. Volume de cartes personnalisé, négocié contractuellement.

- Jusqu'à 10 000 000 de bénéficiaires
- Volume de cartes personnalisé (toutes cartes facturées à 15–20 FCFA/carte)
- 30 000 000 SMS et 5 000 000 messages WhatsApp inclus
- 10 000 Go de stockage sécurisé
- Hébergement souverain optionnel (infrastructure dédiée en pays)
- SLA Premium/Critique contractuel
- Support dédié 24/7
- Accès à tous les modules premium
- Simulation et provisionnement avancés

---

## Dépassement de quota — Comportement automatique

Lorsque votre programme dépasse son quota mensuel de cartes, la plateforme bascule automatiquement en mode facturation à l'unité **sans interrompre la génération**.

- Le montant est débité du solde du compte en temps réel
- La description de la transaction précise les cartes facturées vs. les cartes gratuites
- La génération est **bloquée** si le total dépasse 3 fois le quota mensuel (protection contre les dépassements involontaires)

Vous pouvez à tout moment :

- Consulter votre consommation mensuelle dans **Dashboard → Facturation → Quota**
- Soumettre une demande de rechargement de solde
- Changer de plan pour augmenter le quota inclus

---

## Modules premium

Les fonctionnalités suivantes sont disponibles en supplément, selon votre plan :

| Module | Description | Prix/mois |
| --- | --- | --- |
| **MPI Identité Numérique** | Registre souverain des identités de santé cross-programmes | 45 000 FCFA |
| **SMS Gateway** | Accès étendu aux passerelles SMS supplémentaires | 25 000 FCFA |
| **WhatsApp BSP** | Canal WhatsApp Business Solution Provider dédié | 75 000 FCFA |
| **Advanced Analytics** | Tableaux de bord analytiques avancés, exports | 35 000 FCFA |
| **AI Fraud Detection** | Détection d'anomalies temps réel par IA (IsolationForest) | 60 000 FCFA |
| **Offline Sync** | Synchronisation hors ligne avancée avec résolution de conflits | 20 000 FCFA |
| **Biometric Signature** | Signature biométrique des émissions de cartes | 45 000 FCFA |
| **Sovereign Hosting** | Hébergement dédié en infrastructure nationale ou régionale | 150 000 FCFA |
| **Physical PVC Cards** | Impression de cartes PVC physiques (partenaires certifiés) | 350 FCFA/carte |
| **Dedicated Verification Portal** | Portail de vérification personnalisé, à votre domaine | 85 000 FCFA |
| **Advanced Simulation** | Moteur de simulation avancé, proforma contractuel | 15 000 FCFA |
| **Dedicated Provisioning** | Provisionnement d'infrastructure dédié avec SLA contractuel | 200 000 FCFA |
| **Advanced SLA** | Garanties SLA renforcées avec pénalités contractuelles | 50 000 FCFA |

---

## Matrice des fonctionnalités par plan

| Fonctionnalité | Starter | Regional Ops | National Campaign | Sovereign Enterprise |
| --- | :---: | :---: | :---: | :---: |
| Génération de cartes (quota) | 3 000/mois | 50 000/mois | 100 000/mois | Personnalisé |
| Card Studio (11 modèles) | Oui | Oui | Oui | Oui |
| DHIS2 (quota partagé) | Oui | Oui | Oui | Oui |
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

**Le quota de cartes est-il partagé entre Studio et DHIS2 ?**  
Oui. Le quota mensuel de cartes est commun à tous les moteurs de génération (Card Studio, Campagnes, DHIS2 Tracker). C'est le total qui détermine si les cartes sont gratuites ou facturées.

**Est-ce que les cartes non utilisées sont reportées ?**  
Non. Le quota mensuel de cartes (et SMS/WhatsApp) se remet à zéro le 1er de chaque mois. Le reliquat n'est pas reportable.

**Puis-je changer de plan en cours de mois ?**  
Oui. Le changement de plan est effectif immédiatement. Le nouveau quota (plus élevé) est disponible sans délai.

**Les prix peuvent-ils varier selon le pays ?**  
Oui. Les tarifs peuvent être adaptés par les administrateurs de la plateforme pour refléter les accords institutionnels locaux (USD, EUR, MRU, GNF, etc.).

**Que se passe-t-il si mon solde est insuffisant pour payer le dépassement ?**  
La génération est bloquée avant de démarrer si le solde est inférieur au coût estimé du dépassement. Un message d'erreur indique le coût attendu et le solde disponible.

**Existe-t-il une version d'essai gratuite ?**  
Contactez l'équipe Oomus pour un accès démo ou une période d'évaluation encadrée.

---

> Pour obtenir un devis personnalisé ou discuter d'un déploiement souverain, contactez-nous : **contact@oomus.health**
