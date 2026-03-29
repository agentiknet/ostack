# Provisioning: Cloud Run + Custom Domain + SSL

**Idempotency:** Check if service exists before deploying.

**Note:** This step happens AFTER scaffold code is generated (Phase 3, Step 7), because we need a Dockerfile and app to deploy.

---

## Step 1: Check if Cloud Run service exists

```bash
gcloud run services describe {slug} --region={region} --project={slug} --format="value(status.url)" 2>/dev/null && echo "SERVICE_EXISTS" || echo "SERVICE_MISSING"
```

## Step 2: Deploy to Cloud Run

If missing or if scaffold was just generated:
```bash
gcloud run deploy {slug} \
  --source . \
  --region={region} \
  --project={slug} \
  --allow-unauthenticated \
  --port=3000 \
  --min-instances=0 \
  --max-instances=10 \
  --memory=512Mi \
  --cpu=1
```

The `--source .` flag uses Cloud Build to build the Dockerfile and deploy.

Adjust `--port` based on the framework:
- Next.js: 3000
- FastAPI: 8000
- Hono/Express: 3000
- Rust (Axum): 8080

## Step 3: Verify deployment

```bash
SERVICE_URL=$(gcloud run services describe {slug} --region={region} --project={slug} --format="value(status.url)")
curl -sf "$SERVICE_URL/api/health" -o /dev/null -w "%{http_code}" 2>/dev/null || curl -sf "$SERVICE_URL/health" -o /dev/null -w "%{http_code}" 2>/dev/null || echo "HEALTH_CHECK_FAILED"
```

## Step 4: Map custom domain (if domain provided)

**Skip if:** No domain was provided.

```bash
gcloud run domain-mappings describe --domain={domain} --region={region} --project={slug} 2>/dev/null && echo "MAPPING_EXISTS" || echo "MAPPING_MISSING"
```

If missing:
```bash
gcloud run domain-mappings create \
  --service={slug} \
  --domain={domain} \
  --region={region} \
  --project={slug}
```

SSL is automatically provisioned by Cloud Run when the domain mapping is created and DNS resolves correctly.

## Step 5: Add DNS records for Cloud Run domain mapping

After creating the domain mapping, Cloud Run outputs required DNS records (typically a CNAME or A record). Add them to Cloud DNS:

```bash
# Get the required DNS records from the domain mapping
gcloud run domain-mappings describe --domain={domain} --region={region} --project={slug} --format=json
```

Add the required records to Cloud DNS using `gcloud dns record-sets create`.

## Output

Log:
- Cloud Run service: deployed to {service-url} / already existed
- Health check: passed / failed
- Domain mapping: created / already existed / skipped (no domain)
- SSL: provisioning (auto by Cloud Run)
