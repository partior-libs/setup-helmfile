# setup-helmfile
> Install kubectl, Helm, and Helmfile with specified versions and add them to the PATH.

## Overview
This JavaScript (Node.js 12) action installs a pinned set of Kubernetes toolchain components — `kubectl`, `helm`, and `helmfile` — along with optional Helm plugins (`helm-diff`, `helm-s3`, and any additional plugins). Use it at the start of deployment workflows to ensure a reproducible toolchain regardless of what is pre-installed on the runner.

## Usage

```yaml
name: Deploy with Helmfile
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Helmfile toolchain
        uses: partior-libs/setup-helmfile@main
        with:
          kubectl-version: "1.27.3"
          helm-version: "v3.12.0"
          helmfile-version: "v0.155.0"

      - name: Deploy
        run: helmfile apply
        env:
          KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

### Minimal setup (use defaults)

```yaml
      - name: Setup Helmfile (defaults)
        uses: partior-libs/setup-helmfile@main
```

### With additional Helm plugins

```yaml
      - name: Setup Helmfile with extra plugins
        uses: partior-libs/setup-helmfile@main
        with:
          helm-version: "v3.12.0"
          helmfile-version: "v0.155.0"
          additional-helm-plugins: >
            https://github.com/databus23/helm-diff,
            https://github.com/hypnoglow/helm-s3
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `kubectl-version` | ❌ | `1.22.9` | kubectl version to install |
| `kubectl-release-date` | ❌ | `2022-06-03` | Release date used to construct the kubectl download URL |
| `helm-version` | ❌ | `v3.9.0` | Helm version to install |
| `helmfile-version` | ❌ | `v0.145.0` | Helmfile version to install |
| `install-kubectl` | ❌ | `yes` | Whether to install kubectl (`yes`/`no`) |
| `install-helm` | ❌ | `yes` | Whether to install Helm (`yes`/`no`) |
| `install-helm-plugins` | ❌ | `yes` | Whether to install built-in Helm plugins (`yes`/`no`) |
| `helm-diff-plugin-version` | ❌ | `master` | `helm-diff` plugin version or branch |
| `helm-s3-plugin-version` | ❌ | `master` | `helm-s3` plugin version or branch |
| `additional-helm-plugins` | ❌ | — | Comma-separated list of additional Helm plugin URLs to install |

## Outputs
This action does not set step outputs. Tools are added to `PATH` as a side effect.

## Prerequisites
- Linux runner (`ubuntu-latest`) is recommended. macOS support may vary.
- Internet access is required for downloading binaries from GitHub Releases and the Kubernetes CDN.

## Contributing
Commit message format: `git commit -m "<TICKET_NUMBER> <COMMIT_MESSAGE>"`

## License
See [LICENSE](LICENSE). Upstream source: [`mamezou-tech/setup-helmfile`](https://github.com/mamezou-tech/setup-helmfile).
