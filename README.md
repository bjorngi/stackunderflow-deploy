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
      faro-enabled: true          # optional: frontend telemetry (faro-recipe.md)
    secrets:
      faro_api_key: FARO_API_KEY # repo secret with the faro collector key
    permissions:
      contents: write
      packages: write
```

### Frontend telemetry (Faro)

Apps instrumented with the Grafana Faro Web SDK (faro-recipe.md) get the
SDK config baked into the JS bundle at image build time via `VITE_FARO_*`
Docker build args. `faro-enabled: true` turns it on; the key comes from the
calling repo's secret (mapped to `faro_api_key`) — it's a spam bar by design
(it ships in the public bundle), so the secret is just hygiene. Apps that
don't opt in are completely unaffected (empty build args).

The app's Dockerfile needs matching `ARG VITE_FARO_ENABLED/VITE_FARO_URL/
VITE_FARO_API_KEY` declarations exported as `ENV` before the frontend build
step — see blockbook's Dockerfile for the reference pattern.

The Docker image must be buildable from the repo root. For custom mode
(`spec.source.git.path`), the kustomization in that path must have an image
placeholder named `app`:

```yaml
images:
  - name: app
    newName: ghcr.io/<owner>/<app>
```
