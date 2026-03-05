# CI/CD Standards

> Northwind Traders — Platform Engineering

This document defines the mandatory CI/CD standards that all Northwind Traders repositories must follow. These standards ensure consistency, security, and reliability across the organization.

---

## 1. Pipeline Requirements

All repositories **must** use the centralized reusable workflows from [`northwind-devops-standards`](https://github.com/northwind-traders/northwind-devops-standards). Direct inline workflow definitions are not permitted for build, test, or deployment pipelines.

### Mandatory Pipelines

| Repository Type | Required Workflow(s) |
|----------------|----------------------|
| .NET application | `ci-dotnet.yml` + `security-scan.yml` |
| Node.js application | `ci-node.yml` + `security-scan.yml` |
| Containerized service | Stack-specific CI + `docker-publish.yml` + `security-scan.yml` |
| Library / shared package | Stack-specific CI + `security-scan.yml` |

### Pipeline Triggers

Every repository must configure CI to run on:

- **Push** to `main` (or default branch)
- **Pull requests** targeting `main`
- **Tag pushes** matching `v*` (for Docker publish workflows)

---

## 2. Supported Stacks

| Stack | Minimum Version | Reusable Workflow |
|-------|-----------------|-------------------|
| .NET | 8.0 | `ci-dotnet.yml` |
| Node.js | 20 LTS | `ci-node.yml` |
| Docker | BuildKit enabled | `docker-publish.yml` |

If your project uses a stack not listed above, contact the Platform Engineering team to discuss adding a new reusable workflow.

---

## 3. Branch Protection Requirements

The `main` branch of every repository must enforce:

- **Require pull request reviews** — minimum 1 approving review.
- **Require status checks to pass** — the CI workflow must be a required status check.
- **Require branches to be up to date** — merge commits must include the latest `main`.
- **Require signed commits** — recommended but not mandatory.
- **Do not allow bypassing** — no exceptions for administrators.

---

## 4. Artifact Retention Policies

| Artifact Type | Retention Period |
|---------------|-----------------|
| Test results | 14 days |
| Coverage reports | 14 days |
| Container images (dev/PR) | 30 days |
| Container images (release tags) | Indefinite |
| SARIF security reports | Governed by GitHub Advanced Security |

---

## 5. Security Scanning Mandates

### CodeQL Analysis

Every repository must include the `security-scan.yml` reusable workflow configured with the appropriate language:

```yaml
jobs:
  security:
    uses: northwind-traders/northwind-devops-standards/.github/workflows/security-scan.yml@main
    with:
      language: 'csharp'  # or 'javascript', 'typescript', 'python', etc.
```

CodeQL runs the `security-and-quality` query suite, which covers:

- SQL injection
- Cross-site scripting (XSS)
- Path traversal
- Insecure deserialization
- Hard-coded credentials
- And many more language-specific checks

### Dependency Review

On every pull request, the `dependency-review` job automatically:

- Scans for newly introduced dependencies with known vulnerabilities.
- Blocks dependencies with **high** or **critical** severity CVEs.
- Denies packages using **GPL-3.0** or **AGPL-3.0** licenses.

---

## 6. Coverage Thresholds

All projects must meet the following minimum coverage thresholds:

| Metric | Minimum |
|--------|---------|
| Line coverage | 80% |
| Branch coverage | 75% |
| Function coverage | 80% |

Coverage is enforced automatically by the CI workflows. Builds that fall below the configured `coverage-threshold` input will fail.

### Requesting an Exception

If a repository cannot meet coverage thresholds due to legitimate constraints (e.g., generated code, infrastructure-only projects), file an exception request with the Platform Engineering team. Exceptions must be documented in the repository's README.

---

## 7. Versioning and Pinning

- All third-party GitHub Actions must be pinned to a **specific version tag** (e.g., `actions/checkout@v4`). Using `@latest` or `@main` for third-party actions is prohibited.
- References to `northwind-devops-standards` workflows should use `@main` for the latest stable version.
- Breaking changes to reusable workflows will be communicated via release notes and a migration window of at least 2 weeks.

---

## 8. Runner Requirements

| Workflow | Default Runner | Self-Hosted Option |
|----------|---------------|-------------------|
| CI (.NET / Node.js) | `ubuntu-latest` | Configurable via `runs-on` input |
| Docker Build & Publish | `ubuntu-latest` | Not configurable (BuildKit dependency) |
| Security Scan | `ubuntu-latest` | Configurable via `runs-on` input |

Self-hosted runners must meet the minimum specifications published by the Platform Engineering team and must be ephemeral (fresh environment per job).
