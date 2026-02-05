# Deployment

The application runs on Google Cloud Run with Cloud SQL for the database.

## Prerequisites

1. Google Cloud CLI (`gcloud`) installed
2. Docker installed
3. Access to the GCP project `gen-lang-client-0267720655`

## Services

| Service | URL |
|---------|-----|
| Frontend | `champions-frontend-561975502517.us-central1.run.app` |
| Backend | `champions-backend-561975502517.us-central1.run.app` |

## Deploy Backend

```bash
cd backend
gcloud run deploy champions-backend \
  --source . \
  --region us-central1 \
  --project gen-lang-client-0267720655
```

## Deploy Frontend

```bash
cd frontend
gcloud run deploy champions-frontend \
  --source . \
  --region us-central1 \
  --project gen-lang-client-0267720655
```

## Environment Variables

### Backend (Cloud Run)

Set these in Cloud Run or via Secret Manager:

| Variable | Description | Source |
|----------|-------------|--------|
| `DATABASE_URL` | PostgreSQL connection | Secret Manager |
| `JWT_SECRET` | JWT signing key | Secret Manager |
| `SHARED_PASSWORD` | Login password | Secret Manager |
| `GOOGLE_API_KEY` | Gemini API key | Secret Manager |
| `GEMINI_FILE_SEARCH_STORE` | File Search Store ID | Environment |
| `GEMINI_MODEL` | Model name | Environment |
| `ALLOWED_EMAIL_DOMAIN` | Allowed email domain | Environment |

### Frontend (Cloud Run)

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_API_URL` | Backend URL |

## Secret Manager

Secrets are stored in Google Secret Manager:

| Secret Name | Purpose |
|-------------|---------|
| `jwt-secret` | JWT signing key |
| `google-api-key` | Gemini API key |
| `db-password` | Database password |
| `shared-password` | User login password |

To update a secret:
```bash
echo -n "new-value" | gcloud secrets versions add SECRET_NAME --data-file=-
```

## Database

### Cloud SQL Instance

- Type: PostgreSQL 14
- Region: us-central1
- Name: (check Cloud Console)

### Connect to Database

Using Cloud SQL Proxy:
```bash
cloud_sql_proxy -instances=PROJECT:REGION:INSTANCE=tcp:5432
psql -h localhost -U champions -d loan_expert
```

### Run Migrations

SSH into Cloud Run or run locally with DATABASE_URL pointing to Cloud SQL:
```bash
python scripts/run_migration.py
```

## Monitoring

### View Logs

```bash
# Backend logs
gcloud run logs read champions-backend --region=us-central1 --limit=100

# Frontend logs
gcloud run logs read champions-frontend --region=us-central1 --limit=100
```

### Check Service Status

```bash
# Health check
curl https://champions-backend-561975502517.us-central1.run.app/health
```

## Rollback

To rollback to a previous version:

1. Go to Cloud Console > Cloud Run
2. Select the service
3. Go to Revisions tab
4. Select previous revision
5. Click "Manage Traffic"
6. Route 100% to previous revision

## CI/CD

The backend includes `cloudbuild.yaml` for automated builds:

```bash
gcloud builds submit --config cloudbuild.yaml
```

This builds the Docker image and deploys to Cloud Run automatically.

---

## Deployment Checklist

Before deploying to production:

- [ ] All tests passing
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] File Search Store configured
- [ ] CORS origins updated for production
- [ ] Secrets in Secret Manager

## Troubleshooting Deployments

### Build fails

Check build logs:
```bash
gcloud builds log BUILD_ID
```

### Service won't start

Check logs for startup errors:
```bash
gcloud run logs read SERVICE_NAME --region=us-central1 --limit=100
```

### Database connection issues

Verify Cloud SQL connection:
- Check IAM permissions
- Verify connection string
- Ensure Cloud SQL Admin API is enabled
