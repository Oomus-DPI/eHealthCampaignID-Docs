# Card Studio

Le Card Studio est l'éditeur visuel intégré d'Oomus CampaignID. Il permet de concevoir, personnaliser et prévisualiser les cartes de santé numériques avant leur génération en masse.

---

## Modèles disponibles

Oomus CampaignID propose **11 modèles de cartes** prêts à l'emploi, couvrant les principaux programmes de santé publique :

| Identifiant | Programme | Description |
|---|---|---|
| `cps` | Protection sociale | Carte de protection sociale communautaire |
| `mild` | Paludisme / MILD | Carte de distribution de moustiquaires imprégnées |
| `vaccination` | Vaccination (EPI) | Carte de vaccination standard (Programme élargi de vaccination) |
| `antenatal` | Santé maternelle | Carte de suivi anténatal et consultations prénatales |
| `nutrition` | Nutrition | Carte de suivi nutritionnel (enfant, femme allaitante) |
| `hiv` | HIV/PTME | Carte de programme HIV/PTME (données sensibles protégées) |
| `lab` | Laboratoire | Carte de résultats et suivi biologique |
| `assurance` | Assurance maladie | Carte d'affiliation à l'assurance maladie universelle |
| `refugee` | Réfugiés | Carte d'identification humanitaire |
| `identity` | Identité sanitaire | Carte d'identité nationale de santé |
| `farmercard` | Santé agricole | Carte agriculteur / santé rurale |

---

## Capacités de l'éditeur visuel

### Logos et identité visuelle

Chaque modèle de carte supporte jusqu'à **3 emplacements de logo** configurables :

- **Logo principal** : logo du programme ou du ministère (positionné en haut gauche ou centré)
- **Logo partenaire** : logo d'un partenaire technique ou financier (ONG, agence UN)
- **Logo gouvernement** : logo de l'État ou de la collectivité locale

Formats acceptés : PNG (fond transparent recommandé), JPEG.  
Dimensions recommandées : 200×100 px minimum, ratio respecté automatiquement.

### Couleurs

L'identité couleur de votre programme est entièrement personnalisable :

- **Couleur primaire** : couleur principale de l'en-tête et des accents
- **Couleur secondaire** : couleur de fond, séparateurs, pieds de page
- **Couleur du texte** : adapté automatiquement au contraste (accessibilité)

Le sélecteur de couleur supporte les formats HEX, RGB et les codes couleur des identités visuelles institutionnelles.

### Personnalisation du QR code

Le QR code est l'élément de sécurité central de chaque carte. Options disponibles :

- **Activation/désactivation** du QR code sur la carte
- **Position** : bas-droite (défaut), bas-gauche, centré
- **Taille** : 3 niveaux (compact, standard, large)
- **Style** : QR classique ou avec logo de programme intégré au centre
- **Jeton** : toujours un jeton opaque dérivé SHA-256 (non réversible)

> Le QR code ne contient jamais l'identifiant MPI en clair. Il utilise un jeton cryptographique opaque, garantissant la vie privée des bénéficiaires. Voir [Garanties cryptographiques](../security/cryptographic-guarantees.md).

### Champs dynamiques — Recto

La face recto de la carte contient les informations principales du bénéficiaire. Champs configurables :

| Champ | Type | Obligatoire |
|---|---|---|
| Prénom / Nom | Texte | Oui |
| Date de naissance | Date | Recommandé |
| Numéro de bénéficiaire | Texte | Oui |
| Identifiant MPI | Texte (pied de carte) | Optionnel |
| Groupe cible | Texte | Optionnel |
| Date d'émission | Date | Recommandé |
| Date d'expiration | Date | Optionnel |
| Photo (vignette) | Image | Optionnel |

### Champs dynamiques — Verso

La face verso de la carte peut contenir des informations complémentaires selon le programme :

- Nom et coordonnées du centre de santé
- Liste des vaccins ou actes couverts
- Zone géographique (région, district)
- Informations d'urgence
- Conditions de validité
- Barcode/code-barre complémentaire
- Instructions de vérification (URL du portail + texte)

### Pied de carte — Identifiant MPI

Sur les modèles DHIS2 (vital/emerald/pulse), l'identifiant MPI s'affiche dans le **pied de carte**, remplaçant le code-barre traditionnel. Format : `SN-DKR-26-9XQ7LM2A` — voir [Identité souveraine MPI](mpi-sovereign-identity.md).

---

## Aperçu PNG

Avant tout lancement de génération en masse, le Card Studio génère un **aperçu PNG en temps réel** de la carte finale :

1. Cliquez sur **"Aperçu PNG"** dans l'éditeur
2. Le serveur génère une image PNG haute résolution de la carte recto/verso
3. L'aperçu est affiché directement dans le navigateur
4. Téléchargez l'aperçu pour validation interne ou approbation institutionnelle

L'aperçu utilise des données fictives représentatives. Il reflète exactement le rendu final à la DPI choisie.

---

## Options DPI

La résolution DPI (points par pouce) détermine la qualité d'impression des cartes :

| DPI | Utilisation | Facteur de qualité |
|---|---|---|
| **300 dpi** | Affichage numérique, impression standard | ×1.0 (référence) |
| **450 dpi** | Impression professionnelle, imprimante laser | ×1.4 |
| **600 dpi** | Impression PVC haute qualité, badges plastifiés | ×2.0 |

> Les options DPI disponibles dépendent de votre plan. Le plan Starter offre uniquement le 300 dpi. Les plans Regional Ops et supérieurs débloquent le 450 et 600 dpi.

---

## Export de configuration — YAML / JSON

Chaque design de carte peut être exporté en **YAML ou JSON** pour :

- Versionner la configuration dans un dépôt Git
- Partager un design entre plusieurs campagnes
- Importer un design existant pour une nouvelle campagne
- Audit et documentation interne

Exemple d'export YAML :

```yaml
template_id: vaccination
version: "2.1"
language: fr
colors:
  primary: "#00A651"
  secondary: "#F5F5F5"
logos:
  main: "logo_programme.png"
  partner: "logo_unicef.png"
  government: "logo_ministere.png"
qr_code:
  enabled: true
  position: bottom-right
  size: standard
fields_recto:
  - name: full_name
    label: "Nom complet"
    required: true
  - name: date_of_birth
    label: "Date de naissance"
    format: "DD/MM/YYYY"
  - name: beneficiary_id
    label: "N° Bénéficiaire"
  - name: mpi_id
    label: "ID Santé"
    position: footer
fields_verso:
  - name: health_center
    label: "Centre de santé"
  - name: vaccines_covered
    label: "Vaccins couverts"
    type: list
dpi: 300
```

---

## Bonnes pratiques

- **Validez toujours l'aperçu PNG** avant de lancer une génération en masse — les corrections après génération nécessitent un nouveau job
- **Exportez votre configuration YAML** après chaque modification pour versionner votre design
- **Utilisez le 600 dpi** uniquement si vous prévoyez une impression PVC physique — la taille des fichiers est proportionnellement plus grande
- **Testez avec 5–10 bénéficiaires** avant de lancer une campagne complète de plusieurs milliers de cartes

---

## Prochaines étapes

- [Gestion des campagnes](campaigns.md) — Lancer la génération en masse
- [Identité MPI souveraine](mpi-sovereign-identity.md) — Activer l'identifiant souverain sur vos cartes
- [Distribution multicanal](distribution.md) — Envoyer les cartes générées
