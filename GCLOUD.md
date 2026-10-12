1. **Grant Storage Object Viewer on the build sources bucket:** Fixes storage.objects.get permission denied.

```bash
gsutil iam ch serviceAccount:303879540383-compute@developer.gserviceaccount.com:objectViewer gs://run-sources-custom-zenerp-southamerica-east1
```

_How to verify:_ Run `gsutil iam get gs://run-sources-custom-zenerp-southamerica-east1` to ensure the service account is listed.

2. **Grant Artifact Registry Writer role:** Fixes artifactregistry.repositories.uploadArtifacts denied.

```bash
gcloud projects add-iam-policy-binding custom-zenerp \
    --member="serviceAccount:303879540383-compute@developer.gserviceaccount.com" \
    --role="roles/artifactregistry.writer"
```

_How to verify:_ Run `gcloud projects get-iam-policy custom-zenerp` and check that the binding is present.

3. **Grant Secret Manager Secret Accessor role:** Fixes permission denied on secret versions.

```bash
gcloud secrets add-iam-policy-binding zen-token \
    --member="serviceAccount:303879540383-compute@developer.gserviceaccount.com" \
    --role="roles/secretmanager.secretAccessor"
```

_How to verify:_ Run `gcloud secrets get-iam-policy zen-token` to confirm the role assignment.

4. **Enable required Workload Identity APIs:** Prerequisite for WIF.

```bash
gcloud services enable \
    iam.googleapis.com \
    iamcredentials.googleapis.com \
    sts.googleapis.com \
    --project="custom-zenerp"
```

_How to verify:_ Run `gcloud services list --enabled --filter="config.name:iam.googleapis.com"`.

5. **Create Workload Identity Pool:** GitHub pool creation.

```bash
gcloud iam workload-identity-pools create "github-zenerp-br-custom" \
    --project="custom-zenerp" \
    --location="global" \
    --display-name="GitHub Actions Pool"
```

_How to verify:_ Run `gcloud iam workload-identity-pools describe github-zenerp-br-custom --location="global" --project="custom-zenerp"`.

6. **Create OIDC Provider for GitHub Organization:** Trusts zenerp-br-custom owner.

```bash
gcloud iam workload-identity-pools providers create-oidc "github-zenerp-br-custom" \
    --project="custom-zenerp" \
    --location="global" \
    --workload-identity-pool="github-zenerp-br-custom" \
    --display-name="GitHub Provider" \
    --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner" \
    --attribute-condition="assertion.repository_owner == 'zenerp-br-custom'" \
    --issuer-uri="https://token.actions.githubusercontent.com"
```

_How to verify:_ Run `gcloud iam workload-identity-pools providers describe github-zenerp-br-custom --workload-identity-pool="github-zenerp-br-custom" --location="global"`.

7. **Bind Workload Identity User to Service Account:** Allows GitHub org to impersonate SA.

```bash
gcloud iam service-accounts add-iam-policy-binding "303879540383-compute@developer.gserviceaccount.com" \
    --project="custom-zenerp" \
    --role="roles/iam.workloadIdentityUser" \
    --member="principalSet://iam.googleapis.com/projects/303879540383/locations/global/workloadIdentityPools/github-zenerp-br-custom/attribute.repository_owner/zenerp-br-custom"
```

_How to verify:_ Run `gcloud iam service-accounts get-iam-policy "303879540383-compute@developer.gserviceaccount.com"`.

8. **Grant Service Account Token Creator role:** Fixes iam.serviceAccounts.getAccessToken denied.

```bash
gcloud iam service-accounts add-iam-policy-binding "303879540383-compute@developer.gserviceaccount.com" \
    --project="custom-zenerp" \
    --role="roles/iam.serviceAccountTokenCreator" \
    --member="principalSet://iam.googleapis.com/projects/303879540383/locations/global/workloadIdentityPools/github-zenerp-br-custom/attribute.repository_owner/zenerp-br-custom"
```

_How to verify:_ Check the service account policy bindings to ensure `tokenCreator` is listed.

9. **Grant Cloud Run Admin role:** Fixes run.services.get denied.

```bash
gcloud projects add-iam-policy-binding custom-zenerp \
    --member="serviceAccount:303879540383-compute@developer.gserviceaccount.com" \
    --role="roles/run.admin"
```

_How to verify:_ Run `gcloud projects get-iam-policy custom-zenerp` to confirm `run.admin` is active.

10. **Grant Service Account User role:** Fixes iam.serviceaccounts.actAs denied.

```bash
gcloud iam service-accounts add-iam-policy-binding "303879540383-compute@developer.gserviceaccount.com" \
    --project="custom-zenerp" \
    --role="roles/iam.serviceAccountUser" \
    --member="serviceAccount:303879540383-compute@developer.gserviceaccount.com"
```

_How to verify:_ Run `gcloud iam service-accounts get-iam-policy "303879540383-compute@developer.gserviceaccount.com"` to verify `serviceAccountUser`.

11.

```yaml
constraint: constraints/iam.allowedPolicyMemberDomains
listPolicy:
  allValues: ALLOW
```

```bash
gcloud resource-manager org-policies set-policy policy.yaml --project=custom-zenerp
```
