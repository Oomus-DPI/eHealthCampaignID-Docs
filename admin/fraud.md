# Détection Fraude IA — Vue admin

La section **Détection Fraude IA** donne aux administrateurs une vue globale des anomalies détectées sur l'ensemble de la plateforme, tous programmes confondus. Elle s'appuie sur le module `ai_fraud_detection` combinant règles heuristiques et modèle ML (IsolationForest) pour scorer chaque événement suspect.

> Ce module est disponible pour les programmes disposant du module `ai_fraud_detection` activé. La vue admin agrège les alertes de **tous** les programmes.

---

## Architecture de détection

```
Scan QR → Portail de vérification
         ↓
    Hash du code (SHA-256 64 caractères)
         ↓
    Lookup dans ScanAnomalyLog
         ↓
  ┌──────────────────────────────┐
  │  Règles heuristiques         │  risk_score = f(scan_count, fréquence, IP)
  │  + Modèle IsolationForest    │  ml_score = auto_score(features)
  └──────────────────────────────┘
         ↓
  risk_score = MAX(stored_score, ml_score)
         ↓
  Alerte enregistrée en DB (upsert par code_hash)
```

---

## Tableau de bord admin — Statistiques globales

```bash
GET /api/v1/fraud/admin/summary?days=30
Authorization: Bearer <ADMIN_TOKEN>
```

**Réponse :**

```json
{
  "period_days": 30,
  "total_anomalies": 142,
  "total_scan_events": 4870,
  "avg_risk_score": 0.381,
  "high_risk_count": 23
}
```

| Métrique | Description |
|---|---|
| `total_anomalies` | Codes QR uniques signalés comme suspects |
| `total_scan_events` | Nombre total de scans enregistrés sur la période |
| `avg_risk_score` | Score de risque moyen (0.0 = sûr, 1.0 = très suspect) |
| `high_risk_count` | Anomalies avec `risk_score ≥ 0.7` |

---

## Liste des alertes (admin global)

```bash
GET /api/v1/fraud/admin/alerts?limit=100&min_risk=0.5
Authorization: Bearer <ADMIN_TOKEN>
```

Filtre `min_risk` : retourne uniquement les alertes dont le score dépasse le seuil. Par défaut `0.0` (toutes).

**Champs de chaque alerte :**

| Champ | Type | Description |
|---|---|---|
| `id` | string | Identifiant unique de l'alerte |
| `programme_id` | string | Programme auquel appartient le code scanné |
| `code_partial` | string | Fragment du code (4 premiers caractères) |
| `anomaly_type` | string | Type d'anomalie détectée |
| `scan_count` | integer | Nombre de tentatives de scan de ce code |
| `risk_score` | float | Score de risque final (0.0 – 1.0) |
| `ip_address` | string | Adresse IP de la dernière tentative |
| `first_seen_at` | datetime | Première occurrence |
| `last_seen_at` | datetime | Dernière occurrence |

---

## Types d'anomalies

| Type | Description | Indicateur |
|---|---|---|
| `brute_force` | Code scanné très fréquemment en peu de temps | scan_count élevé, fenêtre courte |
| `invalid_hmac` | Signature HMAC du QR invalide | Falsification possible |
| `replay_attack` | Réutilisation d'un code déjà utilisé | Code à usage unique re-présenté |
| `expired_code` | Code expiré présenté à la vérification | Tentative d'utilisation post-expiration |
| `geo_anomaly` | Scan depuis une géolocalisation inattendue | Combiné avec le module Geospatial |
| `unknown` | Anomalie non catégorisée (IsolationForest) | Détectée par le modèle ML uniquement |

---

## Scores de risque

| Plage | Niveau | Action recommandée |
|---|---|---|
| 0.0 – 0.3 | Faible | Journalisation, pas d'action |
| 0.3 – 0.5 | Modéré | Surveillance renforcée |
| 0.5 – 0.7 | Élevé | Alerte programme, investigation |
| 0.7 – 1.0 | Critique | Blocage IP recommandé, escalade |

---

## Signaler une anomalie manuellement

Les programmes peuvent signaler des anomalies détectées côté client (portail offline, agent de terrain) :

```bash
POST /api/v1/fraud/report
Authorization: Bearer <PROGRAMME_TOKEN>
```

```json
{
  "code_hash": "a3f1b2c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2",
  "code_partial": "DKR-",
  "anomaly_type": "brute_force",
  "ip_address": "196.1.2.3",
  "risk_score": 0.85,
  "campaign_id": "camp_01HXYZ"
}
```

Le système effectue un **upsert** : si le même `code_hash` a déjà été signalé, le compteur de scans est incrémenté et le `risk_score` est mis à jour avec le maximum (`MAX(stored, nouveau, ml_score)`).

---

## Archiver une alerte

```bash
DELETE /api/v1/fraud/alerts/{alert_id}
Authorization: Bearer <PROGRAMME_TOKEN>
```

L'alerte est définitivement supprimée du registre. Les admins peuvent archiver des alertes de n'importe quel programme via leur token admin.

---

## Répartition par type d'anomalie

```bash
GET /api/v1/fraud/by-type
Authorization: Bearer <PROGRAMME_TOKEN>
```

```json
[
  {"type": "brute_force", "count": 89, "avg_risk": 0.74},
  {"type": "invalid_hmac", "count": 31, "avg_risk": 0.91},
  {"type": "expired_code", "count": 18, "avg_risk": 0.42},
  {"type": "unknown", "count": 4, "avg_risk": 0.61}
]
```

---

## Intégration avec le Billing Ledger

Les événements de fraude détectés sont corrélés avec les transactions de facturation pour identifier :
- Les programmes avec un ratio anomalie/génération élevé
- Les campagnes ciblées par des attaques de falsification répétées
- Les patterns de fraude inter-programmes (visible admin uniquement)

---

## Prochaines étapes

- [Portail de vérification (utilisateur)](../features/verification.md)
- [Sécurité — Vue d'ensemble](../security/overview.md)
- [Garanties cryptographiques](../security/cryptographic-guarantees.md)
- [Référence API — Modules Enterprise](../api-reference/enterprise-modules.md)
