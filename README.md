# harden-skill

An agent skill that reads your application's runtime logs, finds new errors, writes tests to reproduce them, fixes the code, and records what it did.

## What it does

Your app runs. It produces logs. Some lines say `ERROR:`. The harden skill reads those logs, figures out which errors are new (not already addressed), and for each one: writes a failing test, fixes the code, and commits everything. A hardening ledger tracks what's been done so the same error is never re-analyzed — even months or years later.

## Install

Copy `skills/harden/` into your agent's skill directory, or point your agent at `skills/harden/SKILL.md`. The skill follows the [Agent Skills standard](https://agentskills.io) and works with any compatible coding agent.

## Use

```
/skill:harden path/to/app.log
```

The skill reads the log, checks the ledger, writes tests, fixes code, commits, and updates the ledger.

## Log requirements

Your log file needs two things:

1. **Timestamped lines** — any format (ISO 8601, syslog, whatever)
2. **`ERROR:` prefix** on error lines

```
[2026-02-22T10:00:00Z] INFO: Server started on port 3000
[2026-02-22T10:00:05Z] ERROR: Cannot read properties of null (reading 'email') at src/auth.ts:42
```

Everything else — format, structure, verbosity, rotation — is up to you.

## What gets created

On first run, the skill creates:

```
.harden/ledger.md
```

The ledger is human-readable markdown recording every error addressed: root cause, tests written, fixes applied, commit hash, timestamp. Commit it to your repo — it's project knowledge.

## How it works

1. Reads the hardening ledger to know what's already been addressed
2. Reads the log file, finds `ERROR:` lines
3. Compares against the ledger to identify **new** errors
4. For each new error: analyzes root cause → writes a failing test → fixes the code → verifies the test passes
5. Commits all tests and fixes in one commit
6. Appends entries to the ledger with the commit hash

Errors that can't be fixed (environmental issues, dependency bugs) are recorded as `acknowledged` so they don't reappear as new.

## Specs

- [Vision](specs/vision.md) — why this exists
- [Requirements](specs/requirements.md) — what it does
- [Design](specs/design-spec.md) — how it works
