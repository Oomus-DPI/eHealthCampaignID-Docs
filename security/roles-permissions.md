# Gestion des rôles, permissions et validations

CampaignID intègre une couche RBAC pour les déploiements institutionnels.

## Rôles fournis

- `programme_user` : accès programme standard, consultation facturation, demande de validation.
- `operator` : opérateur métier, demande de validation.
- `validator` : validation des opérations sensibles.
- `finance_admin` : gestion facturation et validation financière.
- `security_admin` : administration des rôles, permissions et journaux.
- `super_admin` : accès complet, compatible avec l’ancien champ `is_admin`.

## Permissions fines

Les permissions sont exposées par `GET /api/v1/security/permissions`.
Les rôles sont exposés par `GET /api/v1/security/roles`.

Un administrateur sécurité peut affecter un rôle et des permissions explicites via :

```http
PATCH /api/v1/security/programmes/{programme_id}/role
```

## Audit utilisateur

Les actions authentifiées en écriture (`POST`, `PUT`, `PATCH`, `DELETE`) sont journalisées avec :

- utilisateur,
- action,
- ressource,
- statut,
- adresse IP,
- user-agent,
- métadonnées.

Consultation :

```http
GET /api/v1/security/audit-logs
```

## Validation multi-niveaux

Les programmes peuvent soumettre une demande de validation :

```http
POST /api/v1/security/approvals
```

Les validateurs approuvent ou rejettent :

```http
POST /api/v1/security/approvals/{approval_id}/approve
POST /api/v1/security/approvals/{approval_id}/reject
```

L’auto-validation est interdite. Une demande passe à `approved` quand le nombre de niveaux requis est atteint.
