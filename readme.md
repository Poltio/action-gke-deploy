## GKE Deploy action via Google Artifact Registry

Builds the image, pushes it to Artifact Registry and applies the kustomize
directory to GKE. Authenticates with Workload Identity Federation (no keys)
since v1; v0.x releases take a JSON key instead.

All repos deploy as the shared `gke-deployer` service account. Every Poltio
repo may impersonate it (`roles/iam.workloadIdentityUser` is bound to the whole
`github` pool, whose provider only accepts the Poltio org), so a new repo needs
no GCP setup.

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
          GOOGLE_CLOUD_PROJECT_ID: poltio-164412
          WORKLOAD_IDENTITY_PROVIDER: projects/50670027198/locations/global/workloadIdentityPools/github/providers/poltio
          SERVICE_ACCOUNT: gke-deployer@poltio-164412.iam.gserviceaccount.com
          # ...remaining inputs, see action.yaml
```
