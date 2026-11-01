# ci-workflows

Reusable GitHub Actions workflows for Zen in Tech projects. Fix CI once here and every project picks it up through Dependabot.

| Workflow | Runs on | What it checks |
|---|---|---|
| [`nextjs-ci.yml`](.github/workflows/nextjs-ci.yml) | pull requests and pushes to `main` | `npm ci`, lint, typecheck, formatting, tests, build. Node version from `.nvmrc`. |
| [`pr-title.yml`](.github/workflows/pr-title.yml) | pull requests | The title follows [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). It becomes the squash commit and the release note. |
| [`main-guard.yml`](.github/workflows/main-guard.yml) | pushes to `main` | Every new commit on `main` came from a merged pull request, and `main` was not force-pushed. Private repositories on GitHub Free cannot protect branches, so this check makes a direct push visible. |

## Usage

Projects created from [`tpl-nextjs`](https://github.com/Zen-In-Tech/tpl-nextjs) already call these workflows. To add them to another repository:

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
  push:
    branches: [main]
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
permissions:
  contents: read
jobs:
  nextjs:
    uses: Zen-In-Tech/ci-workflows/.github/workflows/nextjs-ci.yml@<commit-sha> # v1.0.0
```

```yaml
# .github/workflows/pr-title.yml
name: PR title
on:
  pull_request:
    types: [opened, edited, reopened, synchronize]
permissions: {}
jobs:
  title:
    uses: Zen-In-Tech/ci-workflows/.github/workflows/pr-title.yml@<commit-sha> # v1.0.0
```

```yaml
# .github/workflows/main-guard.yml
name: Main guard
on:
  push:
    branches: [main]
permissions:
  contents: read
  pull-requests: read
jobs:
  guard:
    uses: Zen-In-Tech/ci-workflows/.github/workflows/main-guard.yml@<commit-sha> # v1.0.0
```

`nextjs-ci.yml` inputs:

| Input | Default | Meaning |
|---|---|---|
| `working-directory` | `.` | Directory with `package.json`, `package-lock.json` and `.nvmrc` |
| `run-tests` | `true` | Run `npm test` |

The project must define the npm scripts `lint`, `typecheck`, `format:check`, `test` and `build`.

## Versions and pinning

- The organization requires every action and reusable workflow to be pinned to a full commit SHA. Use the SHA of a release tag and keep the tag in a comment: `@<sha> # v1.2.0`.
- Releases follow SemVer. A breaking change, such as a new required npm script, is a major version.
- Dependabot updates the SHAs in projects and in this repository every week.

## Security

- Jobs get the minimum `permissions` they need; the checkout does not keep credentials.
- Third-party actions are pinned to a SHA and limited to GitHub and verified creators by the organization policy.
- No secrets are used. A workflow that needs one must receive it from the caller as an explicit `secrets:` input.
