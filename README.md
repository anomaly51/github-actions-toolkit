# GitHub Actions Toolkit

Reusable workflow building blocks for application release pipelines.

## Workflows

- `.github/workflows/app-release.yml` - builds a container image, publishes it,
  updates a Kubernetes deployment manifest in a GitOps repository, synchronizes
  the deployment when Argo CD credentials are provided, waits for the live
  application health/version check, and sends Telegram release notifications.

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
    uses: anomaly51/github-actions-toolkit/.github/workflows/app-release.yml@v6
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
- `platform` and `smoke-test-command` are optional build checks for images that
  need a fixed runtime platform or container smoke test before push.
