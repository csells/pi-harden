# Ledger Format

The ledger is a markdown file at `.harden/ledger.md`. It is version-controlled and append-only.

## Structure

Each entry is an H2 section. Entries are appended at the bottom. The `<timestamp>` is when the error was addressed (now), not when it originally occurred.

```markdown
# Hardening Ledger

## <timestamp>

### Error
<error message as it appeared in the log>

### Root Cause
<what caused this error — the actual bug, not just the symptom>

### Tests Added
- <file path> — "<test name or description>"

### Files Fixed
- <file path> — <what was changed>

### Status
<fixed | acknowledged — reason>
```

## Status values

- **fixed** — a test was written and the code was fixed
- **acknowledged** — the error cannot be fixed in this codebase (environmental, dependency, transient) — include the reason after the dash

## Example: fixed error

```markdown
## 2026-02-22T10:30:00Z

### Error
Cannot read properties of null (reading 'email') at src/auth.ts:42

### Root Cause
getUser() returns null when the session token is expired, but handleLogin()
assumes it always returns a user object.

### Tests Added
- src/auth.test.ts — "returns 401 when session token is expired"

### Files Fixed
- src/auth.ts — added null check after getUser() call, return 401 if null

### Status
fixed
```

## Example: acknowledged error

```markdown
## 2026-02-22T10:30:00Z

### Error
ECONNREFUSED 127.0.0.1:5432

### Root Cause
Database connection refused when PostgreSQL is restarting. Infrastructure
issue, not a code defect.

### Tests Added
(none)

### Files Fixed
(none)

### Status
acknowledged — infrastructure issue outside application control
```

## Matching guidance

When checking if a log error matches a ledger entry, consider:

- Same error type at the same or nearby source location → likely same issue
- Same message with a different line number after code changes → likely same issue
- Same root cause producing a different surface message → likely same issue
- Similar message in a completely different module → likely different issue
