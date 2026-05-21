# GitHub Actions Toolkit

Reusable workflow building blocks for application release pipelines.

## Workflows

- `.github/workflows/app-release.yml` - builds a container image, publishes it,
  updates a Kubernetes deployment manifest in a GitOps repository, synchronizes
  the deployment, waits for the live application to report the expected version,
  and sends Telegram release notifications.

## Actions

- `.github/actions/notify-telegram` - sends start/final release notifications.
- `.github/actions/deployment-manifest-update` - updates an image reference and
  selected environment variables in a Kubernetes deployment manifest.
- `.github/actions/sync-deployment` - triggers and waits for a deployment sync.
- `.github/actions/await-live-version` - waits until a health endpoint reports
  the expected version.

## Versioning

Use immutable tags from application repositories, for example:

```yaml
jobs:
  deploy:
    uses: anomaly51/github-actions-toolkit/.github/workflows/app-release.yml@v4
    with:
      concurrency-group: my-application
```

`concurrency-group` maps directly to GitHub Actions `concurrency.group`.
Use one stable group per application, for example `architect-business-bot`, so
newer runs can cancel stale runs for the same app without affecting other
applications.
