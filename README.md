# python-project-blueprint

Opinionated blueprint for Python packages with automated CI, releases, and publishing.

## What's included

| File | Purpose |
|---|---|
| `pyproject.toml` | hatchling + hatch-vcs, version from git tags |
| `.github/workflows/ci.yaml` | Lint (ruff), type check (mypy), test (pytest), docs build |
| `.github/workflows/release.yaml` | release-please: version bumps + changelog |
| `.github/workflows/publish.yaml` | PyPI publish on tag push or release |
| `.github/workflows/pr-title.yaml` | Conventional commit PR title validation |
| `.github/workflows/dependabot-auto-merge.yaml` | Auto-merge minor/patch updates |
| `.readthedocs.yaml` | Read the Docs with uv |

## Release flow

```
merge to main  →  release-please opens version bump PR
merge that PR  →  git tag + GitHub Release → PyPI publish → RTD rebuild
```

Manual pre-releases:

```bash
git tag v1.0.0-rc.1
git push origin v1.0.0-rc.1
# → PyPI publish + GitHub Release (marked as pre-release)
```

## Setup

### Find and replace

| Placeholder | Replace with | Example |
|---|---|---|
| `my-package` | PyPI package name | `fluxopt` |
| `my_package` | Python import name | `fluxopt` |

### GitHub App for release-please

Needed so release-please PRs trigger CI (the default `GITHUB_TOKEN` can't).

1. [Create a GitHub App](https://github.com/settings/apps/new) with **Contents** and **Pull requests** read & write
2. Install it on your repo
3. Add secrets `APP_ID` and `APP_PRIVATE_KEY`

### PyPI trusted publisher

1. Create a `pypi` environment in repo settings (Settings → Environments)
2. Add a [trusted publisher](https://pypi.org/manage/account/publishing/) — workflow: `publish.yaml`, environment: `pypi`

### Read the Docs

Connect your repo at [readthedocs.org](https://readthedocs.org/). Rebuilds automatically on push.

### Codecov (optional)

Add `CODECOV_TOKEN` as a repository secret.

### Branch protection

Add `CI Success` as a required status check on `main`.
