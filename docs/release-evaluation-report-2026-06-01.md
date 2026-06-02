# Release Evaluation Report

## Summary

| Field | Value |
| --- | --- |
| Feature | Fork import, English copy rewrite, and repost-bot validation |
| Date | 2026-06-01 |
| Evaluator | Release Evaluation Agent |
| Recommendation | Ready for repository review |
| Production approval | Not requested |

## Scope

| Repo / PR | Commit / Branch | Status |
| --- | --- | --- |
| `thematters/matters-repost-bot` | `english-docs-and-source-labels` | Local changes validated; PR pending. |

## Environment Matrix

| Environment | Web URL | GraphQL URL | Test profile | Mutation allowed | Result |
| --- | --- | --- | --- | --- | --- |
| Local | N/A | N/A | Python compile and source dry-run | No | Passed. |
| GitHub Actions | N/A | `https://server.matters.news/graphql` | Scheduled repost workflows | Draft creation only by default | Not run in this evaluation. |
| Production | `https://matters.town` | `https://server.matters.town/graphql` | Production posting | No by default | Not requested. |

## Commands

| Command | Working Directory | Result | Notes |
| --- | --- | --- | --- |
| `rg -n --hidden -g '!.git/**' -g '!.venv/**' -g '!**/__pycache__/**' "[\\p{Han}]|[^\\x00-\\x7F]" .` | repo root | Passed | No fixed Chinese or non-ASCII repository text remains. |
| `.venv/bin/python -m compileall bot` | repo root | Passed | Python syntax/import compile check passed. |
| `.venv/bin/python -m bot.main --source p_articles --state state/mattershklit.json --dry-run` | repo root | Passed | Found 40 refs, parsed 2 new articles, did not advance state. |
| `.venv/bin/python -m bot.main --source thecollectivehk --state state/mattershkrec_collective.json --dry-run` | repo root | Passed | Found 20 refs, parsed 1 new article, did not advance state. |
| `.venv/bin/python -m bot.main --source thewitnesshk --state state/mattershkrec_witness.json --dry-run` | repo root | Passed | Found 20 refs, no new articles under tracked state. |
| `.venv/bin/python -m bot.main --source thewitnesshk --state <temp> --dry-run --max 1` | repo root | Passed | Forced parser coverage for one Witness article with temporary state. |
| `git diff --check` | repo root | Passed | No whitespace errors. |

## Automated Test Results

| Suite | Target | Result | Evidence |
| --- | --- | --- | --- |
| Fixed-copy scan | Full repository, excluding `.git`, `.venv`, and `__pycache__` | Passed | No Chinese or non-ASCII fixed repository text found. |
| Compile | `bot/` | Passed | All bot modules compiled. |
| Source dry-run | `p_articles` | Passed | Parsed 2 new articles and left state unchanged. |
| Source dry-run | `thecollectivehk` | Passed | Parsed 1 new article and left state unchanged. |
| Source dry-run | `thewitnesshk` | Passed | Tracked state no-op plus forced parser run passed. |

## Browser Evidence

| Check | URL | Expected | Result | Evidence |
| --- | --- | --- | --- | --- |
| Browser UI | N/A | No browser surface for this bot repository. | Not applicable. | CLI-only bot. |

## API Evidence

| Check | Endpoint | Expected | Result | Evidence |
| --- | --- | --- | --- | --- |
| Source listing | Source home/API endpoints | Recent refs can be listed. | Passed. | 40 p-articles refs, 20 The Witness refs, 20 The Collective HK refs. |
| Matters API mutation | `https://server.matters.news/graphql` | No production mutation without approval. | Not run. | Dry-run mode only. |

## Feature Acceptance

| Gate | Expected | Result | Evidence |
| --- | --- | --- | --- |
| English fixed copy | README, workflow labels/comments, source credit labels, and generated fixed credit text are English. | Passed. | Repository scan returned no fixed Chinese/non-ASCII text. |
| Dry-run safety | Dry-run must not create drafts or advance state. | Passed after fix. | Dry-run logs include `state not advanced`; tracked state files stayed unchanged. |
| Source parser coverage | Each configured source should list refs and parse at least one article or no-op safely. | Passed. | p-articles, The Witness, and The Collective HK dry-runs passed. |
| State consistency | Scheduled state files should not be changed by validation. | Passed. | `git diff` shows no state-file changes. |

## Blockers

| Severity | Blocker | Owner | Required Action |
| --- | --- | --- | --- |
| Low | Local macOS default Python is 3.9 with LibreSSL, producing an urllib3 warning. | Reviewer / CI | Use GitHub Actions Python 3.11 for final CI confirmation. |

## Human Approvals Needed

| Approval | Needed For | Status | Owner |
| --- | --- | --- | --- |
| Production deploy | Scheduled bot operation in the org repository. | Pending. | Matters maintainers. |
| Production mutation | Creating or publishing real Matters drafts/articles. | Not requested. | Destination account owner. |
| Credentials / test account | Repository Actions secrets. | Pending. | Matters maintainers. |
| GitHub permission | Fork and PR creation. | Available. | `mashbean`. |

## Final Recommendation

`Ready for repository review`.

The repository is suitable for PR review. Scheduled production operation still requires repository secrets and explicit maintainer approval before enabling real draft creation or auto-publish behavior.
