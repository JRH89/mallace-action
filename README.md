# Mallace find-and-fix

Mallace finds security bugs in your Python and JavaScript/TypeScript code and proves each one with a regression test. It then fixes the bug and opens a pull request containing the fix and the test. It runs on your own GitHub Actions runners, and nothing is merged automatically.

## Quick start

1. Sign in at [mallace.codefrontlabs.com/app](https://mallace.codefrontlabs.com/app), create your free license key, and add it as the `MALLACE_LICENSE` repository secret. Without it, Mallace only scans and reports.
2. In **Settings → Actions → General**, enable **Allow GitHub Actions to create and approve pull requests**.
3. Add `.github/workflows/mallace.yml`:

```yaml
name: Mallace
on:
  schedule:
    - cron: '0 6 * * 1'   # weekly
  workflow_dispatch:
permissions:
  contents: write
  issues: write
  pull-requests: write
jobs:
  mallace:
    runs-on: ubuntu-latest
    timeout-minutes: 60
    steps:
      - uses: actions/checkout@v4
      - uses: JRH89/mallace-action@v0
        with:
          license-key: ${{ secrets.MALLACE_LICENSE }}
          budget: '2'          # most the AI may spend per run, in dollars
```

Or sign in at [mallace.codefrontlabs.com/app](https://mallace.codefrontlabs.com/app) and Mallace opens this setup pull request for you.

## What a run does

1. **Scan.** A pinned, offline Semgrep ruleset looks for caller-controlled data reaching command execution, `eval`, SQL construction, unsafe YAML or pickle loads.
2. **Prove.** For each finding, a model writes a regression test. The test runs in a sandbox with no network. Only findings whose test fails on your current code count as confirmed.
3. **Fix.** A second model writes the fix. The test must pass afterwards, and an independent reviewer model must approve the change.
4. **Deliver.** One issue and one pull request per verified fix. Findings are annotated on their lines and a report is written to the job summary.

Spend is a hard cap across every model call in the run, and also counts toward your plan's monthly allowance. Only the code around each finding is sent to the Mallace service to write its test and fix, and it is never stored.

## Plans

Scanning is always free. **Free** includes fixes on 1 repository up to a monthly AI allowance. **Pro** ($29/month or $290/year) covers unlimited repositories with a larger allowance. Manage your plan at [mallace.codefrontlabs.com/app](https://mallace.codefrontlabs.com/app).

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `license-key` | | Your Mallace license key (`MALLACE_LICENSE` secret); not needed for `mode: scan` |
| `mode` | `fix` | `fix` proves, fixes and opens PRs; `scan` only reports |
| `budget` | `2` | Most the AI may spend this run, in dollars |
| `max-findings` | `3` | Findings to prove and fix per run (1-5) |
| `paths` | auto | Space-separated paths to limit the scan |
| `base` | default branch | Branch to fix and open PRs against |
| `github-token` | `GITHUB_TOKEN` | Use a PAT if you want CI to run on Mallace PRs |
| `version` | pinned | Mallace version from PyPI |

Outputs: `findings`, `fixed`, `pull-requests`.

## Run it locally instead

```bash
pip install --pre mallace
mallace activate          # paste your license key
mallace doctor            # checks Docker, your license, GitHub login; prints fixes
mallace fix --scan-only   # free
mallace fix --budget 5
```

Only run Mallace on repositories you own or are authorized to test. Tests and fixes are model-written, so review every pull request before merging.
