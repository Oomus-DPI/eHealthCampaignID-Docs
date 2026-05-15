# Moteur de simulation

Le moteur de simulation d'Oomus CampaignID permet à un programme de santé d'**estimer précisément les coûts, l'infrastructure et les SLA** d'un déploiement avant tout engagement. Il génère des documents proforma contractuels et suit un workflow d'approbation administrateur.

---

## Pourquoi simuler ?

Avant de lancer un programme national de distribution de cartes, les décideurs ont besoin de réponses précises à des questions opérationnelles et financières :

- Combien va coûter la génération de 500 000 cartes en DPI 600 ?
- Quelle infrastructure est nécessaire pour garantir un SLA 99,9% ?
- Quels modules premium sont requis pour mon cas d'usage ?
- Comment documenter l'engagement pour l'approbation budgétaire ?

Le moteur de simulation répond à toutes ces questions et génère les documents nécessaires à l'approbation institutionnelle.

---

## L'assistant wizard en 6 étapes

La simulation est configurée via un assistant guidé en 6 étapes :

### Étape 1 — Sélection du moteur

| Moteur | Description | Cas d'usage |
|---|---|---|
| **Campaign Delivery** | Génération et distribution de cartes numériques | Cartes WhatsApp, SMS, Google Wallet |
| **Studio Print** | Génération haute résolution pour impression physique | Cartes PVC, impression offset |

### Étape 2 — Paramètres de campagne

- Nombre de bénéficiaires cibles
- Type de programme (vaccination, nutrition, MILD, etc.)
- Langue de la campagne
- Modèle de carte sélectionné
- Délai de génération souhaité (24h, 48h, 1 semaine, etc.)

### Étape 3 — Infrastructure

| Option | Description |
|---|---|
| **Infrastructure partagée** | Ressources mutualisées entre programmes (coût réduit) |
| **Infrastructure dédiée** | Ressources exclusives pour votre programme (performance garantie) |

L'infrastructure dédiée est requise pour les SLA Premium et Critique.

### Étape 4 — Gouvernance RBAC

Configuration du modèle de gouvernance :
- Nombre d'utilisateurs et rôles (super_admin, programme_admin, programme_user)
- Niveaux d'approbation requis (1, 2 ou 3 niveaux)
- Périmètre géographique (national, régional, district)

### Étape 5 — Modules premium

Sélection des modules optionnels selon les besoins du programme :

- SMS Gateway étendu
- WhatsApp BSP dédié
- Analytics avancées
- Détection de fraude IA
- Synchronisation hors ligne
- Signature biométrique
- Hébergement souverain
- Impression PVC physique
- Portail de vérification dédié
- SLA avancé
- Provisionnement dédié

### Étape 6 — Résultats et documents

La simulation génère les résultats et les documents exportables.

---

## Options DPI — Multiplicateurs

La résolution DPI impacte directement le coût de génération et le volume de stockage :

| DPI | Facteur multiplicateur | Usage |
|---|---|---|
| 300 dpi | ×1.0 (référence) | Affichage numérique, distribution mobile |
| 450 dpi | ×1.4 | Impression laser, badges |
| 600 dpi | ×2.0 | Impression PVC haute qualité |

---

## Niveaux SLA

Quatre niveaux de SLA sont disponibles, chacun avec un multiplicateur de coût :

| Niveau SLA | Disponibilité cible | Temps de réponse P1 | Multiplicateur |
|---|---|---|---|
| **Standard** | 99,0% | 8 heures ouvrables | ×1.0 |
| **Enhanced** | 99,5% | 4 heures | ×1.3 |
| **Premium** | 99,9% | 1 heure | ×1.8 |
| **Critical** | 99,99% | 15 minutes | ×2.5 |

Les SLA Premium et Critique nécessitent une infrastructure dédiée.

---

## Documents générés

À l'issue de la simulation, les documents suivants sont générés et téléchargeables :

| Document | Format | Contenu |
|---|---|---|
| **Proforma PDF** | PDF | Devis détaillé, conditions tarifaires, SLA contractuels |
| **Simulation Excel** | XLSX (4 onglets) | Détail des coûts, planning, capacité, gouvernance |
| **Configuration JSON** | JSON | Paramètres techniques complets |
| **Configuration YAML** | YAML | Version lisible de la configuration |

### Structure du fichier Excel (4 onglets)

1. **Résumé exécutif** : coût total, bénéficiaires, SLA, modules
2. **Détail des coûts** : décomposition ligne à ligne
3. **Planning de provisionnement** : jalons, délais, livrables
4. **Gouvernance & RBAC** : rôles, niveaux d'approbation, contacts

---

## Workflow d'approbation administrateur

Toute simulation soumise suit un workflow d'approbation avant provisionnement :

```
Soumission simulation
        │
        ▼
   [submitted]
        │
        ▼ (administrateur Oomus examine)
        │
   ┌────┴──────────────────────┐
   │                           │
   ▼                           ▼
[approved]           [modification_requested]
   │                           │
   │                           ▼
   │              (Programme corrige et resoumet)
   │                           │
   │                    [submitted] (itération)
   │
   ▼
[provisioning]
(Équipe technique déploie l'infrastructure)
        │
        ▼
[provisioned]
(Programme opérationnel)
```

### Statuts du workflow

| Statut | Description |
|---|---|
| `submitted` | Simulation soumise pour approbation |
| `approved` | Simulation approuvée, provisionnement lancé |
| `rejected` | Simulation rejetée (motif communiqué) |
| `modification_requested` | Modifications demandées par l'admin |
| `provisioning` | Infrastructure en cours de déploiement |
| `provisioned` | Programme opérationnel |

---

## Accès à la simulation

La simulation est accessible depuis :
- **Dashboard** > Simulation > Nouvelle simulation
- **API** : `POST /simulations/` (détails dans la référence API)

Niveau de plan requis :
- Plan **Regional Ops** : accès simulation basique
- Plan **National Campaign** : accès simulation avancée
- Plan **Sovereign Enterprise** : simulation complète + provisionnement dédié

---

## Prochaines étapes

- [Plans & Tarification](../getting-started/plans-and-pricing.md) — Choisir le bon plan
- [Dashboard & Analytics](dashboard-analytics.md) — Suivre l'utilisation réelle
