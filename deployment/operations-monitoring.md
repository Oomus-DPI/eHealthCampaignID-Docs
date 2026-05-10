# Operations & Monitoring

## Surveiller

- Disponibilite API.
- Latence p95 des endpoints.
- Etat PostgreSQL.
- Etat Redis.
- Queue Celery.
- Jobs echoues.
- Jobs DHIS2 et synchronisations.
- Envois WhatsApp/SMS.
- Emissions Google Wallet.
- Espace de stockage MinIO/S3.
- Erreurs applicatives.

## Flower

Flower permet de suivre les workers Celery et les taches en cours:

```bash
celery -A app.tasks.generation.celery_app flower --port=5555
```

## Logs utiles

- Logs API Uvicorn.
- Logs worker Celery.
- Logs PostgreSQL.
- Logs MinIO.

## Alertes recommandees

- API indisponible.
- Worker absent.
- Redis indisponible.
- Taux de jobs en echec eleve.
- Retard de synchronisation DHIS2.
- Echec massif de distribution.
- Anomalies de generation detectees.
- Espace stockage faible.
- Erreurs 5xx recurrentes.

## Modules temps reel

- Jobs Studio: generations, progression, annulation, historique.
- Jobs DHIS2: synchronisations, liens de telechargement, erreurs.
- Infrastructure: workers, logs, files, etat systeme.
