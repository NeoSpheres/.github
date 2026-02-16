# .github

## Deployment

- GCP

## Build

- Autotag
- Docker

## Test

- Maven

## Hotfixing Java Services

### Branch Naming

| Branch                   | Purpose                                            | Example               |
|--------------------------|----------------------------------------------------|-----------------------|
| `hotfix/v{version}-base` | Base branch from the release tag. PR target.       | `hotfix/v1.33.0-base` |
| `hotfix/v{version}`      | Fix branch from the base. Contains hotfix commits. | `hotfix/v1.33.0`      |

### Process

#### 1. Create base branch from the release tag

```bash
git checkout -b hotfix/v1.33.0-base v1.33.0
git push -u origin hotfix/v1.33.0-base
```

#### 2. Create fix branch and implement the fix

```bash
git checkout -b hotfix/v1.33.0
# Make changes
git add <files>
git commit -m "fix: description of the hotfix"
git push -u origin hotfix/v1.33.0
```

#### 3. Create a PR targeting the base branch

```bash
gh pr create --title "fix: description" --base hotfix/v1.33.0-base
```

The PR must target `hotfix/v{version}-base`, **not** `master`.

#### 4. CI runs automatically

The PR triggers `01_tests.yml` which extracts the version from the branch name, runs tests and spotless, builds a Docker
image (tagged `1.33.0-fix1`), and creates git tag `v1.33.0-fix1`.

Each additional push increments the tag: `v1.33.0-fix2`, `v1.33.0-fix3`, etc.

#### 5. Deploy

Use the Docker image tagged with the fix version (e.g., `1.33.0-fix1`). Adding the `godev` label to the PR triggers
deployment to dev.

#### 6. Backport to master

After deployment, create a new PR from the base branch to `master`:

```bash
gh pr create --title "fix: backport hotfix v1.33.0" --head hotfix/v1.33.0-base --base master
```

### CI Notes

- `lint_pr` is skipped for hotfix branches. All downstream jobs must include `!cancelled()` in their `if` condition to
  avoid being skipped by GitHub Actions' skip propagation.
- Docker images use `{version}-fix{N}` tags instead of the usual `pr-{number}` format.
- Tag resolution queries the GitHub API for existing `v{version}-fix{N}` tags and increments `N` until an unused tag is
  found.
