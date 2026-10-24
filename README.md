# stackunderflow-deploy

Reusable GitHub Actions workflow for apps on the stackunderflow.no application
platform (KRO `Application` CRD + app-webhook + Flux).

Builds the Docker image, pushes it to ghcr.io, updates the deployment reference
in the app repo (KRO `defaultMode.image` or a kustomize `app` placeholder for
custom mode) and commits the change back.

## Usage

```yaml
name: Build
on:
  push:
    branches: [main]
jobs:
  deploy:
    uses: bjorngi/stackunderflow-deploy/.github/workflows/deploy.yaml@main
    with:
      image-name: ghcr.io/<owner>/<app>
    permissions:
      contents: write
      packages: write
```

The Docker image must be buildable from the repo root. For custom mode
(`spec.source.git.path`), the kustomization in that path must have an image
placeholder named `app`:

```yaml
images:
  - name: app
    newName: ghcr.io/<owner>/<app>
```
