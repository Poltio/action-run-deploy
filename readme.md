## Deploy to Google Cloud Run from Google Artifact Registry

Builds the image, pushes it to Artifact Registry and deploys it to an existing
Cloud Run service, tagging the revision with `IMAGE_TAG` (dots become dashes).
Authenticates with Workload Identity Federation (no keys) since v1.

The service account needs `roles/artifactregistry.writer` on the registry
repository (plus reader on any repository your Dockerfile pulls base images
from), `roles/run.developer` on the Cloud Run service,
`roles/iam.serviceAccountUser` on the service's runtime service account, and
`roles/iam.workloadIdentityUser` granted to your repository through your
Workload Identity Provider.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write # Workload Identity Federation login to GCP
    steps:
      - id: deploy
        uses: poltio/action-run-deploy@<sha> # v1.0.0
        with:
          GOOGLE_CLOUD_PROJECT_ID: <project-id>
          WORKLOAD_IDENTITY_PROVIDER: projects/<project-number>/locations/global/workloadIdentityPools/<pool>/providers/<provider>
          SERVICE_ACCOUNT: <deployer>@<project-id>.iam.gserviceaccount.com
          DOCKER_BUILD_ARGS: --build-arg env=production
          ARTIFACT_REGISTRY_REGION: europe-west1
          ARTIFACT_REGISTRY_REPO: my-repo
          ARTIFACT_REGISTRY_IMAGE: my-image
          IMAGE_TAG: ${{ github.ref_name }}
          CLOUD_RUN_SERVICE_NAME: my-service
          CLOUD_RUN_REGION: europe-west1
```

The deployed URL is available as `steps.deploy.outputs.PREVIEW_URL`.
