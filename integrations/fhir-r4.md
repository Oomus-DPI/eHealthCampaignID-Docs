# Interopérabilité HL7 FHIR R4

Oomus CampaignID prend en charge le standard **HL7 FHIR R4** (Fast Healthcare Interoperability Resources, Release 4) pour l'échange d'informations d'identité de santé avec d'autres systèmes de santé numériques.

---

## Endpoint FHIR

```
https://api.oomus.health/fhir/r4/
```

L'endpoint FHIR R4 d'Oomus CampaignID expose les identités MPI sous forme de ressources FHIR standardisées, permettant l'interopérabilité avec tout système compatible FHIR R4.

---

## Ce qui est supporté

### Ressources FHIR

| Ressource | Support | Description |
|---|---|---|
| `Patient` | Lecture + Écriture | Identité du bénéficiaire (MPI) |
| `Identifier` | Lecture | Identifiants cross-système (MPI, DHIS2 UID) |

### Opérations FHIR

| Opération | Méthode | Endpoint | Description |
|---|---|---|---|
| Lecture | `GET` | `/fhir/r4/Patient/{mpi_id}` | Obtenir un patient par MPI ID |
| Recherche | `GET` | `/fhir/r4/Patient?identifier=...` | Rechercher par identifiant |
| Création | `POST` | `/fhir/r4/Patient` | Créer un nouveau patient (→ MPI) |
| PIXm Query | `GET` | `/fhir/r4/$cross-reference` | Résolution cross-référence d'identité |

### Requêtes PIXm (Patient Identity Cross-Reference for Mobile)

Le profil **PIXm** (IHE IT Infrastructure) permet de retrouver tous les identifiants connus d'un même patient dans différents systèmes :

```bash
GET /fhir/r4/$cross-reference?sourceIdentifier=dhis2|abcDEF123gh&targetSystem=oomus-mpi
```

**Réponse :**
```json
{
  "resourceType": "Parameters",
  "parameter": [
    {
      "name": "targetIdentifier",
      "valueIdentifier": {
        "system": "https://oomus.health/mpi",
        "value": "SN-DKR-26-9XQ7LM2A"
      }
    }
  ]
}
```

---

## Ressource Patient FHIR R4 — Exemple

Voici un exemple de ressource `Patient` FHIR R4 exposée par l'endpoint Oomus :

```json
{
  "resourceType": "Patient",
  "id": "SN-DKR-26-9XQ7LM2A",
  "meta": {
    "profile": [
      "http://hl7.org/fhir/StructureDefinition/Patient"
    ],
    "lastUpdated": "2026-05-15T09:00:00Z"
  },
  "identifier": [
    {
      "system": "https://oomus.health/mpi",
      "value": "SN-DKR-26-9XQ7LM2A",
      "type": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/v2-0203",
            "code": "MR",
            "display": "Medical Record Number"
          }
        ]
      }
    },
    {
      "system": "https://dhis2.sante.gov.sn",
      "value": "abcDEF123gh",
      "type": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/v2-0203",
            "code": "EI",
            "display": "Employee Number"
          }
        ]
      }
    }
  ],
  "name": [
    {
      "use": "official",
      "family": "Diallo",
      "given": ["Aminata"]
    }
  ],
  "gender": "female",
  "birthDate": "1995-03-15",
  "address": [
    {
      "district": "Dakar",
      "country": "SN"
    }
  ],
  "telecom": [
    {
      "system": "phone",
      "value": "+221771234567",
      "use": "mobile"
    }
  ]
}
```

---

## Systèmes compatibles

L'endpoint FHIR R4 d'Oomus CampaignID est compatible avec les systèmes suivants :

| Système | Type | Cas d'usage |
|---|---|---|
| **OpenHIE** | Infrastructure d'échange de santé | Point d'accès Client Registry / MPI national |
| **HAPI FHIR** | Serveur FHIR open source | Intégration avec un serveur FHIR existant |
| **Google Cloud Healthcare FHIR R4** | Cloud FHIR | Échange avec des systèmes Google Cloud |
| **Azure Health Data Services** | Cloud FHIR | Intégration Azure |
| **OpenMRS** | Système d'information clinique | Partage d'identité patient |
| **iHRIS** | RH santé | Liaison identité patient-agent de santé |

---

## Cas d'usage

### Résolution d'identité cross-programme

Un bénéficiaire s'inscrit dans un programme de vaccination, puis dans un programme de nutrition. Grâce à FHIR PIXm :

1. Le programme nutrition interroge le MPI Oomus : "Ce patient existe-t-il déjà ?"
2. Le MPI répond avec l'identifiant MPI existant (`SN-DKR-26-9XQ7LM2A`)
3. Les deux programmes partagent la même identité souveraine
4. Toutes les cartes générées portent le même identifiant MPI

### Échange national d'informations de santé

Dans le cadre d'un **Système National d'Information Sanitaire (SNIS)** :

1. Oomus CampaignID publie les identités MPI sur l'endpoint FHIR R4
2. L'infrastructure OpenHIE nationale consomme ces données
3. Les autres systèmes (laboratoire, pharmacie, hôpital) peuvent résoudre l'identité d'un patient via PIXm
4. La vision longitudinale du patient est maintenue à travers tous les points de soin

### Intégration avec un registre patient existant

Si votre organisation dispose déjà d'un registre patient FHIR (HAPI FHIR, Google FHIR) :

1. Configurez la synchronisation bidirectionnelle depuis les paramètres Oomus
2. Les nouveaux MPI créés dans Oomus sont publiés automatiquement vers votre registre FHIR
3. Les mises à jour du registre externe peuvent déclencher une mise à jour du MPI Oomus

---

## Authentification de l'endpoint FHIR

L'endpoint FHIR R4 utilise le même système d'authentification JWT qu'Oomus CampaignID :

```bash
# Requête FHIR authentifiée
curl -X GET "https://api.oomus.health/fhir/r4/Patient/SN-DKR-26-9XQ7LM2A" \
  -H "Authorization: Bearer <VOTRE_TOKEN>" \
  -H "Accept: application/fhir+json"
```

Pour les systèmes machine-to-machine (M2M), un **Client Credentials OAuth2 flow** est disponible — contactez le support Oomus pour la configuration.

---

## Limitations actuelles

| Fonctionnalité | Statut |
|---|---|
| Ressource `Patient` | Supportée |
| Ressource `Observation` | Non supportée (feuille de route) |
| Ressource `Immunization` | Non supportée (feuille de route) |
| Ressource `Encounter` | Non supportée |
| Subscriptions FHIR | Non supportées |
| Bundles FHIR | Supportés (lecture uniquement) |

---

## Prochaines étapes

- [Identité souveraine MPI](../features/mpi-sovereign-identity.md)
- [Référence API MPI](../api-reference/mpi.md)
- [Protection des données](../security/data-protection.md)
