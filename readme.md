## GKE Deploy action via Google Artifact Registry

Builds the image, pushes it to Artifact Registry and applies the kustomize
directory to GKE. Authenticates with Workload Identity Federation (no keys)
since v1; v0.x releases take a JSON key instead.

The image is built on the runner's own Docker daemon, so the layer cache
carries over between jobs on a self-hosted runner. It is built for
`linux/amd64` unless `DOCKER_BUILD_ARGS` sets another `--platform`.

Before the push, [Trivy](https://trivy.dev) scans the local image and fails
the deploy on critical vulnerabilities that already have a fix. It runs from
its pinned image, so the only requirement on the runner is Docker. To let a
deploy through while a fix is pending, list the CVE IDs in a `.trivyignore`
at the root of the checkout, one per line.

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
