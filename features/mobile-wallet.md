# OOMUS Wallet — Application mobile

> React Native 0.81.5 · Expo SDK 54 · TypeScript · iOS + Android · **v8.0**

L'application mobile OOMUS Wallet permet à chaque citoyen d'accéder à son identité souveraine MPI, de vérifier son identité via le wizard KYC, de consulter ses passes de santé, de contrôler le partage de ses données et de vérifier sa couverture d'assurance — directement depuis son téléphone.

---

## Authentification

| Étape | Mécanisme |
| --- | --- |
| **Connexion** | Numéro de téléphone → OTP SMS (30 s resend) |
| **PIN** | 6 chiffres, stocké dans `SecureStore` (iOS Keychain / Android Keystore) |
| **Biométrie** | Face ID (iOS) ou Fingerprint (Android) — verrouillage automatique en arrière-plan |
| **Session** | JWT access + refresh, rotation automatique via `AuthContext` |

---

## Écrans

| Écran | Description |
| --- | --- |
| **LoginScreen** | Saisie numéro de téléphone, validation format, indicatif pays |
| **OtpScreen** | Code OTP 6 chiffres, auto-avancement, collage automatique |
| **PinScreen** | Création et vérification PIN — 6 cases animées |
| **WalletHomeScreen** | `SovereignCard` boarding-pass, résumé MPI, actions rapides (`adjustsFontSizeToFit`), activité récente |
| **PortefeuilleCitoyenScreen** | Activité live WebSocket, `ACTION_LABELS` humains, filtres Tous · Portefeuille · Santé · Identité, toasts temps réel |
| **RegistreIdentitesScreen** | Carte IAL animée, niveaux IAL 0–3, services débloqués, empreinte cryptographique, signalement d'erreur |
| **KycFlowScreen** | Wizard 5 étapes — CNI + selfie biométrique, score MPI propagé en temps réel après finalisation |
| **PassesScreen** | Grille des passes de santé, filtres, overlay `Niveau IAL{n} requis`, pull-to-refresh |
| **PassDetailScreen** | Détail pass + QR HMAC, slider TTL partage temporaire |
| **ConsentsScreen** | Consentements actifs + révocation. GrantModal 2 étapes |
| **AuditScreen** | Journal d'activité chronologique — labels humains, fallback `'Événement système'`, filtres par catégorie |
| **InsuranceScreen** | Couvertures, vérification d'actes médicaux, historique des vérifications |
| **SettingsScreen** | Profil MPI, sécurité (PIN · biométrie), recertification d'identité hebdomadaire, langue, déconnexion |

---

## KYC → Score de Confiance MPI (v8.0)

Le wizard KYC est connecté à la plateforme OOMUS : chaque vérification d'identité met à jour le `MpiTrustScore` en temps réel et rafraîchit automatiquement tous les écrans mobiles.

### Flux complet

```text
KycFlowScreen (étape 5 — Confirmation)
  → POST /api/v1/mobile/kyc/finalize
    → kyc_trust_bridge.finalize_kyc_to_trust()
      → MpiTrustScore upsert  (signal kyc_document_verified=True, weight=25)
      → MpiVerificationEvent  (event_type="kyc_check")
    → publish_wallet_event(wallet_id, "kyc.completed", {trust_score, trust_level, upgraded})
      → WebSocket → SyncContext.syncNow() après 1,2 s
        → PortefeuilleCitoyenScreen + RegistreIdentitesScreen rafraîchis
  → Admin OOMUS GET /mpi/trust/{mpi_id} → TrustScoreCard affiche le nouveau score
```

### Wizard KYC — 5 étapes

| Étape | Contenu |
| --- | --- |
| **1 — Introduction** | Explication du processus, durée estimée, documents acceptés |
| **2 — Type de document** | Sélection CNI / Passeport / Titre de séjour |
| **3 — Scan recto/verso** | Capture guidée, validation qualité image |
| **4 — Selfie biométrique** | Liveness check, score de concordance visage |
| **5 — Confirmation** | Récapitulatif + soumission + affichage du nouveau score MPI |

---

## Portefeuille Citoyen temps réel (v7/v8)

`PortefeuilleCitoyenScreen` affiche l'activité du wallet en temps réel via WebSocket.

