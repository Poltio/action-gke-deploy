## GKE Deploy action via Google Artifact Registry

Builds the image, pushes it to Artifact Registry and applies the kustomize
directory to GKE. Authenticates with Workload Identity Federation (no keys)
since v1; v0.x releases take a JSON key instead.

The service account needs `roles/artifactregistry.writer` on the registry
repository (plus reader on any repository your Dockerfile pulls base images
from), `roles/container.developer`, and `roles/iam.workloadIdentityUser`
granted to your repository through your Workload Identity Provider.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write # Workload Identity Federation login to GCP
    steps:
      - uses: poltio/action-gke-deploy@<sha> # v1.0.0
        with:
          GOOGLE_CLOUD_PROJECT_ID: <project-id>
          WORKLOAD_IDENTITY_PROVIDER: projects/<project-number>/locations/global/workloadIdentityPools/<pool>/providers/<provider>
          SERVICE_ACCOUNT: <deployer>@<project-id>.iam.gserviceaccount.com
          # ...remaining inputs, see action.yaml
```
