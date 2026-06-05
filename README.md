# GitHub Actions Toolkit

Reusable GitHub Actions workflows for apps deployed through Harbor, Argo CD
Image Updater, and Argo CD.

## Harbor ArgoCD App

Use `.github/workflows/harbor-argocd-app.yml` from application repositories when
the app:

- builds a Docker image;
- pushes it to Harbor with tag `sha-<12-char-git-sha>`;
- lets Argo CD Image Updater write that image tag into the GitOps repository;
- waits until Argo CD reports the exact pushed image as `Synced` and `Healthy`.

Minimal caller workflow:

```yaml
name: Build and deploy

on:
  pull_request:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: my-app-deploy-${{ github.ref }}
  cancel-in-progress: true

jobs:
  deploy:
    uses: anomaly51/github-actions-toolkit/.github/workflows/harbor-argocd-app.yml@main
    with:
      app-name: my-argocd-app
      image-repository: harbor.api-api-api.com/my-project/my-image
    secrets: inherit
```

Optional inputs:

- `runner`: defaults to `arc-runner-set`.
- `registry`: defaults to `harbor.api-api-api.com`.
- `build-cache-repository`: shared BuildKit cache repository. The workflow
  stores one cache tag per GitHub repository, so the same repository can be
  reused by frontend, backend, and bot pipelines.
- `buildkit-endpoint`: defaults to the in-cluster remote BuildKit service.
- `build-context`: defaults to `.`.
- `dockerfile`: defaults to `Dockerfile`.
- `platforms`: defaults to `linux/amd64`.
- `deploy-timeout-seconds`: defaults to `1200`.
- `deploy-poll-seconds`: defaults to `5`.
- `argocd-cli-version`: defaults to `v3.4.3`.
- `build-args`: extra Docker build args, one `KEY=VALUE` per line.

Expected caller secrets:

- Harbor push: `HARBOR_USERNAME`/`HARBOR_PASSWORD` or
  `HARBOR_PUSH_USERNAME`/`HARBOR_PUSH_PASSWORD`.
- Argo CD: prefer `ARGOCD_USERNAME` and `ARGOCD_PASSWORD`; `ARGOCD_AUTH_TOKEN`
  is also supported. `ARGOCD_SERVER` is required.
- `ARGOCD_INSECURE` is optional and defaults to insecure/grpc-web mode unless
  set to `false`.

The workflow intentionally verifies the deployed image reference, not just a
successful build. A run is green only after Argo CD summary images contain the
same Harbor image digest tag that this run pushed.
