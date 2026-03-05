# 🏪 Northwind Traders — DevOps Standards

[![Reusable Workflows](https://img.shields.io/badge/workflows-4%20reusable-blue?logo=github-actions)](/.github/workflows)
[![Composite Actions](https://img.shields.io/badge/actions-3%20composite-green?logo=github-actions)](/.github/actions)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](/LICENSE)

Central **Center of Excellence (CoE)** repository providing reusable GitHub Actions workflows and composite actions for standardized CI/CD across all Northwind Traders repositories.

---

## 📦 Reusable Workflows

| Workflow | File | Description |
|----------|------|-------------|
| **.NET CI Pipeline** | [`ci-dotnet.yml`](.github/workflows/ci-dotnet.yml) | Build, test, and enforce coverage thresholds for .NET projects |
| **Node.js CI Pipeline** | [`ci-node.yml`](.github/workflows/ci-node.yml) | Lint, build, test, and enforce coverage thresholds for Node.js projects |
| **Docker Build & Publish** | [`docker-publish.yml`](.github/workflows/docker-publish.yml) | Build, scan with Trivy, and publish container images to GHCR |
| **Security Scan** | [`security-scan.yml`](.github/workflows/security-scan.yml) | CodeQL analysis and dependency review for PRs |

## 🔧 Composite Actions

| Action | Path | Description |
|--------|------|-------------|
| **Setup .NET Environment** | [`setup-dotnet-env`](.github/actions/setup-dotnet-env/action.yml) | Installs .NET SDK, caches NuGet packages, and restores dependencies |
| **Setup Node.js Environment** | [`setup-node-env`](.github/actions/setup-node-env/action.yml) | Installs Node.js, configures package manager caching, and installs dependencies |
| **Publish Test Results** | [`publish-test-results`](.github/actions/publish-test-results/action.yml) | Uploads test results and coverage reports as workflow artifacts |

---

## 🚀 Quick Start

Add a caller workflow to your repository's `.github/workflows/` directory:

### .NET Project

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: northwind-traders/northwind-devops-standards/.github/workflows/ci-dotnet.yml@main
    with:
      dotnet-version: '8.0.x'
      solution-path: './src/MyApp.sln'
      coverage-threshold: 80
```

### Node.js Project

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: northwind-traders/northwind-devops-standards/.github/workflows/ci-node.yml@main
    with:
      node-version: '20'
      package-manager: 'npm'
      coverage-threshold: 80
```

### Docker Project

```yaml
name: Docker

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  docker:
    uses: northwind-traders/northwind-devops-standards/.github/workflows/docker-publish.yml@main
    with:
      image-name: 'my-service'
      platforms: 'linux/amd64,linux/arm64'
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

See the [`examples/`](examples/) directory for complete caller workflow files you can copy directly.

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [CI/CD Standards](docs/ci-cd-standards.md) | Pipeline requirements, supported stacks, and policies |
| [Testing Strategy](docs/testing-strategy.md) | Coverage requirements, test tiers, and naming conventions |
| [Adoption Guide](docs/adoption-guide.md) | Step-by-step guide to onboard your repository |

---

## 🤝 Contributing

1. **Fork** this repository.
2. **Create a feature branch** from `main` (`git checkout -b feature/my-improvement`).
3. **Make your changes** — all workflow and action changes require review from `@northwind-traders/platform-engineering` (see [CODEOWNERS](.github/CODEOWNERS)).
4. **Test your changes** by referencing your fork/branch in a caller workflow.
5. **Open a Pull Request** with a clear description of what changed and why.

### Guidelines

- All reusable workflows must use `workflow_call` trigger exclusively.
- Composite actions must specify `using: 'composite'` and set `shell` on every `run` step.
- Pin third-party actions to specific versions (never use `@latest`).
- Include meaningful `description` fields on all inputs and secrets.
- Update documentation in `docs/` when adding or changing workflows.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

Copyright © 2025 Northwind Traders
