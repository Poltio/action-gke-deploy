## GKE Deploy action via Google Artifact Registry

Builds the image, pushes it to Artifact Registry and applies the kustomize
directory to GKE. Authenticates with Workload Identity Federation (no keys)
since v1; v0.x releases take a JSON key instead.

The build layer cache is stored in Artifact Registry next to the image as
`<image>:buildcache`, so it survives between jobs and runners.

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
      - uses: actions/checkout@<sha> # the action builds from the caller's checkout
        with:
          persist-credentials: false
      - uses: poltio/action-gke-deploy@<sha> # v1.0.0
        with:
          GOOGLE_CLOUD_PROJECT_ID: <project-id>
          WORKLOAD_IDENTITY_PROVIDER: projects/<project-number>/locations/global/workloadIdentityPools/<pool>/providers/<provider>
          SERVICE_ACCOUNT: <deployer>@<project-id>.iam.gserviceaccount.com
          # ...remaining inputs, see action.yaml
```
