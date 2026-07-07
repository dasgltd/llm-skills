---
name: git-workflow
description: Use this skill whenever you (agent or IDE assistant) save, backup, commit, push, branch, open a PR, or rewrite git history on any dasgltd repo. Canonical git convention shared across BOTH ends — Antigravity/Claude on the Mac and Craudinho/Hermes on the Vostro. Read it before any git write.
---

# DASG Git Workflow (canonical — both machines)

GitHub (`dasgltd`, private polyrepo) is the single source of truth. The same repos are cloned on the Mac (Antigravity IDE) and on the Vostro (Craudinho/Hermes headless). This skill is git-versioned in `llm-skills` and synced to both ends, so the rule is identical everywhere.

## The five rules

1. **Pull before, push after.** Start every work session with `git pull --ff-only`; push when you finish a unit of work. Never let a machine drift.
2. **PR-based gate for agent work (model A).** Autonomous/agent sessions (Craudinho, or any non-trivial change) do **not** push straight to `main`. Work on a branch, open a PR, and **Daniel merges** — the human review is the gate. Only trivial solo edits made deliberately by Daniel himself may go direct to `main`.
3. **No lock, but single-writer (model B).** There is no file locking. So **only one session mutates a given repo at a time.** Two sessions committing to the same repo in parallel corrupt each other — if you detect another `claude`/agent process operating the same repo, stop writing and reconcile before proceeding.
4. **Force-push only for authorized history cleanup, and back up first.** History rewrites (`git-filter-repo`, purge of secrets/scratch) are allowed with Daniel's authorization, but **always push a `backup/pre-*` ref to origin first** so the pre-rewrite history is recoverable, then force-push the rewritten branch. Never force-push without that safety ref.
5. **Never commit secrets.** No `.env`, key, token, or hardcoded credential in tracked files — ever. Secrets live in `sops+age` (`.env.sops` committed, plaintext `.env` gitignored). A committed secret is a compromised secret; if one lands, rotate + purge history.

## Execution pattern

Conventional commits, chained to minimize approvals:

```bash
git pull --ff-only
# ...work...
git checkout -b type/short-topic
git add <specific files> && git commit -m "type(scope): descriptive message" && git push -u origin type/short-topic
gh pr create --fill   # Daniel reviews & merges
```

For an authorized history purge:

```bash
git branch backup/pre-purge && git push origin backup/pre-purge   # safety net on origin
git filter-repo --replace-text redactions.txt --force             # or --invert-paths for whole files
git remote add origin <url>                                       # filter-repo drops the remote
git push origin main --force                                      # backup ref stays untouched
```

Prefer conventional prefixes (`feat`, `fix`, `refactor`, `chore`, `style`) summarizing exactly what was done.

## Antigravity note (Mac)
In the Antigravity UI, chain `add && commit && push` into one step so the user approves once, and infer the commit message without asking. But the destination is a **PR branch**, not `main`, for anything beyond a trivial solo edit — the gate in rule 2 still applies.
