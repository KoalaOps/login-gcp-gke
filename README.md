# Login GCP GKE

Authenticates to Google Cloud Platform and configures kubectl for GKE clusters.

## Features

- 🔐 **Workload Identity Federation** - Secure keyless authentication
- 🔑 **Service Account Keys** - Traditional authentication
- ☸️ **GKE Integration** - Automatic kubectl configuration  
- 🐳 **GAR Support** - Google Artifact Registry login
- 🔄 **Multi-cluster** - Handle multiple GKE clusters

## Usage

```yaml
- name: Login to GCP and GKE
  uses: KoalaOps/login-gcp-gke@v1
  with:
    workload_identity_provider: ${{ vars.WIF_PROVIDER }}
    service_account: ${{ vars.WIF_SERVICE_ACCOUNT }}
    project_id: my-project
    location: us-central1
    cluster_name: production
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `project_id` | GCP project ID | ✅ | - |
| `location` | GKE cluster location/region | ✅ | - |
| `cluster_name` | GKE cluster name | ✅ | - |
| `workload_identity_provider` | WIF provider | ❌* | - |
| `service_account` | Service account email | ❌* | - |
| `credentials_json` | Service account JSON key | ❌* | - |
| `gar_location` | GAR location (defaults to cluster location) | ❌ | cluster location |
| `skip_gar_login` | Skip GAR Docker login | ❌ | `false` |

*Use either Workload Identity (provider + service_account) OR credentials_json

## Outputs

| Output | Description |
|--------|-------------|
| `project_id` | GCP project ID |
| `kubectl_context` | Kubernetes context name |
| `gar_registry` | GAR registry URL if configured |

## Examples

### Workload Identity Federation (Recommended)
```yaml
- name: GCP auth with WIF
  uses: KoalaOps/login-gcp-gke@v1
  with:
    workload_identity_provider: projects/123456789/locations/global/workloadIdentityPools/github/providers/github
    service_account: github-actions@my-project.iam.gserviceaccount.com
    project_id: my-project
    location: us-central1
    cluster_name: prod-cluster
```

### Service Account Key
```yaml
- name: GCP auth with key
  uses: KoalaOps/login-gcp-gke@v1
  with:
    credentials_json: ${{ secrets.GCP_CREDENTIALS }}
    project_id: my-project
    location: europe-west1
    cluster_name: staging-cluster
```

### With GAR Docker Login
```yaml
- name: GCP with container registry
  uses: KoalaOps/login-gcp-gke@v1
  with:
    workload_identity_provider: ${{ vars.WIF_PROVIDER }}
    service_account: ${{ vars.WIF_SA }}
    project_id: my-project
    location: us-central1
    cluster_name: prod
    skip_gar_login: false  # Enable GAR login

- name: Push to GAR
  run: |
    docker build -t us-docker.pkg.dev/my-project/images/app:latest .
    docker push us-docker.pkg.dev/my-project/images/app:latest
```

### With Custom GAR Location
```yaml
- name: GCP with different GAR location
  uses: KoalaOps/login-gcp-gke@v1
  with:
    workload_identity_provider: ${{ vars.WIF_PROVIDER }}
    service_account: ${{ vars.WIF_SA }}
    project_id: my-project
    location: us-central1           # GKE cluster in us-central1
    cluster_name: prod
    gar_location: europe-west1       # GAR in europe-west1
    skip_gar_login: false

- name: Push to European GAR
  run: |
    docker build -t europe-docker.pkg.dev/my-project/images/app:latest .
    docker push europe-docker.pkg.dev/my-project/images/app:latest
```

### GCP Only (No GKE)
```yaml
- name: GCP auth only
  uses: KoalaOps/login-gcp-gke@v1
  with:
    credentials_json: ${{ secrets.GCP_CREDENTIALS }}
    project_id: my-project
    location: us-central1
    # No cluster_name - just GCP auth
```

## Prerequisites

### For Workload Identity
1. Create Workload Identity Pool
2. Configure provider for GitHub
3. Create service account with necessary permissions
4. Add `id-token: write` permission to workflow

### For Service Account Key
1. Create service account
2. Generate JSON key
3. Store as GitHub secret

## Permissions Required

### GCP Permissions
- `iam.serviceAccounts.getAccessToken` (for WIF)
- `container.clusters.get` (for GKE)
- `container.clusters.list` (for GKE)

### GitHub Permissions
```yaml
permissions:
  id-token: write  # For WIF
  contents: read
```

## Notes

- Workload Identity is more secure than service account keys
- GAR login configures Docker for the appropriate regional registry
- Works with both regional and zonal GKE clusters
- Compatible with Autopilot and Standard GKE clusters