# OOMUS Wallet — Application mobile

**React Native 0.81.5 · Expo SDK 54 · TypeScript · iOS + Android**

L'application mobile OOMUS Wallet permet à chaque citoyen d'accéder à son identité souveraine MPI, de consulter ses passes de santé, de contrôler le partage de ses données et de vérifier sa couverture d'assurance — directement depuis son téléphone.

---

## Authentification

| Étape | Mécanisme |
|---|---|
| **Connexion** | Numéro de téléphone → OTP SMS (30 s resend) |
| **PIN** | 6 chiffres, stocké dans `SecureStore` (iOS Keychain / Android Keystore) |
| **Biométrie** | Face ID (iOS) ou Fingerprint (Android) — verrouillage automatique en arrière-plan |
| **Session** | JWT access + refresh, rotation automatique via `AuthContext` |

---

## Écrans

| Écran | Description |
|---|---|
| **LoginScreen** | Saisie numéro de téléphone, validation format, indicatif pays |
| **OtpScreen** | Code OTP 6 chiffres, auto-avancement, collage automatique |
| **PinScreen** | Création et vérification PIN — 6 cases animées |
| **WalletHomeScreen** | `SovereignCard` boarding-pass, résumé MPI, actions rapides, activité récente |
| **PassesScreen** | Grille des passes de santé, filtres par statut, pull-to-refresh |
| **PassDetailScreen** | Détail pass + QR HMAC, slider TTL partage temporaire |
| **ConsentsScreen** | Consentements actifs + révocation. GrantModal 2 étapes |
| **AuditScreen** | Journal d'activité chronologique — filtres par catégorie |
| **InsuranceScreen** | Couvertures, vérification d'actes médicaux, historique des vérifications |
| **SettingsScreen** | Profil MPI, sécurité (PIN · biométrie), langue, déconnexion |

---

## Support bilingue FR / EN

L'application est entièrement traduite en français et en anglais.

| Composant | Détail |
|---|---|
| **Bibliothèque** | `i18next` + `react-i18next` + `expo-localization` |
| **Persistance** | `AsyncStorage` — clé `@oomus_language` |
| **Détection** | Langue système de l'appareil au premier lancement, `fr` par défaut |
| **Sélecteur** | Modal 🇫🇷 / 🇬🇧 dans l'écran Paramètres |
| **Couverture** | 450+ clés — tous les écrans, messages d'erreur, formats de date/heure |

---

## Mode hors ligne

Le `SyncContext` maintient un cache local des passes et de l'identité. L'application reste fonctionnelle sans connexion et synchronise automatiquement au retour du réseau.

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

**Variable d'environnement requise :**

```bash
EXPO_PUBLIC_API_URL=https://api.oomus.org
```

---

## Architecture

```text
mobile/src/
├── i18n/
│   ├── fr.ts           ← Traductions françaises
│   ├── en.ts           ← Traductions anglaises
│   └── index.ts        ← Init i18next + expo-localization
├── api/
│   ├── client.ts       ← Axios JWT + refresh auto
│   ├── walletApi.ts    ← passes, consents, audit
│   └── insuranceApi.ts ← assurances citoyen
├── auth/
│   └── AuthContext.tsx ← JWT, PIN, biométrie
├── context/
│   └── SyncContext.tsx ← Cache offline
├── navigation/
│   └── AppNavigator.tsx
├── screens/            ← 10 écrans
├── components/
│   └── SovereignCard.tsx
└── theme/
    └── index.ts        ← Tokens couleurs + getPassConfig()
```
