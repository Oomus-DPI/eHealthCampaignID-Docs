# Portails de Vérification — Vue admin

Les **Portails de Vérification** sont des applications web statiques auto-contenues, générées à la fin de chaque job de génération. Ils permettent à n'importe quel agent de terrain de vérifier l'authenticité d'une carte sanitaire, même sans connexion internet. La vue admin centralise la gestion de tous les portails déployés.

---

## Architecture d'un portail de vérification

```
Job de génération
        │
        ▼ (packaging)
  portail-{campaign_id}/
  ├── index.html           ← Application SPA statique
  ├── registry.json        ← Registre SHA-256 de toutes les cartes
  ├── manifest.json        ← Métadonnées de la campagne
  └── assets/
      ├── verify.js        ← Moteur de vérification (HMAC-SHA256)
      └── style.css
```

Le portail est **auto-suffisant** : il embarque son propre moteur de vérification et n'envoie aucune donnée en dehors du navigateur. La vérification s'effectue 100 % côté client.

---

## Liste des portails déployés (admin)

```bash
GET /api/v1/verify/admin/portals
Authorization: Bearer <ADMIN_TOKEN>
```

**Champs :**

| Champ | Description |
|---|---|
| `portal_id` | Identifiant unique du portail |
| `campaign_id` | Campagne parente |
| `programme_id` | Programme propriétaire |
| `job_id` | Job de génération source |
| `status` | `active` / `revoked` / `expired` |
| `cards_count` | Nombre de cartes couvertes |
| `verifications_count` | Nombre de vérifications enregistrées |
| `last_verification_at` | Dernière vérification |
| `expires_at` | Date d'expiration du portail |
| `deployment_url` | URL publique du portail (si hébergé) |
| `created_at` | Date de création |

---

## Mécanisme de vérification

### Vérification par QR code

```bash
POST /api/v1/verify/qr
Content-Type: application/json

{
  "payload": "OOMUS:SN-DKR-26-9XQ7LM2A:DKR-VAC:2026:a3f1b2..."
}
```

Le moteur :
1. Extrait le `MPI_ID`, le préfixe de campagne et la signature HMAC
2. Vérifie la signature HMAC-SHA256 contre la clé de campagne
3. Consulte le `registry.json` embarqué (hash SHA-256 du payload)
4. Retourne le résultat avec métadonnées de la carte

**Réponse (succès) :**

```json
{
  "valid": true,
  "mpi_id": "SN-DKR-26-9XQ7LM2A",
  "campaign": "Vaccination EPI - Dakar 2026",
  "beneficiary_name": "Fatou Diallo",
  "issued_at": "2026-05-15",
  "status": "authentic"
}
```

### Vérification par référence

```bash
GET /api/v1/verify/ref/{reference_code}
```

Recherche par code alphanumérique court (8 caractères Base36) sans scanner le QR.

### Vérification hors ligne

Le portail statique peut être téléchargé et utilisé sans internet :

1. Télécharger l'archive : `GET /api/v1/jobs/{job_id}/download/portal`
2. Ouvrir `index.html` dans un navigateur
3. Saisir ou scanner le code — la vérification s'effectue localement

---

## Statistiques de vérification — Vue admin

```bash
GET /api/v1/verify/admin/stats?days=30
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "period_days": 30,
  "total_verifications": 48700,
  "successful_verifications": 47950,
  "failed_verifications": 750,
  "unique_cards_verified": 23400,
  "portals_active": 18,
  "verifications_by_channel": {
    "qr_scan": 41200,
    "reference": 6300,
    "offline": 1200
  }
}
```

---

## Statistiques par campagne

```bash
GET /api/v1/verify/campaign/{campaign_id}/stats
```

```json
{
  "campaign_id": "camp_01HXYZ",
  "total_cards": 10000,
  "verified_cards": 7842,
  "verification_rate": 78.4,
  "first_verification_at": "2026-05-16T07:12:00Z",
  "last_verification_at": "2026-05-30T15:34:00Z"
}
```

---

## Révoquer un portail

L'admin peut révoquer un portail de vérification — toutes les futures tentatives de vérification retourneront `status: revoked` :

```bash
PATCH /api/v1/verify/admin/portals/{portal_id}/revoke
Authorization: Bearer <ADMIN_TOKEN>
```

```json
{
  "reason": "Campagne annulée — données non valides",
  "notify_programme": true
}
```

> La révocation est enregistrée dans le `AuditLog` et ne peut pas être annulée. Les portails offline déjà distribués continuent de fonctionner localement — la révocation n'affecte que les vérifications passant par l'API.

---

## Alertes de sécurité — Vérifications suspectes

Le système remonte automatiquement des alertes quand :

| Condition | Seuil | Action |
|---|---|---|
| Même carte vérifiée > 50 fois en 1h | 50 req/h | Alerte `ScanAnomalyLog` + risk_score ≥ 0.7 |
| Vérifications depuis IP inconnue | Géo hors zone campagne | Alerte si module Geospatial actif |
| Signature HMAC invalide | 1 occurrence | Log immédiat + type `invalid_hmac` |
| Code expiré présenté | Date > `expires_at` | Log type `expired_code` |

Ces alertes sont visibles dans la section [Détection Fraude IA](fraud.md).

---

## Déploiement des portails

### Option 1 — CDN intégré Oomus (recommandé)

Le portail est automatiquement hébergé sur le CDN Oomus au format :

```
https://verify.oomus.health/{campaign_prefix}/{portal_id}/
```

### Option 2 — Auto-hébergement

Pour les programmes souverains, le portail peut être hébergé sur l'infrastructure du programme :

1. Télécharger l'archive : `GET /api/v1/jobs/{job_id}/download/portal`
2. Décompresser et héberger sur n'importe quel serveur HTTP statique (Nginx, S3, Azure Static Web)
3. Aucune dépendance backend requise

```bash
# Exemple déploiement Nginx
unzip portail-camp_01HXYZ.zip -d /var/www/verify/
nginx -c /etc/nginx/nginx.conf
```

---

## Prochaines étapes

- [Vérification hors ligne (utilisateur)](../features/verification.md)
- [Détection Fraude IA (admin)](fraud.md)
- [Garanties cryptographiques](../security/cryptographic-guarantees.md)
- [Vérification hors ligne — Sécurité](../security/offline-verification.md)
