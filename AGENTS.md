# AGENTS.md — setup-helmfile

## Purpose
JavaScript GitHub Action (Node.js 12) that installs a versioned Kubernetes toolchain — kubectl, Helm, Helmfile, and optional Helm plugins — and adds them to the runner's PATH. Ensures reproducible deployments by pinning all tool versions.

## Repository Map
```
setup-helmfile/
├── action.yml              # Action metadata — node12, main: dist/index.js
├── src/
│   ├── index.js            # Entry point
│   └── setup.js            # Core install logic for each tool
│   └── setup.test.js       # Tests
├── dist/
│   └── index.js            # Bundled output (committed)
├── package.json
└── README.md
```

## Tech Stack
- **Runtime**: Node.js 12 (`node12`)
- **Language**: JavaScript
- **Tool downloads**: `@actions/tool-cache` (download, extract, cache)
- **PATH management**: `@actions/core.addPath()`
- **Author**: mamezou-tech

## Architecture Patterns
- `src/setup.js` installs each tool in sequence: kubectl → Helm → Helmfile → plugins.
- Each tool is downloaded from its official release URL, extracted, and added to PATH.
- `@actions/tool-cache.find()` checks for a cached version before downloading.
- Helm plugins are installed via `helm plugin install <url>` after Helm is available on PATH.
- `install-kubectl`, `install-helm`, `install-helm-plugins` flags allow selective installation.

## Development Commands
```bash
npm install             # Restore dependencies
npm run build           # Bundle → dist/index.js
npm test                # Run tests (src/setup.test.js)
npm run lint            # Lint

# Test specific tool install
INPUT_HELM_VERSION=v3.12.0 node dist/index.js
```

## Environment Setup
- Node.js 12+ (action); 16+ for development.
- Internet access for binary downloads.
- Linux runner for full compatibility.

## Coding Conventions
- Plain JavaScript; follow existing style in `src/`.
- `yes`/`no` string inputs (not boolean) — compare with `=== 'yes'`.
- Tool version variables should be clearly named and validated before URL construction.
- Keep `dist/index.js` committed and in sync with `src/`.

## Testing
```bash
npm test                    # Jest/Mocha unit tests
```
- Test each tool installs to PATH correctly.
- Test `install-kubectl: no` skips kubectl installation.
- Test `additional-helm-plugins` with multiple comma-separated URLs.
- Test version string validation (rejects invalid semver).
- Integration test: run `kubectl version --client`, `helm version`, `helmfile version` after install.

## Key Abstractions
- **`@actions/tool-cache`** — handles download, extraction, and caching for all tools.
- **`install-*` flags** — allow workflows to opt out of tools they don't need (faster setup).
- **`additional-helm-plugins`** — extensible plugin installation without modifying the action.
- **Version pinning** — defaults reflect a tested, compatible set of tool versions.

## Agentic Task Guidance
✅ Safe to update default versions to newer tested combinations.
✅ Safe to add new tools (e.g., `kustomize`, `kubeseal`) following the existing pattern.
⚠️ After any `src/` change, rebuild `dist/index.js` and commit it.
⚠️ Node.js 12 is EOL — plan upgrade to `node20`.
⚠️ `kubectl-release-date` must match the actual release date for the specified `kubectl-version`.
❌ Do not hardcode download URLs — derive them from version inputs.
❌ Do not install tools globally (outside tool-cache) — use `core.addPath()` for PATH management.

## External Dependencies
- `@actions/core`, `@actions/tool-cache`
- kubectl: https://dl.k8s.io/release/{version}/bin/linux/amd64/kubectl
- Helm: https://get.helm.sh/helm-{version}-linux-amd64.tar.gz
- Helmfile: https://github.com/helmfile/helmfile/releases/download/{version}/helmfile_linux_amd64.tar.gz
- Upstream: https://github.com/mamezou-tech/setup-helmfile

## Common Pitfalls
- **`kubectl-release-date` mismatch**: The download URL includes the release date; an incorrect date returns a 404.
- **`master` plugin versions**: `helm-diff-plugin-version: master` may break if the upstream default branch is renamed or if a breaking change is merged.
- **Self-hosted runners**: Pre-installed Helm plugins may conflict; use `install-helm-plugins: no` and manage plugins separately.
- **macOS runners**: Binary paths and architectures differ; test explicitly on `macos-latest`.
- **Network timeouts**: Large binaries (helm, kubectl) may time out on slow networks; tool-cache retry logic helps but self-hosted runners may need a local mirror.
