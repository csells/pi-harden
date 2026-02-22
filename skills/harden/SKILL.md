---
name: harden
description: >
  Harden any application against its runtime errors. Reads the app's runtime
  log file, finds new errors not yet in the hardening ledger, writes tests to
  reproduce them, fixes the code, and updates the ledger.
  Use when the user says "harden", provides a runtime log file to analyze for
  errors, or wants to systematically fix errors found in application logs.
  Triggers: harden, runtime errors, log errors, error hardening, fix log errors,
  错误加固, ランタイムエラー
---

# Harden

Systematically fix runtime errors found in application logs.

## Invocation

The user provides a path to a runtime log file:

```
/skill:harden path/to/app.log
```

## Log contract

The log file has two requirements:

1. Lines are timestamped (any format: ISO 8601, syslog, etc.)
2. Error lines contain `ERROR:` prefix

Everything else — format, structure, verbosity — varies by application.

## Workflow

### 1. Read the ledger

Read `.harden/ledger.md`. If it doesn't exist, this is the first run — create it with just a `# Hardening Ledger` header. Note the timestamp of the most recent entry.

### 2. Read the log

Read the log file provided by the user. Find all `ERROR:` lines. Focus on errors timestamped after the last ledger entry. On first run, consider all errors.

### 3. Identify new errors

Compare each error against the ledger entries. An error is **new** if:

- No ledger entry describes the same underlying issue, OR
- A ledger entry exists with status `fixed` but the error reappeared after the fix date (the fix didn't hold)

Use judgment to match — the same root cause may produce slightly different messages across runs. Errors already in the ledger as `fixed` (before the error timestamp) or `acknowledged` are not new.

Group errors that share the same root cause — multiple log lines for the same underlying bug count as one error. If no new errors are found, report that and stop — skip steps 4 and 5.

### 4. For each new error

a. **Read the source** at the location mentioned in the error (stack trace, file:line references).

b. **Analyze the root cause** — what input, state, or condition caused this?

c. **Write a failing test** that reproduces the error. Use whatever test framework the project already uses. If no test framework exists, set one up using the standard choice for the project's language (e.g., pytest for Python, Jest for JS/TS, JUnit for Java). Run the test to confirm it fails.

d. **Fix the code** so the test passes. Run the test to confirm.

e. If the error **cannot be reproduced or fixed** (environmental issue, dependency bug, transient failure), skip the test and fix. It will be recorded as `acknowledged` in step 5.

### 5. Update ledger

**Append entries** to `.harden/ledger.md` — one entry per error addressed. See [references/ledger-format.md](references/ledger-format.md) for the format.

### 6. Report

Summarize what was done: how many new errors found, how many fixed, how many acknowledged.
