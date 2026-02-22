# harden-skill

## What this project is

An agent skill that hardens any application against its runtime errors. It reads the app's runtime logs, finds new errors, writes tests to reproduce them, fixes the code, and tracks everything in a hardening ledger.

**Runtime, not development time.** The skill reads logs produced by the application running in production, staging, or local dev — not errors from the coding agent itself.

**Agent-agnostic.** The skill follows the [Agent Skills standard](https://agentskills.io). It works with any coding agent that supports skills and uses only standard tools: read, write, edit, bash.

## Structure

```
skills/harden/
├── SKILL.md                      — the skill
└── references/
    └── ledger-format.md          — ledger format and examples
specs/                            — vision, requirements, design
```

When the skill runs in a project, it creates:
```
.harden/ledger.md                — hardening ledger (version-controlled)
```

## Key concepts

- **Runtime log**: A timestamped log file produced by the running application. Lines prefixed with `ERROR:` are errors. The skill reads this — it never writes to it.
- **Hardening ledger**: Markdown at `.harden/ledger.md` recording every error addressed — root cause, tests, fixes, timestamp.
- **New error**: An error in the log with no matching ledger entry, or one that recurred after a previous fix.
