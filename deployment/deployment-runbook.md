# Deployment Runbook

## Demarrage Docker

```bash
cp .env.example .env
docker compose up -d
```

## Verification

```bash
curl http://localhost:8000/health
```

## Services

- API: `http://localhost:8000`
- Swagger: `http://localhost:8000/docs`
- MinIO Console: `http://localhost:9001`
- Flower: `http://localhost:5555`

## Variables critiques

- `DATABASE_URL`
- `CELERY_BROKER_URL`
- `CELERY_RESULT_BACKEND`
- `SECRET_KEY`
- `ADMIN_PASSWORD`
- `STORAGE_BACKEND`
- `S3_BUCKET_NAME`

## Mise en production

Avant production:

- Remplacer tous les secrets par des valeurs fortes.
- Desactiver les mots de passe de demonstration.
- Configurer HTTPS.
- Configurer un stockage S3 durable.
- Activer les sauvegardes PostgreSQL.
- Ajouter monitoring et alerting.
- Configurer Nginx reverse proxy.
- Activer Cloudflare DNS, SSL et WAF.
- Configurer UFW et Fail2ban.
- Configurer les credentials DHIS2, WhatsApp, SMS et Google Wallet via secret manager.
- Activer GitHub Actions pour build Docker, push registry, deploiement SSH et migrations.

## Infrastructure recommandee

- Nginx Reverse Proxy.
- Cloudflare.
- PostgreSQL 16.
- Redis 7.
- Celery Workers.
- Celery Beat.
- AWS S3 ou DigitalOcean Spaces.
- Flower ou monitoring equivalent.
