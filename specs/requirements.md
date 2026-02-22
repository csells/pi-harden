# harden — Requirements

## What harden is

An agent skill. A markdown file with instructions any coding agent follows using standard tools (read, write, edit, bash). No custom runtime, no framework integration, no agent-specific APIs.

## What harden is NOT

- Not a development-time error capture system — it does not intercept agent tool calls or hook into agent lifecycles
- Not a logging framework — the application produces its own logs however it wants
- Not a log management system — log rotation, retention, and storage are the application's concern
- Not agent-specific — it follows the [Agent Skills standard](https://agentskills.io) and works with any compatible coding agent

## Log Contract

The skill works with any log file that meets two requirements:

1. **Lines are timestamped** — any format the agent can parse (ISO 8601, syslog, etc.)
2. **Errors are prefixed with `ERROR:`** — clearly distinguishable from normal operations

Everything else is up to the application.

## Functional Requirements

### R1: Log Reading

- The skill takes a **file path** to the application's runtime log
- It reads the log looking for `ERROR:` lines
- It extracts timestamps and error content from each line
- It handles any reasonable timestamp format

### R2: New Error Detection

- The skill reads the hardening ledger (`.harden/ledger.md`)
- It compares log errors against ledger entries using timestamps and error descriptions
- An error is **new** if:
  - It has no matching entry in the ledger, OR
  - It matches a `fixed` entry but reappeared after the fix date
- The agent uses judgment to match — exact string matching is not required

### R3: Error Reproduction

- For each new error, the skill writes a **test that reproduces the error**
- The test must **fail** before the fix is applied
- Tests use whatever test framework the project already uses
- If no test framework exists, the skill sets one up

### R4: Error Fixing

- For each new error, the skill **fixes the code** so the reproducing test passes
- Fixes are targeted — they address the specific error, not broad refactors

### R5: Ledger Management

- The ledger lives at `.harden/ledger.md` — version-controlled, human-readable markdown
- Each entry records: timestamp, error description, root cause, tests added, files fixed, status
- Status is `fixed` or `acknowledged` (with reason)
- Append-only — new entries added at the bottom

### R6: Unfixable Error Handling

- Errors the skill cannot reproduce or fix are recorded as `acknowledged` with a reason
- Acknowledged errors are not re-analyzed on subsequent runs

## Invocation

```
/skill:harden path/to/app.log
```

## Deliverables

1. `skills/harden/SKILL.md` — the skill
2. `skills/harden/references/ledger-format.md` — ledger format reference
3. `.harden/ledger.md` — created in target projects on first run

## Non-Goals

- Log aggregation or collection from multiple sources
- Real-time log monitoring or tailing
- Alerting or notification
- Log format standardization
- Agent lifecycle hooks or tool interception
- Agent-specific packaging, APIs, or directory conventions
