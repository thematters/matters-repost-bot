# Matters Repost Bot

This bot mirrors selected source-site articles into Matters drafts.

It currently supports three scheduled repost streams:

| Workflow | Source | Destination account | State file |
| --- | --- | --- | --- |
| `repost-mattershklit.yml` | Formless / P-articles | `@mattershklit` | `state/mattershklit.json` |
| `repost-mattershkrec-witness.yml` | The Witness | `@mattershkrec` | `state/mattershkrec_witness.json` |
| `repost-mattershkrec-collective.yml` | The Collective HK | `@mattershkrec` | `state/mattershkrec_collective.json` |

The default mode is draft-only. Editors review formatting, images, tags, source credit, and licensing in Matters before publishing.

## What It Does

- Fetches recent article references from each configured source.
- Compares source-specific IDs with the matching state file.
- Fetches new articles, cleans source HTML, and uploads images to Matters.
- Creates or updates Matters drafts with title, body, images, tags, source link, credit links, and `arr` license.
- Commits the updated state file back to the repository after successful processing.

## GitHub Actions Setup

Create the required repository secrets under:

`Settings -> Secrets and variables -> Actions -> New repository secret`

| Secret | Used by | Description |
| --- | --- | --- |
| `MATTERS_EMAIL` | Formless / P-articles workflow | Matters login email for `@mattershklit`. |
| `MATTERS_PASSWORD` | Formless / P-articles workflow | Matters password for `@mattershklit`. |
| `MATTERSHKREC_EMAIL` | The Witness and The Collective HK workflows | Matters login email for `@mattershkrec`. |
| `MATTERSHKREC_PASSWORD` | The Witness and The Collective HK workflows | Matters password for `@mattershkrec`. |

Passwords are stored only in GitHub Actions secrets and are read at runtime.

## Schedules

| Workflow | Schedule |
| --- | --- |
| Formless / P-articles | Monday and Thursday at 06:00 Hong Kong time. |
| The Witness | Tuesday and Friday at 06:00 Hong Kong time. |
| The Collective HK | Tuesday and Friday at 06:00 Hong Kong time. |

The workflows use UTC cron values in `.github/workflows/*.yml`.

## First Run

Run each workflow once with `bootstrap=true` before normal operation.

Bootstrap records the currently visible article IDs as already seen and posts nothing. Without this step, the bot may treat older homepage articles as new.

Manual workflow inputs:

| Input | Meaning |
| --- | --- |
| `bootstrap=true` | Record the current source position and skip posting. |
| `dry_run=true` | Fetch and parse articles, but do not call the Matters API or advance state. |
| `publish=true` | Publish immediately instead of leaving drafts. Default is `false`. |

## Local Testing

Install dependencies:

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

Dry-run a source against its tracked state:

```bash
python -m bot.main --source p_articles --state state/mattershklit.json --dry-run
python -m bot.main --source thewitnesshk --state state/mattershkrec_witness.json --dry-run
python -m bot.main --source thecollectivehk --state state/mattershkrec_collective.json --dry-run
```

Force one article through the parser without touching Matters:

```bash
tmp_state="$(mktemp)"
printf '{}' > "$tmp_state"
python -m bot.main --source p_articles --state "$tmp_state" --dry-run --max 1
```

Available flags:

- `--source`: required source name. Current values are `p_articles`, `thewitnesshk`, and `thecollectivehk`.
- `--state`: state JSON path. Defaults to `state/<source>.json`.
- `--bootstrap`: record current refs as seen without posting.
- `--dry-run`: skip all Matters API calls and leave state unchanged.
- `--publish`: publish immediately. Without this flag, the bot leaves drafts for manual review.
- `--max`: cap the number of articles processed in one run.

## File Structure

```text
.
|-- .github/workflows/          # Scheduled and manually triggered repost jobs
|-- bot/
|   |-- config.py               # Shared environment variables and constants
|   |-- main.py                 # Repost orchestrator
|   |-- matters_client.py       # Minimal Matters GraphQL client
|   `-- sources/                # Source-specific list, parse, state, and credit logic
|-- state/                      # Persisted per-stream source state
|-- .env.example                # Local credential template
`-- requirements.txt
```

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| No new articles are found | Confirm the source homepage/API still returns article refs and compare with the matching state file. |
| Login fails | Confirm the relevant GitHub Actions secrets are set for the destination Matters account. |
| Images are missing | Source image servers or Matters uploads may fail intermittently. Check workflow logs for upload warnings. |
| Draft formatting is off | Review the source-specific cleaner in `bot/sources/<source>.py`. |
| Articles were skipped | Lower the relevant state value and rerun a dry run first. |
| A run has too many new articles | Adjust `MAX_ARTICLES_PER_RUN` or `--max`; unprocessed articles are picked up later because state advances only after success. |

## Notes

- Keep source authorization and editorial approval records outside this repository if they contain private information.
- State files are committed to the repository so GitHub Actions can continue from the last successful run.
- Production publishing should remain manual unless the destination account owner explicitly approves auto-publish behavior.
- If the Matters GraphQL schema changes, update `bot/matters_client.py` and verify with a dry run before enabling scheduled posting again.
