# harden — Design

> See also: [Vision](vision.md) · [Requirements](requirements.md)

## It's just a skill

harden is an agent skill — a `SKILL.md` file with instructions any coding agent follows. No extension, no custom tools, no runtime code. The agent uses standard tools (read, write, edit, bash) to do all the work.

The skill follows the [Agent Skills standard](https://agentskills.io) and works with any compatible coding agent.

## Skill Structure

```
skills/harden/
├── SKILL.md                    — the skill instructions
└── references/
    └── ledger-format.md        — ledger format and matching guidance
```

On first run in a project, the skill creates:

```
.harden/
└── ledger.md                   — hardening ledger (version-controlled)
```

## The Workflow

```
┌─────────────────────────────────────────────────┐
│  App runs in production / staging / local dev    │
│  Produces timestamped logs with ERROR: lines     │
└──────────────┬──────────────────────────────────┘
               │
               │  Developer invokes:
               │  /skill:harden path/to/app.log
               │
               ▼
┌─────────────────────────────────────────────────┐
│  1. Read .harden/ledger.md                       │
│     → What's been addressed? Last run timestamp? │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│  2. Read the log file                            │
│     → Find ERROR: lines                          │
│     → Focus on errors after last run             │
│     → Compare against ledger                     │
│     → Identify genuinely NEW errors              │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│  3. For each new error:                          │
│     a. Read source at failing location           │
│     b. Analyze root cause                        │
│     c. Write a failing test                      │
│     d. Fix the code                              │
│     e. Verify test passes                        │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│  4. Append entries to ledger                     │
└─────────────────────────────────────────────────┘
```

## How "New" Detection Works

The agent uses judgment, not pattern matching. It reads ledger entries and log errors and determines whether each error has been addressed.

Factors to consider:
- **Timestamp**: Is this error from after the last hardening run?
- **Similarity**: Does it match a ledger entry in substance, even if the exact message differs?
- **Status**: Was a matching entry `fixed` or `acknowledged`?
- **Recurrence**: Does a `fixed` error appear again after the fix date? If so, the fix didn't hold — treat as new.

This is deliberately fuzzy. A TypeError at `auth.ts:42` and a TypeError at `auth.ts:45` after a code change are probably the same issue. The agent can make that call.

## What the Skill Does NOT Do

- **Does not produce logs.** The application produces its own logs.
- **Does not manage log files.** Rotation, retention, cleanup are the application's concern.
- **Does not hook into any agent.** No lifecycle events, no tool interception, no agent APIs.
- **Does not run at development time.** It reads runtime logs from the running application.
- **Does not monitor logs.** It reads a log file when invoked and processes what it finds.
