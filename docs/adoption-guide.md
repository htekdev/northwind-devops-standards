# Adoption Guide

> Northwind Traders — Platform Engineering

This guide walks you through adopting the Northwind DevOps Standards in your repository. Follow the steps for your project's stack to get up and running with centralized CI/CD in minutes.

---

## Prerequisites

Before adopting the standard workflows, ensure your repository meets the following requirements:

1. **Repository is in the `northwind-traders` GitHub organization** — the reusable workflows are scoped to the organization.
2. **Branch protection is configured on `main`** — see [CI/CD Standards](ci-cd-standards.md#3-branch-protection-requirements).
3. **Test suite exists and runs locally** — the CI workflows execute your existing test commands; they do not create tests for you.
4. **Coverage reporting is configured** in your test runner:
   - **.NET:** Ensure the `coverlet.collector` NuGet package is referenced in test projects.
   - **Node.js:** Ensure your test script supports `--coverage` (Jest, Vitest, and c8 all do).

---

## Step 1: Add a Caller Workflow

Create a new file in your repository at `.github/workflows/ci.yml` with the content for your stack.

### .NET Repositories

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
      build-configuration: 'Release'
      coverage-threshold: 80

  security:
    uses: northwind-traders/northwind-devops-standards/.github/workflows/security-scan.yml@main
    with:
      language: 'csharp'
```

#### Configuration Options

| Input | Default | Description |
|-------|---------|-------------|
| `dotnet-version` | `8.0.x` | .NET SDK version to install |
| `solution-path` | `.` | Path to your `.sln` or `.csproj` file |
| `build-configuration` | `Release` | MSBuild configuration |
| `test-projects` | `**/*Tests/*.csproj` | Glob pattern for test project discovery |
| `coverage-threshold` | `80` | Minimum line coverage percentage (set to `0` to disable) |
| `runs-on` | `ubuntu-latest` | GitHub Actions runner label |

### Node.js Repositories

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
      lint-script: 'lint'
      build-script: 'build'
      test-script: 'test'
      coverage-threshold: 80

  security:
    uses: northwind-traders/northwind-devops-standards/.github/workflows/security-scan.yml@main
    with:
      language: 'javascript'
```

#### Configuration Options

| Input | Default | Description |
|-------|---------|-------------|
| `node-version` | `20` | Node.js version to install |
| `package-manager` | `npm` | Package manager (`npm`, `pnpm`, or `yarn`) |
| `lint-script` | `lint` | npm script name for linting (empty string to skip) |
| `build-script` | `build` | npm script name for building (empty string to skip) |
| `test-script` | `test` | npm script name for running tests |
| `coverage-threshold` | `80` | Minimum line coverage percentage (set to `0` to disable) |
| `runs-on` | `ubuntu-latest` | GitHub Actions runner label |

### Docker Repositories

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
      dockerfile: './Dockerfile'
      context: '.'
      platforms: 'linux/amd64,linux/arm64'
      push: ${{ github.event_name == 'push' }}
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

#### Configuration Options

| Input | Default | Description |
|-------|---------|-------------|
| `image-name` | *(required)* | Docker image name without registry prefix |
| `dockerfile` | `./Dockerfile` | Path to Dockerfile |
| `context` | `.` | Docker build context directory |
| `registry` | `ghcr.io` | Container registry URL |
| `push` | `true` | Whether to push the built image |
| `platforms` | `linux/amd64` | Comma-separated target platforms |
| `scan-severity` | `CRITICAL,HIGH` | Trivy severity threshold |

---

## Step 2: Using Composite Actions Directly

If you need to use the setup actions in custom workflows, reference them directly:

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup .NET
    uses: northwind-traders/northwind-devops-standards/.github/actions/setup-dotnet-env@main
    with:
      dotnet-version: '8.0.x'
      solution-path: './src/MyApp.sln'

  - name: Run custom build steps
    run: dotnet build --configuration Release
```

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup Node.js
    uses: northwind-traders/northwind-devops-standards/.github/actions/setup-node-env@main
    with:
      node-version: '20'
      package-manager: 'pnpm'

  - name: Run custom scripts
    run: pnpm run my-custom-task
```

---

## Step 3: Configure Required Status Checks

After your first successful CI run, configure branch protection on `main`:

1. Go to **Settings → Branches → Branch protection rules**.
2. Click **Add rule** for the `main` branch.
3. Enable **Require status checks to pass before merging**.
4. Search for and add the CI job names (e.g., `Build & Test (.NET 8.0.x)`, `Lint`, `CodeQL Analysis (csharp)`).
5. Enable **Require branches to be up to date before merging**.
6. Save changes.

---

## Troubleshooting

### "Reusable workflow not found" Error

**Cause:** The caller workflow cannot resolve the reusable workflow reference.

**Fix:** Ensure the reference uses the correct format:
```
northwind-traders/northwind-devops-standards/.github/workflows/<workflow>.yml@main
```
Verify that:
- The repository name is spelled correctly.
- The workflow file exists at the specified path.
- Your repository has access to the organization's internal repositories (if applicable).

### Coverage Threshold Failures

**Cause:** Your test suite's line coverage is below the configured `coverage-threshold`.

**Fix:**
1. Run tests locally with coverage to identify uncovered code:
   - .NET: `dotnet test --collect:"XPlat Code Coverage"`
   - Node.js: `npm test -- --coverage`
2. Add tests for uncovered branches and functions.
3. If the threshold is too aggressive for your project, lower `coverage-threshold` temporarily and create a plan to increase coverage incrementally.

### NuGet Restore Failures (.NET)

**Cause:** Private NuGet feeds require authentication.

**Fix:** Add a `nuget.config` file to your repository root with the private feed configured, and pass credentials via repository secrets. The `setup-dotnet-env` action runs `dotnet restore` after caching, so ensure your feed is accessible from GitHub-hosted runners.

### Package Manager Mismatch (Node.js)

**Cause:** The `package-manager` input doesn't match your repository's lock file.

**Fix:** Ensure consistency:
| Package Manager | Required Lock File |
|----------------|-------------------|
| `npm` | `package-lock.json` |
| `pnpm` | `pnpm-lock.yaml` |
| `yarn` | `yarn.lock` |

### Docker Build Failures

**Cause:** Multi-platform builds require Docker Buildx, which is configured automatically by the workflow.

**Fix:** Ensure your `Dockerfile` is compatible with the target platforms. Common issues:
- Using platform-specific base images (e.g., Windows containers when targeting `linux/amd64`).
- Architecture-specific binary downloads without conditional logic.

---

## Migration from Existing CI/CD

If your repository already has CI/CD workflows:

1. **Keep existing workflows temporarily** — rename them (e.g., `ci-legacy.yml`) while testing the new standard workflows.
2. **Add the standard caller workflow** as described in Step 1.
3. **Verify both pipelines pass** on a test PR.
4. **Remove the legacy workflow** once the standard pipeline is confirmed working.
5. **Update branch protection rules** to reference the new job names.

This approach ensures zero downtime during migration and provides a rollback path.
