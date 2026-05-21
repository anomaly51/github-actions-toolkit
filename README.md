# GitHub Actions Toolkit

Reusable workflow building blocks for application release pipelines.

## Workflows

- `.github/workflows/argocd-app-release.yml` - builds or promotes a container image,
  publishes it, updates a Kubernetes deployment manifest in a GitOps repository,
  synchronizes the deployment when Argo CD credentials are provided, waits for
  the live application health/version check, and sends Telegram release
  notifications.
- `.github/workflows/fluxcd-image-release.yml` - builds or promotes a container
  image, publishes it with a Flux Image Automation-compatible tag, optionally
  waits for the live health endpoint, and sends Telegram release notifications.
  It does not edit GitOps manifests directly; FluxCD image automation owns that
  part.

## Actions

- `.github/actions/notify-telegram` - sends start/final release notifications.
- `.github/actions/deployment-manifest-update` - updates an image reference and
  selected environment variables in a Kubernetes deployment manifest.
- `.github/actions/sync-deployment` - triggers and waits for a deployment sync.
- `.github/actions/await-live-version` - waits until a health endpoint reports
  the expected version, or just returns healthy when no version field is set.

## Versioning

Use immutable tags from application repositories, for example:

```yaml
jobs:
  deploy:
    uses: anomaly51/github-actions-toolkit/.github/workflows/argocd-app-release.yml@v9
    with:
      concurrency-group: my-application
```

`concurrency-group` maps directly to GitHub Actions `concurrency.group`.
Use one stable group per application, for example `architect-business-bot`, so
newer runs can cancel stale runs for the same app without affecting other
applications.

Defaults:

- `gitops-branch` defaults to `main`.
- `deployment-app-name` defaults to `concurrency-group`.
- `deployment-container-name` defaults to the first container image in the
  deployment manifest. Set it only for multi-container deployments.
- `version-env-name` can be empty when the application does not expose a live
  deployment version.
- `health-version-field` can be empty when the workflow should wait for HTTP
  health only.
- `source-image-ref` can be set when the release should promote an existing
  image tag instead of running a local Docker build.
- `platform` and `smoke-test-command` are optional build checks for images that
  need a fixed runtime platform or container smoke test before push.

FluxCD release workflows use the same `concurrency-group` and image naming
inputs, but they only publish the image. The tag format is
`main-<run-number>-<sha>`, which matches Flux image policies using a numerical
`main-(?P<build>[0-9]+)-.*` filter.
