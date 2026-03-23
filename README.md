# python-project-blueprint

Opinionated blueprint for Python packages with automated CI, releases, and publishing.

## What's included

| File | Purpose |
|---|---|
| `.github/workflows/ci.yaml` | Lint (ruff) + type check (mypy) + test (pytest matrix) + ci-success gate |
| `.github/workflows/release.yaml` | release-please with GitHub App token |
| `.github/workflows/publish.yaml` | Build + PyPI OIDC publish (tag push or workflow_call) |
| `.readthedocs.yaml` | Read the Docs configuration |
| `.github/workflows/pr-title.yaml` | Conventional commit PR title validation |
| `.github/workflows/dependabot-auto-merge.yaml` | Auto-merge minor/patch dependency bumps |
| `.github/dependabot.yml` | Dependabot config for actions + pip |
| `.release-please-config.json` | release-please configuration |
| `.release-please-manifest.json` | release-please version tracking |
| `pyproject.toml` | hatchling + hatch-vcs build system |
| `.pre-commit-config.yaml` | ruff pre-commit hooks |

## Pipeline overview

```
PR opened           → CI (lint + typecheck + test)
                    → PR title validated (conventional commit)

PR merged to main   → CI runs on main
                    → release-please opens/updates version bump PR

release PR merged   → release-please creates git tag + GitHub Release
                    → tag triggers publish.yaml → PyPI
                    → Read the Docs rebuilds automatically (webhook)

manual tag push     → publish.yaml → PyPI + GitHub Release
  v1.0.0-rc.1         (pre-release tags detected automatically)
```

## Setup checklist

### 1. Find and replace

Search and replace these placeholders:

| Placeholder | Replace with | Example |
|---|---|---|
| `my-package` | PyPI package name | `fluxopt` |
| `my_package` | Python import name | `fluxopt` |

### 2. GitHub App for release-please

A GitHub App token is used so that release-please PRs trigger CI workflows
(the default `GITHUB_TOKEN` cannot do this).

1. Create a GitHub App (org or personal) with permissions:
   - **Contents**: Read & write
   - **Pull requests**: Read & write
   - **Metadata**: Read-only
2. Install the app on your repository
3. Add repository secrets:
   - `APP_ID`: the app's ID
   - `APP_PRIVATE_KEY`: the app's private key

If you already have an app from another project, reuse it.

### 3. PyPI trusted publisher

1. Go to https://pypi.org/manage/project/YOUR-PACKAGE/settings/publishing/
2. Add a trusted publisher:
   - **Owner**: your GitHub org/user
   - **Repository**: your repo name
   - **Workflow**: `publish.yaml`
   - **Environment**: `pypi`
3. Create a GitHub environment named `pypi` in your repo settings

### 4. Read the Docs

1. Connect your repo at [readthedocs.org](https://readthedocs.org/)
2. RTD rebuilds automatically on push and tag creation (versioned docs)

Docs dependencies live in `[project.optional-dependencies] docs` so both
RTD (`pip install .[docs]`) and local dev (`uv sync --group dev` via
`include-group`) resolve from the same source. CI also validates the docs
build via `mkdocs build --strict`.

### 5. Codecov (optional)

Add `CODECOV_TOKEN` as a repository secret.

### 6. Branch protection

Add `CI Success` as a required status check on your main branch.

## Release flows

### Regular release

1. Merge PRs with conventional commit titles (`feat:`, `fix:`, etc.)
2. release-please automatically opens a version bump PR with CHANGELOG update
3. Merge the release-please PR → git tag created → PyPI publish → docs deploy

### Pre-release

```bash
git tag v1.0.0-rc.1
git push origin v1.0.0-rc.1
```

Creates a GitHub Release marked as pre-release and publishes to PyPI.

### Hotfix

```bash
git tag v1.0.1
git push origin v1.0.1
```

Publishes to PyPI and creates a GitHub Release. Note: skips CHANGELOG (managed by release-please).

## Customization

### Skip docs

Delete `.readthedocs.yaml`, `mkdocs.yml`, and the `docs/` directory.
Remove the `docs` job from `ci.yaml` and `docs` from the `ci-success` needs list.

### Skip typecheck

Remove the `typecheck` job from `ci.yaml` and `mypy` from dev dependencies.

### Add OS matrix

```yaml
# in ci.yaml test job
matrix:
  os: [ubuntu-24.04, macos-latest, windows-latest]
  python-version: ["3.12", "3.13"]
```

### Separate CI per branch

If your project has a `develop` → `main` branching model (like tsam), split `ci.yaml`
into `ci-develop.yaml` and `ci-main.yaml` with separate triggers and test matrices.