- Badge **EN DIRECT** pulsant quand la connexion est active
- Toasts animés sur chaque nouvel événement reçu
- `ACTION_LABELS` map (16 entrées) — aucun nom technique visible côté citoyen
- Filtre par catégorie : Tous · Portefeuille · Santé · Identité

| Événement interne | Label affiché |
| --- | --- |
| `WALLET_ACCESSED` | Connexion au portefeuille |
| `PASS_ISSUED` | Nouveau document émis |
| `CONSENT_GRANTED` | Autorisation accordée |
| `CONSENT_REVOKED` | Autorisation retirée |
| `IDENTITY_SYNCED` | Identité synchronisée |
| `KYC_COMPLETED` | Vérification d'identité terminée |
| *(inconnu)* | Activité |

---

## Registre d'Identités Souveraines (v7/v8)

`RegistreIdentitesScreen` est le tableau de bord de l'identité souveraine du citoyen.

| Élément | Détail |
| --- | --- |
| **Carte IAL animée** | Gradient souverain, barre de progression IAL animée, score 0–100 |
| **Niveaux IAL** | IAL3 — Biométrie certifiée · IAL2 — Document officiel · IAL1 — Téléphone vérifié · IAL0 — Vérification d'identité requise |
| **Vérifications** | Téléphone · Document officiel validé · Biométrie certifiée · Identité certifiée |
| **Services débloqués** | Grid par niveau IAL — santé, paiements, mobilité, éducation |
| **Empreinte cryptographique** | `Empreinte numérique : <SHA-256>` — intégrité du registre |
| **Signalement** | Modal `FlagModal` — 4 types d'erreur, référence de suivi, envoi asynchrone |
| **Source de données** | Registre Officiel · Registre Campagne · Portefeuille seul |

---

## Support bilingue FR / EN

L'application est entièrement traduite en français et en anglais.

| Composant | Détail |
| --- | --- |
| **Bibliothèque** | `i18next` + `react-i18next` + `expo-localization` |
| **Persistance** | `AsyncStorage` — clé `@oomus_language` |
| **Détection** | Langue système de l'appareil au premier lancement, `fr` par défaut |
| **Sélecteur** | Modal FR / EN dans l'écran Paramètres |
| **Couverture** | 450+ clés — tous les écrans, messages d'erreur, formats de date/heure |

---

## Mode hors ligne

Le `SyncContext` maintient un cache local des passes et de l'identité. L'application reste fonctionnelle sans connexion et synchronise automatiquement au retour du réseau. Après tout événement WebSocket, `syncNow()` est déclenché après 1,2 s.

---

## Démarrage

```bash
cd mobile
npm install

# Développement
npx expo start          # QR code Expo Go
npx expo run:ios        # Simulateur iOS
npx expo run:android    # Émulateur Android

# Build cloud (EAS)
eas build --platform ios
eas build --platform android
```

Variable d'environnement requise :

```bash
EXPO_PUBLIC_API_URL=https://api.oomus.org
```

---

## Architecture

```text
mobile/src/
├── i18n/
│   ├── fr.ts                    ← Traductions françaises (450+ clés)
│   ├── en.ts                    ← Traductions anglaises
│   └── index.ts                 ← Init i18next + expo-localization
├── api/
│   ├── client.ts                ← Axios JWT + refresh auto
│   ├── walletApi.ts             ← passes, consents, audit
│   ├── kycApi.ts                ← wizard KYC + finalize
│   └── insuranceApi.ts          ← assurances citoyen
├── auth/
│   └── AuthContext.tsx          ← JWT, PIN, biométrie
├── context/
│   └── SyncContext.tsx          ← Cache offline + sync WebSocket
├── features/
│   └── identity/
│       ├── KycFlowScreen.tsx    ← Wizard KYC 5 étapes
│       └── types.ts             ← KycFlowStep, KycSubmitPayload…
├── navigation/
│   └── AppNavigator.tsx
├── screens/
│   ├── WalletHomeScreen.tsx
│   ├── PortefeuilleCitoyenScreen.tsx
│   ├── RegistreIdentitesScreen.tsx
│   ├── PassesScreen.tsx
│   ├── AuditScreen.tsx
│   ├── ConsentsScreen.tsx
│   ├── InsuranceScreen.tsx
│   └── SettingsScreen.tsx
├── components/
│   └── SovereignCard.tsx
└── theme/
    └── index.ts                 ← Tokens couleurs + getPassConfig()
```
