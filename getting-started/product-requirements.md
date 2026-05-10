# Product Requirements Document - Oomus eHealth CampaignID v3.5

## Objectif produit

Permettre a un ministere de la sante, une ONG, un programme national ou un partenaire technique de creer, personnaliser, generer, distribuer, verifier et superviser des cartes digitales securisees pour des campagnes de sante publique.

## Fonctionnalites principales

- Creation et authentification des comptes.
- Gestion des campagnes.
- Card Studio no-code.
- Upload de templates YAML/JSON.
- Preview du template et previsualisation PNG.
- Lancement de generation asynchrone.
- Suivi temps reel par WebSocket.
- Telechargement PDF, ZIP, registre et rapport.
- Facturation SaaS par usage.
- Verification publique par code.
- Synchronisation DHIS2 Tracker.
- Distribution WhatsApp et SMS.
- Emission Google Wallet.
- Verification offline par portail statique.
- Monitoring des workers, synchronisations et generations.
- Detection d'anomalies sur les codes generes.

## Utilisateurs cibles

- Administrateur plateforme.
- Gestionnaire de programme.
- Operateur de campagne.
- Verificateur public.
- Equipe de supervision terrain.
- Partenaire technique.

## Card Studio

Le Card Studio est un module no-code de conception de cartes digitales personnalisees.

Templates disponibles:

- Vaccination.
- Nutrition.
- PTME / VIH.
- Assurance maladie.
- Carte generique.
- Carte agriculteur.
- Carte refugie.
- Carte laboratoire.
- Carte prenatale.
- Distribution MILD.
- Chimioprevention paludisme.

Fonctionnalites:

- Gestion des logos.
- Couleurs personnalisees.
- QR codes dynamiques.
- Champs dynamiques.
- Champs verso.
- Apercu temps reel.
- Export JSON.
- Previsualisation PNG.

## Criteres d'acceptation

- Un utilisateur authentifie peut creer une campagne.
- Une campagne sans template ne peut pas etre generee.
- Un job expose son statut et sa progression.
- Un job termine expose des URLs de telechargement.
- Un code voucher genere peut etre verifie sans authentification.
- Une campagne peut etre alimentee depuis DHIS2 Tracker si l'integration est configuree.
- Une carte peut etre distribuee par canal numerique active.
- Le portail de verification peut fonctionner hors connexion avec un registre exporte.
- Les QR codes ne doivent pas exposer de donnees medicales sensibles.
