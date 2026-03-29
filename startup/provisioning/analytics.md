# Provisioning: GA4 + Google Search Console

**Idempotency:** Check if GA4 property exists, check if Search Console is verified.

---

## GA4 Setup

### Step 1: Enable the Analytics Admin API

This should already be enabled in the GCP project step, but verify:
```bash
gcloud services list --enabled --filter="name:analyticsadmin.googleapis.com" --project={slug} --format="value(name)" 2>/dev/null
```

If not enabled:
```bash
gcloud services enable analyticsadmin.googleapis.com --project={slug}
```

### Step 2: Create GA4 property

Use the Google Analytics Admin API via REST (authenticated with gcloud):

```bash
# Get access token
TOKEN=$(gcloud auth print-access-token)

# List existing properties to check for duplicates
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://analyticsadmin.googleapis.com/v1beta/properties?filter=displayName='{project-name}'" \
  | grep -c "properties" || echo "0"
```

If no matching property exists:
```bash
# Create GA4 property
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{project-name}",
    "timeZone": "Europe/Paris",
    "currencyCode": "EUR",
    "industryCategory": "TECHNOLOGY"
  }' \
  "https://analyticsadmin.googleapis.com/v1beta/properties"
```

Extract the property ID from the response (field: `name`, format: `properties/XXXXXXX`).

### Step 3: Create web data stream

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{project-name} Web",
    "defaultUri": "https://{domain}"
  }' \
  "https://analyticsadmin.googleapis.com/v1beta/properties/{property-id}/dataStreams"
```

Extract the measurement ID from the response (`webStreamData.measurementId`, format: `G-XXXXXXXXXX`).

### Step 4: Store values

- Store `measurementId` in `.env` as `NEXT_PUBLIC_GA4_MEASUREMENT_ID`
- Store `propertyId` and `measurementId` in `project.json` under `services.ga4`

**If API calls fail:** Log the error and inform the user:
> GA4 property creation via API failed. Create it manually at https://analytics.google.com and paste the measurement ID. I'll store it for you.

---

## Google Search Console Setup

### Step 1: Add DNS TXT verification record

```bash
# Add TXT record to Cloud DNS for domain verification
gcloud dns record-sets create {domain}. \
  --type=TXT \
  --ttl=300 \
  --rrdatas='"google-site-verification={verification-token}"' \
  --zone={slug}-zone \
  --project={slug}
```

**Note:** The verification token comes from the Search Console API. Use the Sites API to initiate verification.

### Step 2: Verify via Search Console API

```bash
TOKEN=$(gcloud auth print-access-token)

# Add site to Search Console
curl -s -X PUT \
  -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/webmasters/v3/sites/https%3A%2F%2F{domain}%2F"
```

### Step 3: Verify ownership

The DNS TXT record should auto-verify. Check:
```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/webmasters/v3/sites/https%3A%2F%2F{domain}%2F"
```

**If verification fails:** DNS may not have propagated yet. Log:
> Search Console: site added, verification pending (DNS propagation may take up to 48h). Re-run /startup to check again.

## Output

Log:
- GA4: property created (ID: {property-id}, measurement: {measurement-id}) / failed (manual setup needed)
- Search Console: verified / pending / failed
