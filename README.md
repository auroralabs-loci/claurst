# LOCI Mirror Template

Template for mirroring upstream repositories with LOCI analysis.

## Setup

1. Create repo from this template
2. Add topic `loci-mirror` to receive automatic overlay updates
3. Configure the required variables and secrets below

## Required Configuration

### Variables (`Settings > Variables > Actions`)

| Variable | Description |
|----------|-------------|
| `UPSTREAM_REPO` | Upstream repo to mirror (e.g., `openssl/openssl`) |
| `MIRROR_MAX_UPSTREAM_PRS` | Max PRs to mirror per sync (e.g., `3`) |
| `MIRROR_UPSTREAM_PR_LOOKBACK_DAYS` | Only mirror PRs updated within N days (e.g., `2`) |
| `SYNC_PRS_HOURS` | Hours (UTC) to run PR sync (e.g., `1,5,9,13,17,21`). Empty = every hour |
| `LOCI_BACKEND_URL` | LOCI backend API URL |
| `LOCI_ENV` | LOCI environment (optional, defaults to `PROD__AL_DEMO`) |

### Secrets (`Settings > Secrets and variables > Actions`)

| Secret | Description |
|--------|-------------|
| `MIRROR_REPOS_WRITE_PAT` | PAT with repo write access (push branches, create PRs) |
| `LOCI_API_KEY` | LOCI API key |

### Topics

To get fresh updates from mirror-template repository, make sure to add `loci-mirror` topic to your repo. 

## Restricting Upstream Actions

To prevent upstream workflows from running in your mirror, restrict allowed actions:

1. Go to **Settings → Actions → General**
2. Under "Actions permissions", select **Allow auroralabs-loci, and select non-auroralabs-loci, actions and reusable workflows**
3. In the text box, add the following (replace `REPO_NAME` with your upstream repo name):

```
!auroralabs-loci/REPO_NAME/.github/workflows/*,
actions/cache@v3,
actions/checkout@v4,
actions/setup-python@v5.6.0,
auroralabs-loci/REPO_NAME/.github/workflows/loci-analysis.yml@*,
auroralabs-loci/REPO_NAME/.github/workflows/sync-upstream-prs.yml@*,
auroralabs-loci/REPO_NAME/.github/workflows/process-pending-prs.yml@*,
```

This blocks all upstream workflows (`!.../*`) and then whitelists only the LOCI workflows and required actions.

## Customization

Edit `.github/workflows/loci-analysis.yml` and search for `@configure:` comments:

| Location | What to change |
|----------|----------------|
| `LOCI_PROJECT` | Your LOCI project name |
| `Install dependencies` step | Commands to install build tools |
| `Build` step | Commands to build your binaries |
| `binaries:` | Paths to built binaries for upload |

## How It Works

- **`sync-upstream-prs.yml`**: Syncs `main` to upstream, creates `loci/main-${sha}` branches with overlay, and mirrors upstream PRs as `loci/pr-${num}-${branch}` (or `loci/pending-pr-*` when the base branch is new)
- **`process-pending-prs.yml`**: Promotes `loci/pending-pr-*` branches to `loci/pr-*` and creates mirror PRs
- **`loci-analysis.yml`**: Runs LOCI analysis on pushes to `loci/main-*` and on PRs
- **`sync-overlay.yml`**: *(template repo only)* Syncs overlay changes to all repos with `loci-mirror` topic