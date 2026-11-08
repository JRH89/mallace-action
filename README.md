# Mallace find-and-fix

Mallace finds security bugs in your Python and JavaScript/TypeScript code and proves each one with a regression test. It then fixes the bug and opens a pull request containing the fix and the test. It runs on your own GitHub Actions runners, and nothing is merged automatically.

## Quick start

1. Add an `OPENAI_API_KEY` repository secret. Without it, Mallace only scans and reports.
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
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
          budget: '2'          # most the AI may spend per run, in dollars
```

Or sign in at [mallace.codefrontlabs.com/app](https://mallace.codefrontlabs.com/app) and Mallace opens this setup pull request for you.

## What a run does

1. **Scan.** A pinned, offline Semgrep ruleset looks for caller-controlled data reaching command execution, `eval`, SQL construction, unsafe YAML or pickle loads.
2. **Prove.** For each finding, a model writes a regression test. The test runs in a sandbox with no network. Only findings whose test fails on your current code count as confirmed.
3. **Fix.** A second model writes the fix. The test must pass afterwards, and an independent reviewer model must approve the change.
4. **Deliver.** One issue and one pull request per verified fix. Findings are annotated on their lines and a report is written to the job summary.

Spend is a hard cap across every model call in the run.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `openai-api-key` | | OpenAI-compatible key; not needed for `mode: scan` |
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
mallace doctor            # checks Docker, your key, GitHub login; prints fixes
mallace fix --scan-only   # free
mallace fix --budget 5
```

Only run Mallace on repositories you own or are authorized to test. Tests and fixes are model-written, so review every pull request before merging.
