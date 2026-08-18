# KitOps Pack & Push

A GitHub Action that packs a [ModelKit](https://kitops.org/docs/modelkit/intro/) from your repo's `Kitfile` using the [Kit CLI](https://github.com/kitops-ml/kitops) and pushes it to a registry like [Jozu Hub](https://jozu.ml) or any other OCI compliant registry.

Use this to publish a versioned, portable package of your model, dataset, code, and config as part of your CI/CD pipeline.

> **Prerequisite:** your repo needs a `Kitfile` already. If it doesn't have one, see [`kitfile-generator-action`](https://github.com/nathan-young1/kitfile-generator-action) to add one automatically.

## What it does

1. Installs the Kit CLI on the runner
2. Logs in to your chosen registry using credentials you supply
3. Packs your repo into a ModelKit, tagged as you specify
4. Pushes the ModelKit to the registry

## Usage

Create `.github/workflows/publish-modelkit.yml` in your repo:

```yaml
name: Publish ModelKit

on:
  release:
    types: [published]
  workflow_dispatch:

concurrency:
  group: publish-modelkit

permissions:
  contents: read

jobs:
  pack-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: nathan-young1/kit-pack-push-action@v1
        with:
          registry: jozu.ml # You can put a different registry here.
          repository: your-namespace/repo-name
          tag: ${{ github.event.release.tag_name || 'latest' }}
          username: ${{ secrets.JOZU_USERNAME }}
          password: ${{ secrets.JOZU_PASSWORD }}
```

### Why `on: release`?

We recommend triggering this action when you **publish a GitHub Release**, not on every push. This mirrors how most container images get published, keeps your registry from filling up with a new ModelKit on every commit, and lets the release tag map directly onto the ModelKit tag.
`workflow_dispatch` also is included so you can also trigger a publish manually.

If your AI/ML project needs a different trigger (e.g. every merge to `main`), you can change the `on:` block accordingly.

## Setting up your secrets

This action needs credentials to log in to your registry. **Never hardcode these in your workflow file.**

1. Go to your repo on GitHub
2. **Settings → Secrets and variables → Actions**
3. Click **New repository secret**
4. Add a secret named `JOZU_USERNAME` with your registry's (e.g Jozu Hub) username/email
5. Add a second secret named `JOZU_PASSWORD` with your registry's (e.g Jozu Hub) password or access token
6. Reference them in your workflow exactly as shown in the usage example: `${{ secrets.JOZU_USERNAME }}` / `${{ secrets.JOZU_PASSWORD }}`

If you're pushing to a different OCI registry (not Jozu Hub), use that registry's credentials instead, just make sure the `registry` input in the yaml above matches its hostname.

## Inputs

| Name | Description | Required | Default |
|---|---|---|---|
| `registry` | Registry hostname, e.g. `jozu.ml` | Yes | `jozu.ml` |
| `repository` | Target repository path, e.g. `your-namespace/repo-name` | Yes | — |
| `tag` | Tag for the ModelKit, e.g. `latest` or a version | No | `latest` |
| `username` | Registry username | Yes | — |
| `password` | Registry password or access token | Yes | — |
| `kit-version` | Kit CLI version to install | No | `latest` |

## Permissions

This action only needs to read your repo to pack and push to external registry:

```yaml
permissions:
  contents: read
```

## Concurrency

```yaml
concurrency:
  group: publish-modelkit
```

We recommend a `concurrency` group so that overlapping publish runs will queue rather than run in parallel.

## Tagging

Common patterns:

- **Release-driven:** `tag: ${{ github.event.release.tag_name }}` — each GitHub Release produces a matching ModelKit version. Recommended default.
- **Always latest:** `tag: latest` — simplest, but overwrites the previous ModelKit on every publish.
- **Both:** run the action twice with different tags, or extend the workflow to push a versioned tag and re-tag it as `latest`.

## Troubleshooting

- **Login fails:** double check your secrets are named exactly `JOZU_USERNAME` / `JOZU_PASSWORD` (or whatever you passed to `username`/`password`) and that the account has push access to the target `repository`.
- **Pack fails:** make sure a valid `Kitfile` exists in the repo root before this action runs. See [`kitfile-generator-action`](https://github.com/nathan-young1/kitfile-generator-action) if you need one generated.
- **Push fails with a permissions/404 error:** confirm the `repository` path exists (or that your account/org is allowed to create it) on the target registry.

## License

[Put the licence Kitops or Jozu wants to use here]

## Feedback / Issues

Please file issues in this repository. For questions about the Kit CLI itself, see the [main KitOps repository](https://github.com/kitops-ml/kitops).